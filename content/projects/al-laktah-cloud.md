---
title: "al-laktah-cloud"
date: 2026-10-25
description: "My personal cloud on Oracle Cloud's Always Free tier: Terraform-built VMs, Docker Compose services behind Caddy, a self-hosted NetBird VPN, nightly encrypted backups and outside monitoring, at no cost."
stack: ["Terraform", "Oracle Cloud (OCI)", "Docker Compose", "Caddy", "NetBird", "restic", "Gatus", "1Password"]
repo: "https://github.com/al-laktah/al-laktah-cloud"
---

My personal cloud on Oracle Cloud's Always Free tier, at no cost. Terraform builds the infrastructure, the services run as Docker Compose stacks behind Caddy and are backed up every night, a second, tiny VM watches it all from outside, and a third runs my NetBird VPN server.

## Infrastructure

- **Main VM:** an Ampere A1 (Arm) VM with 2 OCPUs, 12 GB RAM and a 50 GB boot volume, running Ubuntu 24.04.
- **Micro VMs:** two AMD micro VMs with 1/8 OCPU and 1 GB RAM each. One is the watchtower, which monitors the main VM and verifies the backups; the other runs the NetBird VPN server.
- **Network:** a dedicated VCN with a public subnet and a security list that lets SSH, HTTP and HTTPS (including HTTP/3) in. The NetBird VM gets its own network security group, which opens STUN's UDP port on that VM only. SSH logins are key-only, YubiKey security keys included.
- **DNS:** Cloudflare A records for the VMs and for each website, all following the VMs' public IPs.
- **Storage:** three Object Storage buckets: Terraform state, the main VM's backups and NetBird's backups. A separate bootstrap config creates them, so they outlive any rebuild of the VMs.
- **Access:** the VMs sign in to OCI as *instance principals*, so none of them holds any cloud credentials.

Everything fits inside the Always Free allowance: the whole A1 allowance, both AMD micro VMs and 150 of the 200 GB of block storage.

## Services

- **Caddy edge proxy:** the only container on the main VM that publishes ports. It gets and renews HTTPS certificates for every site and routes each hostname to its container over a shared Docker network, while databases and caches stay on each stack's private network.
- **Pocket ID:** a passkey-only OpenID Connect provider that signs me in to the other services with a YubiKey. No passwords and no public sign-up.
- **Outline:** a self-hosted wiki with Postgres and Redis, signing in through Pocket ID.
- **[Shorti](/projects/shorti/):** my URL shortener, deployed from its own repository.

A setup script prepares each VM (updates, its own firewall, Docker, log rotation), and a `deploy.sh` script ships a stack to it: it syncs the files, fills in the stack's secrets from 1Password with `op inject`, writes them straight to the VM over SSH and runs `docker compose up -d`.

## VPN

The second micro VM runs [NetBird](https://netbird.io)'s control server for a WireGuard mesh between my devices. It signs them in and helps them find each other; their traffic flows directly between them, and only goes through the server's relay when no direct connection is possible. The stack runs the NetBird server (management, signal, relay and STUN in one process), its dashboard, and a Caddy of its own for HTTPS.

Devices sign in through NetBird's built-in identity provider or through Pocket ID. The built-in admin account stays as a fallback, so I can still get in when the main VM, and with it Pocket ID, is down.

## Backups

Every night, a backup stack on the main VM dumps Outline's Postgres database and copies the SQLite databases of Pocket ID and Shorti with SQLite's online backup, which stays consistent while the apps keep writing. restic then encrypts those copies and the uploaded files on the VM, uploads only what changed, keeps 7 daily, 4 weekly and 6 monthly snapshots, and checks the repository. The NetBird VM backs up its own databases the same way, to a bucket of its own.

## Monitoring

The watchtower runs two containers:

- **Gatus** checks the sites and the NetBird server over HTTPS and the main VM's SSH port every minute. It alerts my phone through ntfy after three failed checks in a row, or when a certificate has less than 10 days left.
- **backup-check** verifies both sets of backups every day, read-only: the latest one must be under 26 hours old, `restic check` re-reads a different quarter of the data each day (all of it every 4 days), and the databases are restored to prove they load, with Outline's dump going into a throwaway Postgres.

If a daily check stops reporting, a heartbeat in Gatus raises an alert. If the watchtower itself goes quiet, an external dead man's switch notices.

## What I focused on

- **Two providers wired together.** The DNS records point at the VMs' IPs, so if a VM is rebuilt, the same `apply` updates DNS.
- **Guard rails as code.** Variable validation rejects anything beyond the free tier and catches malformed IDs and hostnames before any API call is made.
- **Remote state done properly.** The state bucket is bootstrapped separately to solve the chicken-and-egg problem, versioned for recovery, locked against concurrent runs and protected with `prevent_destroy`.
- **Least privilege.** Each VM can write only to its own backups bucket, so the NetBird VM, the most exposed one, can't reach the main VM's backups. The watchtower can only read, so it can't damage what it checks.
- **Backups you can restore from.** Consistent database copies, encrypted and deduplicated, and proven restorable every day on a separate machine.
- **Config changes that take effect.** `docker compose up -d` only recreates a container when its own definition changes, not when a config file it reads does. `deploy.sh` fingerprints each stack's files and secrets, so any change restarts the containers that depend on them.
- **Working around platform limits.** OCI's API rejects FIDO2 (`sk-`) SSH keys, so they're installed through cloud-init. `ignore_changes` stops new image releases from forcing a rebuild, and Terraform looks up which availability domain offers the micro shape.
- **The right tool for each layer.** Terraform owns the VMs, network and DNS, Docker Compose owns the apps, and 1Password owns the secrets.
- **No secrets in git.** Personal values live in git-ignored files, each stack commits only an `.env.tpl` with 1Password references, and `op run` hands the Cloudflare token to each Terraform command instead of leaving it in the shell.
- **CI.** GitHub Actions runs `terraform fmt` and `terraform validate` on both Terraform configs, checks every service stack with `docker compose config` and lints every shell script with a pinned, checksummed ShellCheck, without needing cloud credentials.

## Next steps

- Publishing Shorti's images to a registry, so `deploy.sh` can deploy it like the other stacks

The repository's README has an architecture diagram and a step-by-step setup guide.

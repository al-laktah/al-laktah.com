---
title: "al-laktah-cloud"
date: 2026-10-25
description: "My personal cloud on Oracle Cloud's Always Free tier: Terraform-built VMs, Docker Compose services behind Caddy, nightly encrypted backups and monitoring from a second VM, at no cost."
stack: ["Terraform", "Oracle Cloud (OCI)", "Docker Compose", "Caddy", "Cloudflare", "restic", "Gatus", "1Password"]
repo: "https://github.com/al-laktah/al-laktah-cloud"
---

My personal cloud on Oracle Cloud's Always Free tier, at no cost. Terraform builds the infrastructure, the services run as Docker Compose stacks behind Caddy and are backed up every night, and a second, tiny VM watches it all from outside.

## Infrastructure

- **Main VM:** an Ampere A1 (Arm) VM with 2 OCPUs, 12 GB RAM and a 50 GB boot volume, running Ubuntu 24.04.
- **Watchtower VM:** an AMD micro VM (1/8 OCPU, 1 GB RAM) that monitors the main one and verifies its backups.
- **Network:** a dedicated VCN with a public subnet and a security list that lets SSH, HTTP and HTTPS (including HTTP/3) in. SSH logins are key-only, YubiKey security keys included.
- **DNS:** Cloudflare A records for the VMs and for each website, all following the VMs' public IPs.
- **Storage:** two Object Storage buckets, one for Terraform state and one for backups. A separate bootstrap config creates them, so they outlive any rebuild of the VMs.
- **Access:** both VMs sign in to OCI as *instance principals*. The main VM may write to the backups bucket, the watchtower may only read it, and neither holds any cloud credentials.

Everything fits inside the Always Free allowance.

## Services

- **Caddy edge proxy:** the only container that publishes ports. It gets and renews HTTPS certificates for every site and routes each hostname to its container over a shared Docker network, while databases and caches stay on each stack's private network.
- **Pocket ID:** a passkey-only OpenID Connect provider that signs me in to the other services with a YubiKey. No passwords and no public sign-up.
- **Outline:** a self-hosted wiki with Postgres and Redis, signing in through Pocket ID.
- **[Shorti](/projects/shorti/):** my URL shortener, deployed from its own repository.

A setup script prepares each VM (updates, its own firewall, Docker, log rotation), and a `deploy.sh` script ships a stack to it: it syncs the files, fills in the stack's secrets from 1Password with `op inject`, writes them straight to the VM over SSH and runs `docker compose up -d`.

## Backups

Every night, a backup stack dumps Outline's Postgres database and copies the SQLite databases of Pocket ID and Shorti with SQLite's online backup, which stays consistent while the apps keep writing. restic then encrypts those copies and the uploaded files on the VM, uploads only what changed, keeps 7 daily, 4 weekly and 6 monthly snapshots, and checks the repository.

## Monitoring

The watchtower runs two containers:

- **Gatus** checks the sites over HTTPS and the main VM's SSH port every minute. It alerts my phone through ntfy after three failed checks in a row, or when a certificate has less than 10 days left.
- **backup-check** verifies the backups every day, read-only: the latest one must be under 26 hours old, `restic check` re-reads a different quarter of the data each day (all of it every 4 days), and the databases are restored to prove they load, with Outline's dump going into a throwaway Postgres.

If the daily check stops reporting, a heartbeat in Gatus raises an alert. If the watchtower itself goes quiet, an external dead man's switch notices.

## What I focused on

- **Two providers wired together.** The DNS records point at the VMs' IPs, so if a VM is rebuilt, the same `apply` updates DNS.
- **Guard rails as code.** Variable validation rejects anything beyond the free tier and catches malformed IDs and hostnames before any API call is made.
- **Remote state done properly.** The state bucket is bootstrapped separately to solve the chicken-and-egg problem, versioned for recovery, locked against concurrent runs and protected with `prevent_destroy`.
- **Least privilege.** Each VM's policy covers one bucket, and the watchtower can only read it, so it can't damage what it checks.
- **Backups you can restore from.** Consistent database copies, encrypted and deduplicated, and proven restorable every day on a separate machine.
- **Working around platform limits.** OCI's API rejects FIDO2 (`sk-`) SSH keys, so they're installed through cloud-init. `ignore_changes` stops new image releases from forcing a rebuild, and Terraform looks up which availability domain offers the micro shape.
- **The right tool for each layer.** Terraform owns the VMs, network and DNS, Docker Compose owns the apps, and 1Password owns the secrets.
- **No secrets in git.** Personal values live in git-ignored files, and each stack commits only an `.env.tpl` with 1Password references.
- **CI.** GitHub Actions runs `terraform fmt` and `terraform validate` on both Terraform configs and checks every service stack with `docker compose config`, without needing cloud credentials.

## Next steps

- Publishing Shorti's images to a registry, so `deploy.sh` can deploy it like the other stacks

The repository's README has an architecture diagram and a step-by-step setup guide.

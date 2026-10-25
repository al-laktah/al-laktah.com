---
title: "al-laktah-cloud"
date: 2026-10-25
description: "My personal cloud on Oracle Cloud's Always Free tier: Terraform builds the Arm VM, network, Cloudflare DNS and remote state, and Docker Compose runs the services behind Caddy."
stack: ["Terraform", "Oracle Cloud (OCI)", "Docker Compose", "Caddy", "Cloudflare", "1Password", "GitHub Actions"]
repo: "https://github.com/al-laktah/al-laktah-cloud"
---

My personal cloud on Oracle Cloud's Always Free tier, at no cost. Terraform builds the infrastructure: an Arm VM, its network, Cloudflare DNS records and remote state. The services on the VM run as Docker Compose stacks behind a single Caddy proxy.

## What Terraform builds

- **Compute:** one Ampere A1 (Arm) VM with 2 OCPUs, 12 GB RAM and a 50 GB boot volume, running Ubuntu 24.04.
- **Network:** a dedicated VCN with a public subnet, an internet gateway and a security list that lets SSH, HTTP and HTTPS (including HTTP/3) in. SSH logins are key-only, YubiKey security keys included.
- **DNS:** Cloudflare A records for the VM and for each website it serves, all following the VM's public IP.
- **State:** a private, versioned OCI Object Storage bucket with locking, created by a separate bootstrap config.

Everything fits inside the Always Free allowance.

## Services on the VM

- **Caddy edge proxy:** the only container that publishes ports. It gets and renews HTTPS certificates for every site and routes each hostname to its container over a shared Docker network, while databases and caches stay on each stack's private network.
- **Pocket ID:** a passkey-only OpenID Connect provider that signs me in to the other services with a YubiKey. No passwords and no public sign-up.
- **Outline:** a self-hosted wiki with Postgres and Redis, signing in through Pocket ID.
- **[Shorti](/projects/shorti/):** my URL shortener, deployed from its own repository.

A `deploy.sh` script ships a stack to the VM: it syncs the files, fills in the stack's secrets from 1Password with `op inject`, writes them straight to the VM over SSH and runs `docker compose up -d`. It's safe to re-run, and reloads Caddy when a site is added.

## What I focused on

- **Two providers wired together.** The DNS records point at the VM's IP, so if the VM is rebuilt, the same `apply` updates DNS.
- **Guard rails as code.** Variable validation rejects anything beyond the free tier and catches malformed IDs and hostnames before any API call is made.
- **Remote state done properly.** The state bucket is bootstrapped separately to solve the chicken-and-egg problem, versioned for recovery, locked against concurrent runs and protected with `prevent_destroy`.
- **Working around platform limits.** OCI's API rejects FIDO2 (`sk-`) SSH keys, so the keys are installed through cloud-init instead. `ignore_changes` stops new image releases from forcing a rebuild.
- **The right tool for each layer.** Terraform owns the VM, network and DNS, Docker Compose owns the apps, and 1Password owns the secrets.
- **No secrets in git.** Credentials come from `~/.oci` and 1Password at runtime. Personal values live in git-ignored files, and each stack commits only an `.env.tpl` with 1Password references.
- **CI.** GitHub Actions runs `terraform fmt` and `terraform validate` on both Terraform configs and checks every service stack with `docker compose config`, without needing cloud credentials.

## Next steps

- Automated backups of the service data to Object Storage
- Publishing Shorti's images to a registry, so `deploy.sh` can deploy it like the other stacks

The repository's README has an architecture diagram and a step-by-step setup guide.

---
title: "al-laktah-cloud"
date: 2026-10-25
description: "My personal cloud on Oracle Cloud's Always Free tier, built with Terraform: an Arm VM, its network, Cloudflare DNS and remote state, at no cost."
stack: ["Terraform", "Oracle Cloud (OCI)", "Cloudflare", "cloud-init", "GitHub Actions"]
repo: "https://github.com/al-laktah/al-laktah-cloud"
---

My personal cloud on Oracle Cloud's Always Free tier, defined entirely in Terraform. One `apply` builds an Arm VM, its network and a Cloudflare DNS record, with the Terraform state stored remotely, all at no cost.

## What it builds

- **Compute:** one Ampere A1 (Arm) VM with 2 OCPUs, 12 GB RAM and a 50 GB boot volume, running Ubuntu 24.04.
- **Network:** a dedicated VCN with a public subnet, an internet gateway and a security list that only lets SSH in. Logins are key-only, YubiKey security keys included.
- **DNS:** a Cloudflare A record that follows the VM's public IP.
- **State:** a private, versioned OCI Object Storage bucket with locking, created by a separate bootstrap config.

Everything fits inside the Always Free allowance.

## What I focused on

- **Two providers wired together.** The DNS record points at the VM's IP, so if the VM is rebuilt, the same `apply` updates DNS.
- **Guard rails as code.** Variable validation rejects anything beyond the free tier and catches malformed IDs before any API call is made.
- **Remote state done properly.** The state bucket is bootstrapped separately to solve the chicken-and-egg problem, versioned for recovery, locked against concurrent runs and protected with `prevent_destroy`.
- **Working around platform limits.** OCI's API rejects FIDO2 (`sk-`) SSH keys, so the keys are installed through cloud-init instead. `ignore_changes` stops new image releases from forcing a rebuild.
- **No secrets in git.** Credentials come from `~/.oci` and a password manager at runtime. Personal values live in git-ignored files, with committed `.example` templates.
- **CI.** GitHub Actions runs `terraform fmt` and `terraform validate` on every push and pull request, without needing cloud credentials.

The repository's README has an architecture diagram and a step-by-step setup guide.

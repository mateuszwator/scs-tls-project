# Reproduce all three CVEs with Ansible

One playbook brings each lab up (vulnerable + patched), runs the attack, and
**asserts** the expected result. It wraps the per-lab scripts, so it is the
single entry point for reproducing everything.

| Lab | Vulnerable | Patched | Assertion |
|-----|-----------|---------|-----------|
| nginx CVE-2025-23419 | bypass succeeds | bypass rejected | `BYPASS SUCCESSFUL` only on vuln |
| CrateDB CVE-2024-37309 | renegotiation DoS | refused, CPU flat | `reproduced` vuln / `fix in effect` patched |
| Traefik CVE-2025-66491 | HTTP 200 (MITM) | HTTP 500 | `HTTP status: 200` and `500` present |

## Where to run it

Ansible has **no native Windows control node**, so run this from **WSL2**,
Linux, or macOS. On Windows, Docker Desktop's WSL2 integration exposes the same
Docker engine (and published `127.0.0.1` ports) inside your WSL distro.

## Prerequisites (on the control node)

- `docker` (Docker Desktop engine reachable), `openssl`, `bash`, `perl`
- `java` + `keytool` (CrateDB keystore)
- `k3d` + `kubectl` (Traefik lab)
- `ansible` (`pip install ansible`) — no extra collections needed

Quick WSL2 (Ubuntu) setup:

```bash
sudo apt update && sudo apt install -y ansible openssl perl default-jdk
# k3d + kubectl:
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
sudo curl -Lo /usr/local/bin/kubectl "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && sudo chmod +x /usr/local/bin/kubectl
# Docker: enable Docker Desktop > Settings > Resources > WSL integration for this distro
```

## Run

```bash
cd ansible
ansible-playbook site.yml                  # all three
ansible-playbook site.yml --tags nginx     # just one
ansible-playbook site.yml --tags cratedb
ansible-playbook site.yml --tags traefik
ansible-playbook site.yml --tags teardown  # stop containers + delete k3d cluster
```

A clean run ends with green `ok` on each `assert` task.

> The Traefik lab also has a Kubernetes-native variant (using the
> `kubernetes.core` / `community.crypto` collections) in
> `../cve-2025-66491-traefik/ansible/`, if you prefer that over the wrapper here.

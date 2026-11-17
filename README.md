# scs-tls-project

Three recent TLS CVEs, each reproduced vulnerable vs patched, locally on `127.0.0.1`.
Old, public, already-fixed bugs — this just demonstrates them.

| CVE | Software | Issue | Fixed in |
|---|---|---|---|
| CVE-2025-23419 | nginx | client-cert auth bypass via TLS session resumption | 1.27.4 / 1.26.3 |
| CVE-2024-37309 | CrateDB | CPU DoS via client-initiated TLS renegotiation | 5.7.2 |
| CVE-2025-66491 | Traefik | inverted `proxy-ssl-verify` disables backend cert check (MITM) | 3.6.3 |

## Requirements

Docker + Compose, OpenSSL, `java`/`keytool` (CrateDB keystore), and `k3d` + `kubectl`
for the Traefik lab.

## Run everything

```bash
./demo.sh
```

## Run one lab

```bash
cd cve-2025-23419-nginx   # or the other cve-* dir
../common/gen-certs.sh "$PWD/certs"
docker compose up -d
./run.sh                  # see the script header for args
```

## Traefik (also via Ansible)

```bash
cd cve-2025-66491-traefik/ansible
ansible-galaxy collection install -r requirements.yml && pip install ansible kubernetes
ansible-playbook site.yml                  # builds the lab, asserts vuln=200 / patched=500
ansible-playbook site.yml --tags teardown  # clean up
```

Run Ansible from Linux/macOS/WSL2. Certs are generated locally and git-ignored.

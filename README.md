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

## Reproduce with Ansible

One playbook brings every lab up and verifies vulnerable vs patched:

```bash
cd ansible
ansible-playbook site.yml                  # all three (asserts each result)
ansible-playbook site.yml --tags nginx     # or cratedb / traefik
ansible-playbook site.yml --tags teardown  # stop everything
```

Run it from Linux/macOS/WSL2. See [`ansible/README.md`](ansible/README.md).
Certs are generated locally and git-ignored.

(The Traefik lab also has a Kubernetes-native playbook in
`cve-2025-66491-traefik/ansible/`.)

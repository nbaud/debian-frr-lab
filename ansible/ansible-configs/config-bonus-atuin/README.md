# Atuin Shell History Sync (Bonus Configuration)

This Ansible configuration installs and configures an Atuin server on `vm03` using Docker Compose, and installs the Atuin client on `vm01` and `vm02`.

It assumes `config1-ospf` has already been applied so that IP connectivity is established across hosts:
- `vm01`: `10.0.10.2`
- `vm02`: `10.0.20.2`
- `vm03`: `10.0.30.2`

## Deployment

Navigate to this directory and run the playbook:

```bash
cd ~/it/vagrant/debian-frr-lab/ansible/ansible-configs/config-bonus-atuin

ansible all -m ping
ansible-playbook atuin.yml
```

## Verification

Check the Docker container status on `vm03`:

```bash
ssh vm03 'sudo docker compose -f /opt/atuin/docker-compose.yml ps'
```

Verify server network reachability from a client:

```bash
ssh vm01 'curl -i http://10.0.30.2:8888'
```
*(Note: Receiving any HTTP response proves network connectivity; the root endpoint may return HTTP 404).*

Verify client installations and configured sync address:

```bash
ssh vm01 '/home/nico/.atuin/bin/atuin --version'
ssh vm01 '/home/nico/.atuin/bin/atuin config get sync_address'
ssh vm02 '/home/nico/.atuin/bin/atuin --version'
ssh vm02 '/home/nico/.atuin/bin/atuin config get sync_address'
```

## Interactive Demonstration Workflow

1. On `vm01` (Register new account & publish initial sync):

```bash
atuin register -u nico -e nico@example.invalid
atuin key
echo "hello from vm01"
atuin sync
```

> [!IMPORTANT]
> Save or display the encryption key output by `atuin key` if needed for reference. Note that the encryption key is secret in a real deployment, and plain HTTP / open registration are acceptable only for this isolated demo lab network.

2. On `vm02` (Log into account & search history):

```bash
atuin login -u nico
atuin sync
atuin search "hello from vm01"
echo "hello from vm02"
atuin sync
```

3. Back on `vm01` (Sync & verify history from `vm02`):

```bash
atuin sync
atuin search "hello from vm02"
```

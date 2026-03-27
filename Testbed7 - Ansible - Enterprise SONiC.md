<p align="center">
  <i>Ansible Automation for Enterprise SONiC</i>
</p>
<p align="center">
  Built by <b>Jason S</b>
</p>
<p align="center">
  <a href="#">
    <img src="https://img.shields.io/badge/▶ Watch Demo-YouTube-red?style=for-the-badge&logo=youtube"/>
  </a>
</p>

---
<p align="center">
  <b>Ansible Enterprise SONiC configuration via OpenConfig YANG REST API using Ansible</b>
</p>

---

## Overview

This testbed demonstrates how to use Ansible to automate configuration management on a Broadcom Enterprise SONiC switch (M2-W6510-32C) via REST API (OpenConfig YANG over HTTPS). The control node is a macOS machine; no software installation is required on the switch.

The Enterprise SONiC Management Framework container exposes a full REST API based on OpenConfig and SONiC YANG models, which Ansible communicates with directly over HTTPS — no SSH CLI involved.

> The `dellemc.enterprise_sonic` collection works because both Dell and this platform share the same Broadcom Enterprise SONiC Management Framework, exposing identical OpenConfig YANG models over REST API.

---

## Architecture

```
macOS (Control Node)
  └── ansible-playbook
        └── dellemc.enterprise_sonic collection
              └── HTTPS (port 443)
                    └── mgmt-framework container (Enterprise SONiC)
                          └── ConfigDB (Redis)
                                └── SONiC daemons (vlanmgrd, bgpd, ...)
                                      └── ASIC / Kernel
```

**Key point:** Ansible communicates with the switch via REST API (HTTPS PUT/GET/DELETE), not SSH CLI. The `dellemc.enterprise_sonic` collection handles GET → diff → PUT automatically, providing idempotent operations.

---

## Environment

| Component | Detail |
|---|---|
| Switch | M2-W6510-32C |
| NOS | Broadcom Enterprise SONiC |
| Control Node | macOS (Apple Silicon) |
| Ansible | 2.17+ |
| Collection | dellemc.enterprise_sonic 3.2.0 |
| Connection | httpapi (HTTPS port 443) |
| YANG | OpenConfig + SONiC native YANG |

---

## Prerequisites

### Verify Management Framework is running on the switch

```bash
ssh admin@<switch-ip>
docker ps | grep mgmt
```

Expected output:
```
293847382dx   docker-sonic-mgmt-framework:latest   "/usr/local/bin/supe…"   Up   mgmt-framework
```

### Verify REST API is reachable

```bash
curl -sk -u admin:<password> \
  https://<switch-ip>/restconf/data/openconfig-interfaces:interfaces \
  | python3 -m json.tool | head -20
```

If you see `openconfig-interfaces:interfaces` in the response, the REST API is working.

---

## Installation (macOS)

### 1. Install Ansible

```bash
pip3 install ansible
```

Verify:
```bash
ansible --version
```

### 2. Install the Enterprise SONiC collection

```bash
ansible-galaxy collection install dellemc.enterprise_sonic
```

Verify:
```bash
ansible-galaxy collection list | grep dellemc
```

Verify:
```bash
ansible-galaxy collection install dellemc.enterprise_sonic --upgrade
```

Expected output:
```
dellemc.enterprise_sonic   4.1.0
```

---

## Project Structure

```
sonic-fabric/
├── inventory.ini       # Device inventory
├── test.yaml           # Test playbook (create VLAN)
└── cleanup.yaml        # Cleanup playbook (delete VLAN)
```

---

## Configuration

### inventory.ini

```ini
[sonic_switches]
sonic1 ansible_host=<switch-ip>

[sonic_switches:vars]
ansible_user=admin
ansible_password=<password>
ansible_network_os=dellemc.enterprise_sonic.sonic
ansible_httpapi_use_ssl=true
ansible_httpapi_validate_certs=false
ansible_connection=httpapi
```

**Key variables:**

| Variable | Value | Purpose |
|---|---|---|
| `ansible_connection` | `httpapi` | Use REST API, not SSH |
| `ansible_network_os` | `dellemc.enterprise_sonic.sonic` | Select the correct httpapi plugin |
| `ansible_httpapi_use_ssl` | `true` | HTTPS (port 443) |
| `ansible_httpapi_validate_certs` | `false` | Skip self-signed cert validation |

---

## Playbooks

### test.yaml — Create VLAN

```yaml
- name: Test connection
  hosts: sonic_switches
  gather_facts: no
  connection: httpapi
  tasks:
    - name: Create VLAN 999
      dellemc.enterprise_sonic.sonic_vlans:
        config:
          - vlan_id: 999
        state: merged
      register: result
    - debug:
        var: result
```

### cleanup.yaml — Delete VLAN

```yaml
- name: Cleanup test
  hosts: sonic_switches
  gather_facts: no
  connection: httpapi
  tasks:
    - name: Delete VLAN 999
      dellemc.enterprise_sonic.sonic_vlans:
        config:
          - vlan_id: 999
        state: deleted
```

---

## Running the Playbooks

```bash
cd ~/sonic-fabric

# Create VLAN 999
ansible-playbook -i inventory.ini test.yaml

# Delete VLAN 999
ansible-playbook -i inventory.ini cleanup.yaml
```

---

## Expected Output

### Create (first run)

```
TASK [Create VLAN 999]
changed: [sonic1]

result:
  before: []
  after:
    - vlan_id: 999
      autostate: true
  changed: true
```

### Create (second run — idempotent)

```
TASK [Create VLAN 999]
ok: [sonic1]

changed: false
```

The second run returns `ok` instead of `changed` — this is idempotency in action. The module performs a GET first, detects VLAN 999 already exists, and skips the PUT.

### Delete

```
TASK [Delete VLAN 999]
changed: [sonic1]

result:
  before:
    - vlan_id: 999
  after: []
  changed: true
```

---

## How It Works

Each Ansible module in the `dellemc.enterprise_sonic` collection follows this pattern:

```
1. GET  → fetch current device state via REST API
2. DIFF → compare current state with desired state (from playbook)
3. PUT  → push only the delta (nothing if already in desired state)
4. GET  → fetch new state to populate 'after'
```

The REST paths used for VLAN operations:

| Operation | Method | REST Path |
|---|---|---|
| Create | PUT | `restconf/data/sonic-vlan:sonic-vlan/VLAN/VLAN_LIST=VlanXXX` |
| Delete | DELETE | `restconf/data/sonic-vlan:sonic-vlan/VLAN/VLAN_LIST=VlanXXX` |
| Read | GET | `restconf/data/sonic-vlan:sonic-vlan/VLAN` |

---

## state Options

| State | Behavior |
|---|---|
| `merged` | Add config to existing (non-destructive) |
| `deleted` | Remove specified config |
| `replaced` | Replace config for specified objects |
| `overridden` | Replace entire resource config on device |

---

## License Note

The `dellemc.enterprise_sonic` collection is licensed under GPL v3. It is used here for testbed validation purposes. The collection is installed directly from Ansible Galaxy and is not distributed or modified.

---

## References

- [Micas M2-W6510-32C](https://micasnetworks.com/products/open-network-switches/class-100g/M2-W6510-32C)
- [dellemc.enterprise_sonic on Ansible Galaxy](https://galaxy.ansible.com/dellemc/enterprise_sonic)
- [dellemc.enterprise_sonic on GitHub](https://github.com/ansible-collections/dellemc.enterprise_sonic)
- [OpenConfig YANG Models](https://www.openconfig.net)

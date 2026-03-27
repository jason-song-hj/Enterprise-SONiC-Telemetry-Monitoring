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
| NOS | SONiC-OS-4.5.1-Enterprise_Advanced |
| Control Node | macOS (Apple Silicon) |
| Ansible | 2.17+ |
| Collection | dellemc.enterprise_sonic 4.1.0 |
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
├── inventory.ini            # Device inventory
├── group_vars/
│   └── sonic_switches.yaml  # Shared variables (VLANs, IPs)
├── fabric.yaml              # Main playbook
└── cleanup.yaml             # Cleanup playbook
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

### group_vars/sonic_switches.yaml

```yaml
vlans:
  - vlan_id: 10
    description: "Server"
  - vlan_id: 20
    description: "Storage"
  - vlan_id: 30
    description: "Replication"
```

---

## Playbooks

### fabric.yaml — Full configuration

```yaml
- name: Configure Enterprise SONiC
  hosts: sonic_switches
  gather_facts: no
  connection: httpapi
  tasks:
    - name: Create VLANs
      dellemc.enterprise_sonic.sonic_vlans:
        config: "{{ vlans }}"
        state: merged

    - name: Configure SVI IPs
      dellemc.enterprise_sonic.sonic_l3_interfaces:
        config:
          - name: Vlan10
            ipv4:
              addresses:
                - address: 192.168.10.1/24
          - name: Vlan20
            ipv4:
              addresses:
                - address: 192.168.20.1/24
          - name: Vlan30
            ipv4:
              addresses:
                - address: 192.168.30.1/24
        state: merged

    - name: Create Loopback0
      dellemc.enterprise_sonic.sonic_interfaces:
        config:
          - name: Loopback0
        state: merged

    - name: Configure Loopback0 IP
      dellemc.enterprise_sonic.sonic_l3_interfaces:
        config:
          - name: Loopback0
            ipv4:
              addresses:
                - address: 10.0.0.1/32
        state: merged
```

> **Note:** Loopback interfaces must be created first with `sonic_interfaces` before assigning an IP with `sonic_l3_interfaces`. Unlike VLAN interfaces which are created automatically when a VLAN is added, Loopback interfaces require explicit creation.

### cleanup.yaml — Remove all config

```yaml
- name: Cleanup all config
  hosts: sonic_switches
  gather_facts: no
  connection: httpapi
  tasks:
    - name: Delete Loopback0 IP
      dellemc.enterprise_sonic.sonic_l3_interfaces:
        config:
          - name: Loopback0
            ipv4:
              addresses:
                - address: 10.0.0.1/32
        state: deleted

    - name: Delete Loopback0
      dellemc.enterprise_sonic.sonic_interfaces:
        config:
          - name: Loopback0
        state: deleted

    - name: Delete SVI IPs
      dellemc.enterprise_sonic.sonic_l3_interfaces:
        config:
          - name: Vlan10
            ipv4:
              addresses:
                - address: 192.168.10.1/24
          - name: Vlan20
            ipv4:
              addresses:
                - address: 192.168.20.1/24
          - name: Vlan30
            ipv4:
              addresses:
                - address: 192.168.30.1/24
        state: deleted

    - name: Delete VLANs
      dellemc.enterprise_sonic.sonic_vlans:
        config: "{{ vlans }}"
        state: deleted
```

---

## Running the Playbooks

```bash
cd ~/sonic-fabric

# Apply full configuration
ansible-playbook -i inventory.ini fabric.yaml

# Remove all configuration
ansible-playbook -i inventory.ini cleanup.yaml
```

---

## Expected Output

### fabric.yaml (first run)

```
TASK [Create VLANs]
changed: [sonic1]

TASK [Configure SVI IPs]
changed: [sonic1]

TASK [Create Loopback0]
changed: [sonic1]

TASK [Configure Loopback0 IP]
changed: [sonic1]
```

### fabric.yaml (second run — idempotent)

```
TASK [Create VLANs]
ok: [sonic1]

TASK [Configure SVI IPs]
ok: [sonic1]

TASK [Create Loopback0]
ok: [sonic1]

TASK [Configure Loopback0 IP]
ok: [sonic1]
```

All tasks return `ok` — nothing pushed to the device because the desired state already matches.

### Verify via REST API

```bash
# Check SVI IPs
curl -sk -u admin:<password> \
  https://<switch-ip>/restconf/data/sonic-interface:sonic-interface \
  | python3 -m json.tool

# Check Loopback
curl -sk -u admin:<password> \
  https://<switch-ip>/restconf/data/sonic-loopback-interface:sonic-loopback-interface \
  | python3 -m json.tool
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

## More Examples

For more module usage examples covering interfaces, VLANs, LAG, L2/L3, and port breakout, see the official collection playbook examples:

[sonic_playbooks](https://github.com/ansible-collections/dellemc.enterprise_sonic/tree/main/playbooks/common_examples)

---

## License Note

The `dellemc.enterprise_sonic` collection is licensed under GPL v3. It is used here for testbed validation purposes. The collection is installed directly from Ansible Galaxy and is not distributed or modified.

---

## References

- [Micas M2-W6510-32C](https://micasnetworks.com/products/open-network-switches/class-100g/M2-W6510-32C)
- [dellemc.enterprise_sonic on Ansible Galaxy](https://galaxy.ansible.com/dellemc/enterprise_sonic)
- [dellemc.enterprise_sonic on GitHub](https://github.com/ansible-collections/dellemc.enterprise_sonic)
- [OpenConfig YANG Models](https://www.openconfig.net)

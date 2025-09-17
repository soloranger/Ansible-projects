# Cisco Switch Interface Privilege Automation with Ansible

This repository contains **inventory** and **Ansible playbooks** to automatically configure multiple Cisco switches.  
It creates a limited user account that can only control interface status (`shutdown` / `no shutdown`) and descriptions, without full administrative access to the switch.

---

## 📋 Prerequisites

1. **Python 3.8+**
2. Install Ansible and required collections:
   ```bash
   pip install ansible
   ansible-galaxy collection install cisco.ios ansible.netcommon
   ```

3. Ensure SSH is enabled on all switches:

    ```
   crypto key generate rsa
   ip ssh version 2
   line vty 0 4
   login local
   transport input ssh
   ```
4. Assign a management IP address to the correct VLAN interface and configure a default gateway.

## 📂 Project Structure

```
.
├── inventory/
│   └── hosts.ini          # List of all switches
├── playbooks/
│   └── apply.yml
└── README.md
└── ansible.cfg
```

## 📂 Inventory (hosts.ini)

All switches are listed inside inventory/hosts.ini:

```
[switches]
sw1  ansible_host=Switch IP

[switches:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=cisco.ios.ios
ansible_user=admin
ansible_password=AdminLoginPassword123
ansible_become=yes
ansible_become_method=enable
ansible_become_password=EnableSecret123
```

* Each host has a unique alias (swX).

* Common variables like connection method, network OS, and credentials are defined in [switches:vars].


## ▶️ Running the Playbook

Run on all switches:
```
ansible-playbook -i inventory/hosts.ini playbooks/apply.yml
```

Run on a single switch:

```
ansible-playbook -i inventory/hosts.ini playbooks/apply.yml -l sw11 -vv
```

## ⚙️ What the Playbook Does

* Creates a limited user (default: ops) with privilege level 2.

* Grants access only to:

  * configure terminal
  * interface
  * shutdown
  * no shutdown
  * description

* Ensures the user cannot perform higher-level tasks (like reload, write memory).
* Saves pre-check and post-check outputs for every switch to verify changes.

# 🛠 Troubleshooting
* Error: paramiko is not installed
→ Fix: pip install paramiko

* Error: Unable to connect to port 22
→ Ensure management VLAN is up and SSH is enabled on the switch.

* Error: authenticity of host can't be established
→ Either add the host key with ssh-keyscan or disable host-key checking:

```
export ANSIBLE_HOST_KEY_CHECKING=False
```

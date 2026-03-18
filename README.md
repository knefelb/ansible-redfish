# Ansible Redfish Playbooks

Ansible playbooks for managing HPE ProLiant servers via Redfish/iLO API.

## Prerequisites

```bash
pip install ansible
ansible-galaxy collection install community.general
```

## Setup

1. Copy secrets: `cp secrets.yml.example secrets.yml`
2. Edit with real passwords: `vim secrets.yml`
3. Encrypt: `ansible-vault encrypt secrets.yml`

## Playbooks

| # | Playbook | Description | Tags |
|---|----------|-------------|------|
| 01 | `01-inventory-report.yml` | Pull system info, CPU, memory, storage, firmware, NICs | — |
| 02 | `02-health-check.yml` | Pre-maintenance health validation (fails if not OK) | — |
| 03 | `03-power-management.yml` | Graceful shutdown, restart, power on/off | — |
| 04 | `04-pxe-boot.yml` | Set one-time PXE boot + reboot for OS deployment | — |
| 05 | `05-bios-settings.yml` | Read or change BIOS attributes | `read`, `write` |
| 06 | `06-firmware-update.yml` | Push firmware via SimpleUpdate | — |
| 07 | `07-user-management.yml` | List or create iLO local accounts | `list`, `create` |
| 08 | `08-event-subscriptions.yml` | Manage Redfish event subscriptions (webhooks) | `list`, `create`, `delete` |
| 09 | `09-boot-order.yml` | Read, set, or clear boot override | `read`, `set`, `clear` |

## Usage Examples

```bash
# Inventory report — all servers
ansible-playbook 01-inventory-report.yml --ask-vault-pass

# Health check — single server
ansible-playbook 02-health-check.yml --ask-vault-pass -l dar-vm01

# Graceful restart
ansible-playbook 03-power-management.yml --ask-vault-pass --extra-vars "power_action=GracefulRestart"

# PXE boot a specific server
ansible-playbook 04-pxe-boot.yml --ask-vault-pass -l dar-vm02

# Read BIOS settings
ansible-playbook 05-bios-settings.yml --ask-vault-pass --tags read

# Change BIOS setting (pending until reboot)
ansible-playbook 05-bios-settings.yml --ask-vault-pass --tags write \
  --extra-vars '{"bios_attributes": {"ProcHyperthreading": "Enabled"}}'

# Firmware update
ansible-playbook 06-firmware-update.yml --ask-vault-pass \
  --extra-vars "firmware_url=http://fileserver/ilo5_304.bin"

# List iLO users
ansible-playbook 07-user-management.yml --ask-vault-pass --tags list

# Create read-only monitoring user on all iLOs
ansible-playbook 07-user-management.yml --ask-vault-pass --tags create \
  --extra-vars '{"new_user": "monitor", "new_pass": "Mon1tor!", "new_role": "ReadOnly"}'

# Read boot order
ansible-playbook 09-boot-order.yml --ask-vault-pass --tags read

# Set persistent HDD boot
ansible-playbook 09-boot-order.yml --ask-vault-pass --tags set \
  --extra-vars '{"boot_target": "Hdd", "boot_mode": "Continuous"}'
```

## Inventory

Edit `inventory.yml` to add/remove servers. Each host needs:
- `baseuri` — iLO IP address
- Credentials in `secrets.yml` (shared) or per-host overrides

## Safety

- Power and boot playbooks require interactive confirmation
- BIOS writes go to `/Bios/Settings` (pending) — no changes until reboot
- Firmware updates may require reboot to activate
- Destructive tags use `never` — must be explicitly called

## Redfish Resources

- [HPE iLO 5 Redfish API Docs](https://hewlettpackard.github.io/ilo-rest-api-docs/ilo5/)
- [DMTF Redfish Spec](https://www.dmtf.org/standards/redfish)
- [Ansible redfish_info module](https://docs.ansible.com/ansible/latest/collections/community/general/redfish_info_module.html)
- [Ansible redfish_command module](https://docs.ansible.com/ansible/latest/collections/community/general/redfish_command_module.html)

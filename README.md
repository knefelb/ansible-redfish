# Ansible Redfish Playbooks

Ansible playbooks for managing HPE ProLiant servers via the Redfish API (iLO 5).

## Prerequisites

```bash
pip install ansible
ansible-galaxy collection install community.general
```

## Setup

1. Copy and encrypt secrets:
   ```bash
   cp secrets.yml.example secrets.yml
   # Edit with your iLO password
   ansible-vault encrypt secrets.yml
   ```

2. Update `inventory.yml` with your server iLO IPs

## Playbooks

| # | Playbook | Description |
|---|----------|-------------|
| 01 | `inventory-report` | Pull model, serial, BIOS, CPU, RAM from all servers |
| 02 | `health-check` | Validate system, thermal, power, storage health |
| 03 | `power-management` | Graceful shutdown, power on, force restart |
| 04 | `boot-override` | One-time PXE boot, BIOS setup, persistent HDD |
| 05 | `bios-settings` | Read/modify BIOS attributes (pending until reboot) |
| 06 | `firmware-update` | Push firmware via SimpleUpdate |
| 07 | `user-management` | Create, delete, update iLO user accounts |
| 08 | `event-subscriptions` | Set up webhook alerts for hardware events |
| 09 | `virtual-media` | Mount ISOs for remote OS installation |
| 10 | `log-collection` | Pull/clear IML and iLO event logs |

## Usage Examples

```bash
# Fleet inventory report
ansible-playbook playbooks/01-inventory-report.yml --vault-password-file ~/.vault_pass

# Health check all servers
ansible-playbook playbooks/02-health-check.yml --vault-password-file ~/.vault_pass

# Graceful restart a specific server
ansible-playbook playbooks/03-power-management.yml -e "action=GracefulRestart" -l dar-vm01 --vault-password-file ~/.vault_pass

# One-time PXE boot for OS deployment
ansible-playbook playbooks/04-boot-override.yml -e "boot_target=Pxe boot_mode=Once reboot_after=true" -l dar-vm01 --vault-password-file ~/.vault_pass

# Read BIOS settings
ansible-playbook playbooks/05-bios-settings.yml -l dar-vm01 --vault-password-file ~/.vault_pass

# Set BIOS attributes
ansible-playbook playbooks/05-bios-settings.yml -e '{"bios_changes": {"ProcHyperthreading": "Enabled"}, "reboot_after": true}' --vault-password-file ~/.vault_pass

# Mount ISO and boot from it
ansible-playbook playbooks/09-virtual-media.yml -e "iso_url=http://fileserver/ubuntu.iso boot_from_media=true" -l dar-vm01 --vault-password-file ~/.vault_pass

# Collect logs and save to file
ansible-playbook playbooks/10-log-collection.yml -e "save_to=/tmp/logs" --vault-password-file ~/.vault_pass
```

## Inventory

Servers are defined in `inventory.yml`. Each host needs:
- `baseuri` — iLO IP address
- Connection is `local` (API calls run from the control node, not via SSH to iLO)

## Security Notes

- Passwords stored in `ansible-vault` encrypted `secrets.yml`
- All Redfish calls use HTTPS (self-signed certs accepted)
- Create a dedicated read-only iLO user for monitoring playbooks
- Use a privileged user only for power/config/firmware operations

## Related

- [Redfish iLO Study Notes](https://github.com/knefelb/redfish-ilo-study)
- [HPE iLO 5 Redfish API Docs](https://hewlettpackard.github.io/ilo-rest-api-docs/ilo5/)
- [Ansible Redfish Modules](https://docs.ansible.com/ansible/latest/collections/community/general/redfish_info_module.html)

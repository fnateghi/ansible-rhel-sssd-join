# ansible-rhel-sssd-join
Ansible role to join RHEL-based Linux systems to an Active Directory domain.
This role uses realm, sssd, and Kerberos to automate domain joining for RHEL-compatible systems such as Rocky Linux and AlmaLinux.

### Requirements (Managed Node):

- Python 3.7+ is required on the managed node
- Python 3.12 is recommended for full compatibility (fallback to 3.9 if needed)
- The pexpect Python module must be available on the managed node (used for handling interactive realm join)

### Control Node (Ansible Runner):
- This role is designed to be executed from an Ansible environment running in a Python 3.12 virtualenv

---

## Features
- Installs required packages (SSSD, realm, Kerberos, etc.)
- Optionally removes previous SSSD config in ad_prepare.yml
- Joins system to the AD domain using provided credentials
- Configures SSSD with access control via AD groups
- Enables home directory creation using PAM
- Deploys sudoers configuration for AD groups

---

## Directory Structure
```
roles/
└── ad_domain_join/
    ├── defaults/main.yml         # Default variables (domain name, groups, etc.)
    ├── tasks/
    │   ├── main.yml              # Includes other task files
    │   ├── ad_prepare.yml        # Cleanup, install packages
    │   ├── ad_join.yml           # Realm join and SSSD setup
    │   └── ad_configure.yml      # Templates, sudoers, PAM config
    ├── templates/
    │   ├── sssd.conf.j2
    │   ├── krb5.conf.j2
    │   └── sudoers.ad.j2
    ├── files/
    │   ├── nsswitch.conf
    │   └── pam.sss
    └── handlers/main.yml         # Restart sssd if config changes
```

---

## Required Variables
Set these Variables in your inventory or playbook:

```yaml
sssd:
  debuglevel: 0x0780
  domain:
  kdc_servers:
# Dfault OU where the Servers joined. Redefine it in Groups or Hosts if you want them in a different OU.
  computerOU: "OU=Linux,OU=Server,DC=your,DC=domain,DC=test"
  groups_admins:
    - your_AD_Security_Group
  cache_credentials: 'true'
  account_cache_expiration: 2
  offline_credentials_expiration: 2
  entry_cache_timeout: 14400
# Set to true to force a rejoin of the domain.
sssd_force_rejoin: true
# Optional: if you need to specify the computer account OU in full LDAP DN format.
host_sssd_sudo_files: []
host_sssd_groups_sudo: []
# Secure credentials for joining the domain. Provide these via Ansible Vault or extra-vars. Decryption pass --> KeePass
ad_join_user: 
ad_join_pass: 
krb5_libdefaults: []

```

> #### Use Ansible Vault to encrypt credentials. Password to decrypt the vault in KEEPASS!

---

## Example Playbook
```yaml
- name: Join AD Domain
  hosts: linux_clients
  become: true
  roles:
    - ansible-rhel-sssd-join
```

---

## Post-Join Validation
After running the role, test:

```bash
realm list
getent passwd youruser@yourdomain
getent group your_AD_Security_Group
id youruser
sudo whoami
```

---

## Author
Created by **Farhad Nateghi**

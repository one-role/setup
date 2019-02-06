# Encrypted files

This directory contains certificates and encryption keys, which we do
**not** want in Git in a readable way.  These files should be encrypted
with Ansible Vault.

The command to do that is:

```
ansible-vault --vault-password-file=/root/.AnsibleVaultPassword encrypt <file>
```

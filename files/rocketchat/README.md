# Encrypted files

This directory contains certificate public and private keys. We do **not** want
the private keys Git in a readable way.  The `*.key` files should be encrypted
with Ansible Vault.

The command to do that is:

```
ansible-vault --vault-password-file=/root/.AnsibleVaultPassword encrypt <file.key>
```

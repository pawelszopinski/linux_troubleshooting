# SSH Total Lockout (All Authentication Methods Disabled)

## Problem

Lost all SSH access to a Linux server after modifying authentication settings.

Connection attempt:

```bash
ssh user@<server_ip>
```
Output:
```bash
Permission denied, please try again.
```
Even when using the correct password.


**Impact**
- Complete loss of SSH access
- No working authentication method available
- Unable to use ssh-copy-id for recovery
- Requires direct console / VM access

**What I broke**
- Disabled public key authentication
- Removed existing authorized keys
- Password authentication was also disabled
- Changes applied without verifying access

## Quick Setup (minimal)

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh

ssh-keygen
ssh-copy-id user@<server_ip>
ssh user@<server_ip>
```

## Investigation

Check effective configuration:
```bash
sudo sshd -T
```
Relevant output:

```bash
pubkeyauthentication no
passwordauthentication no
kbdinteractiveauthentication no
```


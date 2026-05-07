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

## Root Cause

All authentication methods were disabled:

*Public key authentication*: disabled
*Password authentication*: disabled
*Keyboard-interactive authentication*: disabled

Result:

→ No valid authentication method available
→ SSH login impossible

### Why ssh-copy-id failed
```bash
ssh-copy-id user@<server_ip>
```
Requires working authentication (usually password). Server rejected all authentication attempts.

Result is that public key could not be installed

Process:
1. Connects to the server using SSH.
2. Authenticates (e.g. with password)
3. Appends the public key to 

```bash
~/.ssh/authorized_keys
```
No authentication method working = connection is rejected

## Recovery

Access server/VM via console and restore at leat one authentication method in:

```bash
/etc/ssh/sshd_config
```
Then restart ssh. Login should succed after restoring a valid authentication method.

## Prevention
- always test ssh access before applying changes
- keep at least one working login method

## Key Takeaways
- SSH can be fully locked down by misconfiguration
- authentication methods must be explicitly enabled
- *sshd -T* is useful for veryfing effective configuration
- Tools like ssh-copy-id depend on working authentication
- Misleading prompts (password request) can be distracting

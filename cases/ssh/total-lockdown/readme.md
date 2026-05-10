# SSH Authentication Lockout Incident

## Intent
Harden SSH access by restricting authentication methods (disabling password login and relying solely on SSH keys).

## Problem

After applying SSH security changes, remote access to the server became unavailable.

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


## Investigation

1. Check effective configuration:
```bash
sudo sshd -T
```
Relevant output:

```bash
pubkeyauthentication no
passwordauthentication no
kbdinteractiveauthentication no
```
2. Log analysis

```bash
journalctl -u ssh
journalctl -u ssh -f
```
Findings:

```bash
Connection closed by authenticating user user <ip_address>
```
This indicated:
- daemon was running
- attempts were reaching the erver
- sessions was terminated during auth phase

However, logs at default level (LogLevel INFO) did not provide enough detail to identify the exact cause.
3. Increasing log verbosity

To gain deeper insight, logging level can be increased:

In sshd_config let's change LogLevel:

```bash
LogLevel DEBUG
```
4. Authentication negotiation analysis

Using increased log verbosity revealed authentication attempts:

- publickey  
- keyboard-interactive  
- password  

All methods were attempted by the client but rejected by the server.

This confirmed that:

- SSH connection and handshake were successful  
- authentication phase was reached  
- no authentication methods were allowed by configuration  

**Result: authentication failure was caused by configuration, not connectivity or credentials**

### Logging behavior observation

Increasing LogLevel to VERBOSE did not provide additional insights.

Reason:
- no authentication methods were enabled
- therefore no additional authentication details were available to log

Only DEBUG level revealed the authentication negotiation process.

Conclusion:
Log verbosity usefulness depends on the system state, not only on configuration level. 

### Security consideration – debug logging

Enabling DEBUG2 revealed detailed authentication data, including:

- username
- public key fingerprint used during login

This confirms that high debug levels expose sensitive authentication metadata.

For this reason:
- DEBUG levels should only be used temporarily
- logs should be reviewed and cleaned if necessary after debugging

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

Authentication was failing not because of incorrect credentials, 
but because no authentication methods were allowed by the SSH configuration.
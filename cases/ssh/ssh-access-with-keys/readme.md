# SSH Access with Key Authentication (CentOS ↔ Linux Client)

## Goal

Configure SSH access using key-based authentication and disable password login.

---

## Environment

- Server: Ubuntu 
- Client: CentOS 
- User: user

## Setup 

1. Install and enable SSH on server

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
```

2. Check ip address

```bash
ip a
```
3. Verify SSH configuration

```bash
sudo nano /etc/ssh/sshd_config 
```
Ensure:
```bash 
PasswordAuthentication yes
PubkeyAuthentication yes
```

4. Restart service
```bash 
sudo systemctl restart ssh
```

5. Key based authentication

```bash
ssh-keygen
``` 
6. Check permissions (default when you do ssh-keygen but can vary with manual copying)

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```


7. Copy key to server
```bash
ssh-copy-id user@ip 
```

##  Testing

```bash
ssh user@ip
```
Login should work without password.

## Bonus
1. Disable password authentication 

```bash 
sudo nano /etc/ssh/sshd_config 
```
set PasswordAuthentication no

2. On client
```bash
nano ~/.ssh/config
```

Host <name of your choice>
    Hostname <server's ip address>
    User <username>
    IdentifyFile ~/.ssh/<nameOfKey>

Save and exit. Now you can ssh in by using

```bash
ssh <name of your choice>
```

## What can go wrong? - common issues

### Incorrect permissions
### SSH service not running
### Firewall blocking connection
### Wrong keyfile used
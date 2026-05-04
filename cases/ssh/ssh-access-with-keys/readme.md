# 🔐 SSH Access with Key Authentication (CentOS ↔ Linux Client)

## 📌 Goal

Set up SSH access between two Linux machines using key-based authentication (without password login).

---

## 🧱 Environment

- Server: CentOS VM 
- Client: Ubuntu VM
- User: centos

---

## 🔑 Step 1: Generate SSH key (on client)

```bash
ssh-keygen 

##  Step 2: Verify key exists(on client)

```bash
ls ~/.ssh

Expected values:
nameOfKey and nameOfKey.pub

## 🔑 Step 3: Copy public key via ssh-copy-id

Edit on server

```bash
sudo nano /etc/ssh/sshd_config

```bash
PasswordAuthentication yes
PubkeyAuthentication yes

```bash
sudo systemctl restart sshd
ssh-copy-id user@ip 


## 🔑 Step 4: Test

```bash
ssh user@ip

Working without password!
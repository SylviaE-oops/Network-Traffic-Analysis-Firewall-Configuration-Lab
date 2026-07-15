# Network Security Labs

## Project Overview
This lab focused on capturing and analyzing common network traffic between Windows 11 and Ubuntu virtual machines using Wireshark. I configured virtual networking, verified communication between systems, analyzed several network protocols, and modified firewall rules to observe how traffic changed in real time.

The goal was not only to identify different protocols, but also to understand how operating systems communicate and how network configurations affect connectivity.

## Objectives

- Configure communication between Windows and Ubuntu virtual machines.
- Capture and analyze ICMP traffic with Wireshark.
- Observe how firewall rules affect network communication.
- Capture SSH traffic after configuring an SSH server.
- Analyze DNS queries and responses.
- Practice troubleshooting common networking issues.
- Examine Remote Desktop Protocol (RDP) traffic

## Labs

- Lab 1 – Observe ICMP Traffic
- Lab 2 – Configure Ubuntu Firewall
- Lab 3 – Observe SSH Traffic
- Lab 4 – Observe DNS Traffic


## Key Concepts Covered

- IPv4 Addressing
- Virtual Networking
- NAT vs. Host-Only Adapters
- ICMP Echo Request / Echo Reply
- Packet Capture
- Firewall Rules
- SSH Encryption
- DNS Name Resolution
- Network Troubleshooting
- Protocol Analysis

---

# Lab Workflow

## 1. Configure Virtual Networking

Initially, both virtual machines were connected using VirtualBox's default NAT adapter.

### Windows 11

| Adapter | IPv4 Address | Purpose |
|----------|--------------|---------|
| Ethernet | 192.168.56.101 | Host-Only |
| Ethernet 2 | 10.0.3.15 | NAT |

### Ubuntu

| Adapter | IPv4 Address | Purpose |
|----------|--------------|---------|
| Ethernet | 192.168.56.102 | Host-Only |
| Ethernet 2 | 10.0.3.15 | NAT |

### Network Configuration

- **Host-Only Adapter**
  - Communication between Windows and Ubuntu VMs

- **NAT Adapter**
  - Internet connectivity

Initially, the virtual machines could not communicate because each VM was behind its own virtual NAT router.

To resolve this, I added a Host-Only adapter to both virtual machines while keeping the NAT adapter for Internet access.

After verifying the IP addresses again using:

**Windows**

```powershell
ipconfig
```

**Ubuntu**

```bash
hostname -I
```

both systems were able to successfully communicate with each other.

> 📷 *Insert screenshots of Windows and Ubuntu IP configuration.*

---

# Lab 1: Observe ICMP Traffic

Started Wireshark and applied the following display filter:

```text
icmp
```

### Test 1 — Ping a Public Website

From Windows PowerShell:

```powershell
ping www.google.com
```

Observed:

- ICMP Echo Requests
- ICMP Echo Replies
- Internet traffic using the NAT adapter

> 📷 *Insert Wireshark screenshot.*

---

### Test 2 — Windows to Ubuntu

From Windows:

```powershell
ping 192.168.56.102
```

Observed:

- ICMP Echo Request
- ICMP Echo Reply

showing successful communication between both virtual machines.

> 📷 *Insert screenshot.*

---

### Test 3 — Ubuntu to Windows

From Ubuntu:

```bash
ping 192.168.56.101
```

While capturing packets in Wireshark, I observed Ubuntu continuously sending ICMP Echo Requests.

Initially, Windows received the requests but did not send ICMP Echo Replies back.

This indicated the issue was related to the responding host rather than Ubuntu's `ping` command.

---

#  Lab 2: Configure Firewall

Started a continuous ping from Windows:

```powershell
ping 192.168.56.102 -t
```

Initially every request received a successful reply.

Wireshark showed:

- ICMP Echo Request
- ICMP Echo Reply

---

## Attempt 1 — UFW

Checked firewall status:

```bash
sudo ufw status
```

Edited:

```text
/etc/ufw/before.rules
```

Commented out:

```text
-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
```

Reloaded UFW:

```bash
sudo ufw disable
sudo ufw enable
```

After testing, ICMP traffic was still allowed.

This method did not produce the expected behavior.

---

## Attempt 2 — iptables

Used:

```bash
sudo iptables -I INPUT -p icmp --icmp-type echo-request -j DROP
```

Immediately, the Windows continuous ping changed to:

```text
Request timed out.
```

Wireshark showed ICMP Echo Requests leaving Windows, but Ubuntu no longer replied because the firewall dropped the packets before they reached the operating system.

To restore connectivity:

```bash
sudo iptables -D INPUT -p icmp --icmp-type echo-request -j DROP
```

> 📷 *Insert screenshots.*

---

# Lab 3: Observe SSH Traffic

Initially, SSH connections failed.

Checking the service:

```bash
sudo systemctl status ssh
```

showed that the SSH server was not installed.

Installed OpenSSH:

```bash
sudo apt update
sudo apt install openssh-server
```

Enabled the service:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

Successfully connected from Windows to Ubuntu.

Executed several Linux commands:

```bash
whoami
pwd
ls
hostname
ip addr
```

Exited the session after testing.

---

### Wireshark Observations

SSH traffic was visible in Wireshark, but the actual commands and responses were encrypted.

Unlike ICMP or DNS traffic, SSH protects the contents of the communication, making only packet metadata (such as TCP segments and packet sizes) visible.

> 📷 *Insert Wireshark screenshot.*

---

# Lab 4: Observe DNS Traffic

While capturing packets, I generated DNS traffic by accessing:

- google.com
- disney.com

Observed:

- DNS Standard Query
- DNS Standard Response
- Returned IPv4 addresses

Unlike SSH traffic, DNS queries and responses were readable in plain text, allowing me to identify the requested domain names and their corresponding IP addresses.

> 📷 *Insert Wireshark screenshot.*

---

# Troubleshooting Documentation

| Problem | Investigation | Resolution |
|----------|--------------|------------|
| Windows and Ubuntu could not communicate | Both VMs were connected through separate NAT networks | Added a Host-Only adapter to each VM |
| Lost Internet connectivity after changing adapters | Host-Only provides local communication only | Added a second NAT adapter |
| Windows did not respond to Ubuntu pings | Examined ICMP packets in Wireshark | Determined Windows was not sending Echo Replies |
| UFW rule change did not block ICMP | Reloaded UFW and tested again | Used an iptables rule instead |
| SSH connection failed | Checked SSH service status | Installed and started OpenSSH Server |

---

# What I Learned

- How VirtualBox networking modes affect communication between virtual machines.
- The difference between NAT and Host-Only adapters.
- How ICMP Echo Requests and Echo Replies appear in packet captures.
- How firewall rules influence network communication.
- The difference between UFW configuration and temporary iptables rules.
- How SSH encrypts application traffic.
- How DNS resolves domain names into IP addresses.
- How Wireshark can be used to troubleshoot connectivity issues.

---

# IT Support Relevance

This lab simulates several common IT support and systems administration tasks, including:

- Verifying network connectivity.
- Troubleshooting communication between devices.
- Configuring virtual machine networking.
- Capturing and analyzing network traffic.
- Installing and configuring SSH services.
- Testing firewall configurations.
- Understanding DNS resolution.
- Using packet captures to identify networking issues.

These are practical skills commonly used in Help Desk, Desktop Support, Junior Systems Administration, and Network Support roles.

---

# Skills Demonstrated

- Network Troubleshooting
- Wireshark Packet Analysis
- ICMP Analysis
- DNS Analysis
- SSH Configuration
- Linux Administration
- Windows PowerShell
- VirtualBox Networking
- Firewall Configuration (UFW & iptables)
- IPv4 Networking
- TCP/IP Fundamentals
- Packet Filtering
- Root Cause Analysis
- Technical Documentation

---
# Author

**Asibong (Sylvia) Ephraim**  
IT Analyst & IT Support Specialist

- 💼 **LinkedIn:** https://www.linkedin.com/in/asibong-ephraim
- 🌐 **Portfolio:** https://github.com/SylviaE-oops
- 📧 **Email:** ephraimasibong@gmail.com

---


# Network Traffic Analysis Lab

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
| Ethernet | 192.168.xx.xxx | Host-Only |
| Ethernet 2 | 10.0.x.xx | NAT |

### Ubuntu

| Adapter | IPv4 Address | Purpose |
|----------|--------------|---------|
| Ethernet | 192.168.xx.xxx | Host-Only |
| Ethernet 2 | 10.0.x.xx | NAT |

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


<img width="494" height="186" alt="ping www google com " src="https://github.com/user-attachments/assets/962e5a85-bbfe-4f55-8441-ff5ef533120a" />

<img width="643" height="448" alt="reply and responds from ping www google com" src="https://github.com/user-attachments/assets/17b9d535-bdea-459d-97a9-d5f3ee796389" />



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

<img width="443" height="212" alt="ping from windows11 to linux " src="https://github.com/user-attachments/assets/866e4b36-d65e-42a4-8877-ef9d0301e6c0" />


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

<img width="418" height="335" alt="Lab 2 - continuous pings from Windows 11" src="https://github.com/user-attachments/assets/83d9d114-2664-4f07-a7d5-e170b67ebe0e" />


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

<img width="346" height="83" alt="Lab 2 - time in and time out" src="https://github.com/user-attachments/assets/aaade671-e4be-419b-962e-eb95911e45f4" />
<img width="460" height="417" alt="Lab 2 - time out" src="https://github.com/user-attachments/assets/a9d23c97-d620-4a6f-b800-a06cdba1c895" />
<img width="453" height="384" alt="Lab 2 - timed out comes back " src="https://github.com/user-attachments/assets/dcc98323-6558-4cd7-9198-df8282570d16" />


---

# Lab 3: Observe SSH Traffic

Initially, SSH connections failed.

<img width="627" height="39" alt="Lab 3 - ssh doesn&#39;t connect" src="https://github.com/user-attachments/assets/f6de8cd4-b3f4-4d96-b2e0-44303d8bb3db" />


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

<img width="643" height="475" alt="Lab 3 - wireshark from ssh" src="https://github.com/user-attachments/assets/3af09282-649a-4cd3-bba8-9d30233063a7" />


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

<img width="401" height="470" alt="Lab 4 - powershell" src="https://github.com/user-attachments/assets/dd41ad39-db90-42c3-a677-7e5cec1c8744" />

<img width="641" height="477" alt="Lab 4 - wireshark observe" src="https://github.com/user-attachments/assets/a14d1362-6530-4387-8372-7c6c17f6aa34" />


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


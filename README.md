# Secure iPhone Photo Backup: Self-Hosted Personal Cloud (iCloud Replacement)

Tired of Apple's storage limits, slow backups, and privacy concerns? I built a fast, private "home cloud" using Owlfiles on iPhone + Tailscale VPN + Ubuntu server on an Intel NUC with RAID storage.

**Result**: Upload 2,000+ photos/videos directly from iPhone to my secured personal RAID in **under 5 minutes** — anywhere, encrypted, no subscriptions or vendor lock-in.

This homelab demonstrates secure remote access, zero-trust networking, encrypted file transfers, and private cloud storage — skills I apply in my GRC/cloud security work.

## Table of contents
- [Why I Built This](#why-i-built-this)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
  - [1. Install & Connect Tailscale (All Devices)](#1-install--connect-tailscale-all-devices)
  - [2. Create Storage Folder on Server](#2-create-storage-folder-on-server)
  - [3. SFTP Remote File Access (Main Method for Uploads)](#3-sftp-remote-file-access-main-method-for-uploads)
  - [4. Samba (SMB) Share (Optional Fallback)](#4-samba-smb-share-optional-fallback)
  - [5. GUI Remote Access (RDP via xrdp + Tailscale)](#5-gui-remote-access-rdp-via-xrdp--tailscale)
  - [6. Security Hardening](#6-security-hardening)
- [Results & Screenshots](#results--screenshots)
- [Lessons Learned](#lessons-learned)
- [Author](#author)
- [License](#license)
- [Contributing](#contributing)

## Why I Built This

After years of iPhone photo backup struggles:

- **2019**: EaseUS MobiMover transferred 3,000+ files in <30 min (free version worked then).
- **2023**: Owlfiles connected iPhone to local Windows network folders — super fast uploads.
- **Now**: Added remote access via Tailscale VPN + Ubuntu server/RAID on an old Intel NUC.
- **Breakthrough**: Owlfiles + SFTP over Tailscale = direct, secure uploads from anywhere.

This solved my photo space issues better than iCloud — faster, private, and good for testing cybersecurity concepts.

## Features

- Secure remote file access via Tailscale (zero-trust VPN)
- Direct iPhone uploads to home RAID storage (via Owlfiles SFTP/SMB)
- GUI management via RDP (xrdp) for easy folder setup
- Privacy-focused: No third-party cloud risks, full control
- Encrypted end-to-end (Tailscale WireGuard + SSH/SFTP/RDP TLS)

## Tech Stack

- Owlfiles (iOS file manager — supports SFTP, SMB, WebDAV)
- Tailscale VPN (secure mesh network, no port forwarding)
- Ubuntu Server on Intel NUC
- RAID array for reliable storage (e.g., mdadm)
- SFTP (over SSH) for file transfers
- xrdp for RDP GUI access
- Samba (SMB) as optional fallback

## Prerequisites

- Intel NUC (or similar) with Ubuntu Server installed
- RAID configured (e.g., software RAID via mdadm)
- Tailscale account (free tier OK)
- Owlfiles app on iPhone
- RDP client (Microsoft Remote Desktop, Remmina, etc.)

## Setup Guide

**Note:** This guide uses SFTP as the primary upload method.

### 1. Install & Connect Tailscale (All Devices)

On Ubuntu server & Linux laptop:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

On iPhone: install the Tailscale app and sign in to the same tailnet. Note the server's Tailscale IPv4 address:

```bash
tailscale ip -4
# Example: 100.x.x.x
```

### 2. Create Storage Folder on Server

```bash
sudo mkdir -p /srv/photos
sudo chown -R $USER:$USER /srv/photos
sudo chmod -R 0775 /srv/photos
```

(If you run services under a dedicated user, set the appropriate owner instead of $USER.)

### 3. SFTP Remote File Access (Main Method for Uploads)

SFTP over SSH (port 22) + Tailscale = encrypted, secure transfers.

Install SSH:

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

Harden SSH (edit `/etc/ssh/sshd_config`):

- Set `PermitRootLogin no`
- Set `PasswordAuthentication no` (use SSH keys)

After editing, restart SSH:

```bash
sudo systemctl restart ssh
```

Generate an SSH key on your laptop and copy it to the server (replace username and IP):

```bash
ssh-keygen -t ed25519
ssh-copy-id yourusername@100.x.x.x
```

Configure Owlfiles (iPhone):

- Add Connection → **SFTP**  
  - Host: Tailscale IP (e.g., 100.x.x.x)
  - Port: 22
  - Username: your Ubuntu user
  - Auth: SSH key (or password if you haven't disabled password auth)
  - Path: /srv/photos

IMPORTANT: When adding a connection in Owlfiles, selecting **SFTP** as the connection type is crucial. Owlfiles offers SMB and WebDAV too, but a plain Ubuntu server running OpenSSH will only accept SFTP. If you pick SMB or WebDAV in Owlfiles without configuring the corresponding server-side services (Samba for SMB, a WebDAV server for WebDAV), the connection will fail on Ubuntu. For this guide, always use SFTP.

Upload photos/videos — fast & encrypted.

### 4. Samba (SMB) Share (Optional Fallback)

For local or VPN-based SMB access:

Install Samba:

```bash
sudo apt update
sudo apt install samba -y
```

Add to `/etc/samba/smb.conf` (append):

```
[photos]
   path = /srv/photos
   browsable = yes
   writable = yes
   valid users = yourusername
```

Add Samba user and restart:

```bash
sudo smbpasswd -a yourusername
sudo systemctl restart smbd
```

Connect in Owlfiles: SMB → Host: Tailscale IP → Port: 445 → Username/password

Note: SMB over the public internet is not recommended without a VPN (we use Tailscale).

### 5. GUI Remote Access (RDP via xrdp + Tailscale)

Install a lightweight desktop and xrdp:

```bash
sudo apt update
sudo apt install xfce4 xfce4-goodies -y
sudo apt install xrdp -y
sudo adduser xrdp ssl-cert
echo xfce4-session > ~/.xsession
sudo systemctl restart xrdp
```

If you use UFW and want to restrict RDP to the Tailscale range:

```bash
sudo ufw allow from 100.64.0.0/10 to any port 3389
sudo ufw deny 3389
sudo ufw reload
```

Connect using an RDP client to the server's Tailscale IP and your Ubuntu credentials.

### 6. Security Hardening

- Zero public exposure: avoid port forwarding; use Tailscale for remote access.
- Firewall: block/expose only necessary ports and restrict to Tailscale address space.
- Use SSH keys and disable password auth.
- Consider enabling Tailscale ACLs to restrict which devices can access the server.
- Keep system updated:

```bash
sudo apt update && sudo apt upgrade -y
```

- xrdp uses TLS; enable NLA if your client supports it.

## Results & Screenshots

- Speed: 2,000+ files in <5 min via SFTP over Tailscale.
- Privacy: Full control, no Apple/Google involvement.

(Add screenshots here: Owlfiles uploading, RDP session, Tailscale connected, speed test, architecture diagram. Use GitHub uploads or external links.)

## Lessons Learned

- Tailscale makes remote homelabs simple & secure (zero-trust in action).
- Self-hosting beats vendor lock-in for data privacy in many use-cases.
- Combining Owlfiles + SFTP/RDP gives flexible, fast access.

## Author

Built by Wesley Cain III — Cybersecurity Analyst (GRC & Cloud Security)  
LinkedIn: https://www.linkedin.com/in/wesley-cain-iii-a6b0a6224/

## License

MIT — see LICENSE file.

## Contributing

Feel free to fork, adapt, or open issues/PRs. If you submit commands or configuration changes, please include notes on the OS and version you tested on.
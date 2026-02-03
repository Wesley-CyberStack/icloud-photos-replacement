#Introduction to my repo

After years of struggling to backup my iPhone photos, I finally found a solution. The first thing that seemed to actually work for me was an app called EaseMobiMover(or something along those lines). A while back, in 2019, you were able to get away with using it for free. I couldn’t believe how quickly it uploaded my photos. It constantly asked me to subscribe to the full version, but I was able to make it work for free. It blew my mind how much quicker it was than any photo import I had ever done. 3,000+ photos and videos in less than 30 minutes. Crazy. I knew there had to be a better way. Back in 2023, I found Owlfiles. I found out that using Owlfiles, I could connect to my Windows PC’s local folders (once I created a network folder) and upload my files there. WOW, so fast. From there, I wondered if it was possible to set this up remotely. 

Fast forward to now, I have been playing with VM’s and VPN’s, so that I can access my home lab remotely, and run security test and get real-time alerts. That’s when it clicked. I wonder if this VPN can work in conjunction with Owlfiles. That’s when I decided to build an at home RAID array in compatablity with an ubuntu-linux-server(on an Intel Nuk that I had laying around.) Turns out; IT WORKS BETTER THAN I THOUGHT. 2,000+ photos and videos off my iPhone, straight to my “personal, private, home cloud(secured)" in less than FIVE MINUTES.

# Secure iPhone Photo Backup: Self-Hosted Personal Cloud (iCloud Replacement)

Tired of Apple's storage limits, slow backups, and privacy concerns? I built a fast, private "home cloud" using Owlfiles on iPhone + Tailscale VPN + Ubuntu server on an Intel NUC with RAID storage.

**Result**: Upload 2,000+ photos/videos directly from iPhone to my secured personal RAID in **under 5 minutes** — anywhere, encrypted, no subscriptions or vendor lock-in.

This homelab demonstrates secure remote access, zero-trust networking, encrypted file transfers, and private cloud storage — skills I apply in my GRC/cloud security work.

## Why I Built This

After years of iPhone photo backup struggles:

- **2019**: EaseUS MobiMover transferred 3,000+ files in <30 min (free version worked then, despite upsells).
- **2023**: Owlfiles connected iPhone to local Windows network folders — super fast uploads.
- **Now**: Added remote access via Tailscale VPN + Ubuntu server/RAID on old Intel NUC.
- **Breakthrough**: Owlfiles + SFTP over Tailscale = direct, secure uploads from anywhere.

This solved my photo space issues better than iCloud — faster, private, and great for testing real cybersecurity concepts.

## Features

- Secure remote file access anywhere via Tailscale (zero-trust VPN)
- Direct iPhone uploads to home RAID storage (via Owlfiles SFTP/SMB)
- GUI management via RDP (xrdp) for easy folder setup
- Privacy-focused: No third-party cloud risks, full control
- Encrypted end-to-end (Tailscale WireGuard + SSH/SFTP/RDP TLS)

## Tech Stack

- **Owlfiles** (iOS file manager — supports SFTP, SMB, WebDAV)
- **Tailscale VPN** (secure mesh network, no port forwarding)
- **Ubuntu Server** on Intel NUC
- **RAID array** for reliable storage
- **SFTP** (over SSH port 22) for file transfers
- **xrdp** for RDP GUI access
- **Samba** (SMB) as optional fallback

## Prerequisites

- Intel NUC (or similar) with Ubuntu Server installed
- RAID configured (e.g., software RAID via mdadm)
- Tailscale account (free tier works)
- Owlfiles app on iPhone
- RDP client (Microsoft Remote Desktop, Remmina, etc.)

## Setup Guide

### 1. Install & Connect Tailscale (All Devices)

On Ubuntu server & Linux laptop:
(```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up)
On iPhone: Install Tailscale app, log in to same tailnet.
Note server's Tailscale IP (run tailscale ip -4 → e.g., 100.x.x.x)
2. Create Storage Folder on Server
Bashsudo mkdir -p /srv/photos
sudo chown -R $USER:$USER /srv/photos
sudo chmod -R 0775 /srv/photos
3. SFTP Remote File Access (Main Method for Uploads)
SFTP over SSH (port 22) + Tailscale = encrypted, secure transfers.

Install SSH:Bashsudo apt install openssh-server -y
sudo systemctl enable --now ssh
Harden SSH config (sudo nano /etc/ssh/sshd_config):BashPermitRootLogin no
PasswordAuthentication no  # Use keys insteadRestart:Bashsudo systemctl restart ssh
Set up key auth (from laptop):Bashssh-keygen -t ed25519
ssh-copy-id yourusername@100.x.x.x  # Tailscale IP
Connect from Owlfiles (iPhone):
Add Connection → SFTP
Host: Tailscale IP (100.x.x.x)
Port: 22
Username: your Ubuntu user
Auth: SSH key (or password if not using keys)
Path: /srv/photos
→ Upload photos/videos — fast & encrypted!


4. Samba (SMB) Share (Optional Fallback)
For local or VPN-based SMB access:

Install:Bashsudo apt install samba -y
Edit /etc/samba/smb.conf (add at bottom):text[photos]
path = /srv/photos
browsable = yes
writable = yes
valid users = yourusername
Add user:Bashsudo smbpasswd -a yourusername
Restart:Bashsudo systemctl restart smbd
Connect in Owlfiles: SMB → Host: Tailscale IP → Port 445 → Username/password

5. GUI Remote Access (RDP via xrdp + Tailscale)
For easy visual folder/Samba setup:

Install lightweight desktop:Bashsudo apt install xfce4 xfce4-goodies -y
Install xrdp:Bashsudo apt install xrdp -y
sudo adduser xrdp ssl-cert
Set session:Bashecho xfce4-session > ~/.xsession
Restart:Bashsudo systemctl restart xrdp
Firewall (Tailscale-only):Bashsudo ufw allow from 100.64.0.0/10 to any port 3389
sudo ufw deny 3389
sudo ufw reload
Connect: Use RDP client → Server: Tailscale IP (100.x.x.x) → Port 3389 → Your Ubuntu username/password

6. Security Hardening

Zero public exposure: Tailscale encrypts everything (WireGuard); no port forwarding.
Firewall: Block external ports 22/3389/445; allow only Tailscale range (100.64.0.0/10).
Use SSH keys (disable password auth).
Tailscale ACLs: Restrict to your devices only.
Keep system updated: sudo apt update && sudo apt upgrade
xrdp uses TLS; enable NLA if possible.

Results & Screenshots

Speed: 2,000+ files in <5 min via SFTP over Tailscale.
Privacy: Full control, no Apple/Google involvement.

(Add screenshots here: Owlfiles uploading, RDP session, Tailscale connected, speed test, architecture diagram. Use GitHub upload or external links.)
Lessons Learned

Tailscale makes remote homelabs simple & secure (zero-trust in action).
Self-hosting beats vendor lock-in for data privacy.
Combining Owlfiles + SFTP/RDP gives flexible, fast access.

Built by Wesley Cain III — Cybersecurity Analyst (GRC & Cloud Security)
LinkedIn: https://www.linkedin.com/in/wesley-cain-iii-a6b0a6224/
Feel free to fork, adapt, or open issues/PRs!
MIT License — see LICENSE file.

## Project: Hardened AWS Cloud Infrastructure & Secure Mail Server
Author: Singh
Focus: Infrastructure Hardening, Nginx Security, Postfix/DKIM Implementation

### Project Overview
This project documents the end-to-end deployment and hardening of an Amazon EC2 instance (Rocky Linux/Amazon Linux 2023). The goal was to build a "Security-First" environment that hosts a secure Nginx web server and a fully authenticated Postfix Mail Transfer Agent (MTA).

### Phase 1: OS & Network Hardening
Before installing services, the base OS was hardened to reduce the attack surface.

SSH Security: Disabled password authentication and root login. Migrated to Ed25519 public-key authentication only.

Firewall Configuration: Implemented firewalld with a strict "deny-by-default" policy, only allowing ports 22, 80, 443, 25, and 587.

Kernel Hardening: Created a custom sysctl policy to prevent common network attacks:

TCP SYN Cookies: Enabled to mitigate SYN Flood (DoS) attacks.

Source Routing: Disabled to prevent IP spoofing.

Intrusion Prevention: Configured Fail2Ban to automatically jail IPs exhibiting brute-force behavior.

Resource Management: Established a 2GB Swap file to prevent Out-of-Memory (OOM) errors on T-series micro instances.

### Phase 2: Secure Web Deployment (Nginx)
The Nginx installation focuses on Information Obscurity and Transport Layer Security (TLS).

Banner Grabbing Protection: Disabled server_tokens to hide Nginx versioning from scanners.

Browser Security Headers: Integrated X-Frame-Options and X-Content-Type-Options to protect against clickjacking and MIME-sniffing.

Automated SSL/TLS: Deployed Certbot (Let's Encrypt) with a custom systemd timer for automated 90-day renewals.

SSL Hardening: Restricted protocols to TLSv1.2 and TLSv1.3 only; enabled HSTS (Strict Transport Security).

### Phase 3: Authenticated Mail Server (Postfix + DKIM)
To achieve a 10/10 deliverability score, a comprehensive cryptographic authentication chain was implemented.

MTA Hardening: Configured Postfix to listen only on localhost for internal mail origination to prevent open-relay exploitation.

DNS Security Suite:

MX Records: Defined mail exchange priority in Route 53.

SPF: Created a Sender Policy Framework record to authorize only the specific EC2 IP.

DKIM (OpenDKIM): Generated RSA keys and established a "Milter" bridge to sign outgoing mail cryptographically.

DMARC: Implemented a reporting policy to monitor alignment between SPF and DKIM.

### Technical Skills Demonstrated
AWS: EC2, Route 53, IAM Policy Creation (Least Privilege), Security Groups.

Linux Admin: Rocky/Amazon Linux, Systemd, DNF, Log Analysis.

Security: Cryptography (RSA/Ed25519), Perimeter Defense, Kernel Tuning.

### Sanitized Command Reference
#### SSH Configuration (/etc/ssh/sshd_config.d/99-custom-security.conf)
Bash
PermitRootLogin no
PasswordAuthentication no
MaxAuthTries 3
#### Kernel Hardening (/etc/sysctl.d/99-hardening.conf)
Bash
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.tcp_syncookies = 1
#### Nginx Security Headers (/etc/nginx/conf.d/security.conf)
Nginx
server_tokens off;
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
ssl_protocols TLSv1.2 TLSv1.3;

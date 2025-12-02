🔐 Ubuntu 20.04 Hardening Script – Conduro Style :

⚠ Important Notes
This script is meant only for freshly installed Ubuntu Server 20.04 systems.

Avoid running on servers with existing firewall or logging configurations, as it may cause conflicts or service disruptions.

Always create backups and test on non-production systems first.

Some steps (like disabling logs) might impact your ability to audit or troubleshoot.

🛠 Setup Instructions
Run this single command to download and execute the hardening script:

bash
Copy
Edit
wget -O ./install.sh https://condu.ro/install.sh && chmod +x ./install.sh && sudo ./install.sh
Fetches the latest version.

Makes it executable.

Runs it with root privileges.

🚀 What the Script Does: Detailed Breakdown
1. 📦 Install Core Utilities
```bash
apt-get install wget sed git -y
```

Installs essential CLI tools:

wget: downloads files over HTTP(S).

sed: stream editor for file manipulation.

🔄 Fully Update the System:

```bash
apt-get update -y && apt-get full-upgrade -y
```
Updates package lists.

Upgrades installed packages to latest stable versions.

Patches vulnerabilities and bugs.

Ensures system stability and security.

. 🧰 (Optional) Install Latest Golang:

```bash
rm -rf /usr/local/go
wget -q -c https://dl.google.com/go/$(curl -s https://golang.org/VERSION?m=text).linux-amd64.tar.gz -O go.tar.gz
tar -C /usr/local -xzf go.tar.gz
echo "export GOROOT=/usr/local/go" >> /etc/profile
echo "export PATH=/usr/local/go/bin:$PATH" >> /etc/profile
source /etc/profile
rm go.tar.gz
```
Removes previous Go installations.

Installs latest Go (useful for dev tools).

Adds Go to system environment variables.

4. 🌐 Set Cloudflare DNS for Faster, Private Name Resolution

```bash
truncate -s0 /etc/resolv.conf
echo "nameserver 1.1.1.1" | tee -a /etc/resolv.conf
echo "nameserver 1.0.0.1" | tee -a /etc/resolv.conf
```

Uses Cloudflare DNS (1.1.1.1, 1.0.0.1).

Faster and more privacy-respecting than typical ISP DNS.

5. ⏲ Configure NTP (Network Time Protocol)
```bash
truncate -s0 /etc/systemd/timesyncd.conf
echo -e "[Time]\nNTP=time.cloudflare.com\nFallbackNTP=ntp.ubuntu.com" | tee -a /etc/systemd/timesyncd.conf
```
Syncs system time with Cloudflare’s NTP server, falling back to Ubuntu’s.

Accurate time critical for:

SSL certificates

Logs and auditing

Scheduled jobs

🧱 Harden Kernel Network Settings (sysctl.conf)

```bash
wget -q -c https://raw.githubusercontent.com/conduro/ubuntu/main/sysctl.conf -O /etc/sysctl.conf
sysctl -p
```

Applies security tweaks:

Protects against IP spoofing.

Blocks ICMP flood attacks.

Disables source routing.

Disables IPv6 system-wide.

Logs suspicious packets (martians).

Restricts kernel pointer info leaks.

Enables panic on out-of-memory conditions.

 🔐 Secure SSH Daemon Configuration

```bash
wget -q -c https://raw.githubusercontent.com/conduro/ubuntu/main/sshd.conf -O /etc/ssh/sshd_config
service ssh restart
```
Disables password authentication; uses SSH keys only.

Limits login attempts (MaxAuthTries 3).

Disables X11 forwarding, agent forwarding, environment forwarding.

Enables PAM authentication.

Blocks empty passwords and Kerberos.

Logs SSH access attempts.

Allows optional SSH port customization.


🔕 Disable System Logging Services (Optional)

```bash
systemctl stop systemd-journald.service && systemctl disable --now systemd-journald.service
systemctl stop rsyslog.service && systemctl disable --now rsyslog.service
```

Stops and disables journald and rsyslog to free resources.

Warning: Disabling logs removes audit trails, which may complicate troubleshooting.

9. 🔥 Configure Firewall (UFW)

```bash
ufw disable
echo "y" | ufw reset
ufw logging off
ufw default deny incoming
ufw default allow outgoing
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 22/tcp   # Change to custom port if needed
ufw --force enable
```

Resets UFW to defaults.

Blocks all incoming by default except:

HTTP (80)

HTTPS (443)

SSH (22 or custom)

Turns off verbose logging to save space.

Disable IPv6 firewalling and kernel support:

```bash
sed -i "/ipv6=/Id" /etc/default/ufw
echo "IPV6=no" | tee -a /etc/default/ufw
sed -i "/GRUB_CMDLINE_LINUX_DEFAULT=/Id" /etc/default/grub
echo 'GRUB_CMDLINE_LINUX_DEFAULT="ipv6.disable=1 quiet splash"' | tee -a /etc/default/grub
update-grub2
```

Fully disables IPv6 networking system-wide.

🧹 Clean Up Old Logs & Unused Packages

```bash
find /var/log -type f -delete
rm -rf /usr/share/man/*
apt-get autoremove -y && apt-get autoclean -y
```

Deletes all old logs.

Removes manual pages (optional).

Cleans orphaned packages and package cache

🔄 Reload Configurations and Services

```bash
sysctl -p
update-grub2
systemctl restart systemd-timesyncd
ufw --force enable
service ssh restart
```

📝 Final Recommendations:

Backup your system before running.

Test all services post-hardening to ensure availability.

Use SSH key-based authentication only.

Monitor firewall and logs closely after deployment.

For production, consider keeping logs enabled or forwarding to centralized logging.

Keep the system regularly updated for security patches.











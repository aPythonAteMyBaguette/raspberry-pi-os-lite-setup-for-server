Raspberry Pi Initial Setup Guide
================================

1 - Create new sudo user (adjust username )
------------------------
sudo adduser --allow-bad-names username
sudo usermod -aG sudo username
# Check if sudo user
groups username

2 - Delete pi user and all associated folders
---------------------------------------------
sudo deluser --remove-home pi
# On new username, test if sudo works by:
sudo whoami  # should return 'root'
# Check if pi folders deleted
ls /home
# Check if pi user deleted
sudo find / -user pi

3 - Update the OS
-----------------
sudo apt update && sudo apt upgrade -y
sudo apt autoremove

4 - Disable SSH for pi user (even if deleted)
---------------------------------------------
sudo nano /etc/ssh/sshd_config
# Add the line:
DenyUsers pi
sudo systemctl restart ssh

5 - Disable SSH for root user
-----------------------------
sudo nano /etc/ssh/sshd_config
# Find and set:
PermitRootLogin no
sudo systemctl restart ssh

6 - Install and configure UFW
-----------------------------
sudo apt update
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
# Allow all connections from local network (adjust subnet, here it is 192.168.XX.0/24)
sudo ufw allow from 192.168.XX.0/24
# Or limit SSH connections from local network to reduce attacks
sudo ufw limit proto tcp from 192.168.XX.0/24 to any port 22
# Enable UFW with automatic confirmation
echo "y" | sudo ufw enable
sudo ufw logging medium  # or LOW
sudo ufw status verbose  # optional to verify

6bis - Disable Avahi (mDNS for LAN, optional)
---------------------------------------------
sudo systemctl disable --now avahi-daemon
sudo systemctl disable --now avahi-daemon.socket
sudo systemctl mask avahi-daemon
# Check status:
systemctl is-enabled avahi-daemon avahi-daemon.socket
systemctl is-active avahi-daemon avahi-daemon.socket
# To enable later if needed:
sudo systemctl enable --now avahi-daemon

7 - Disable Wi-Fi adapter (optional)
------------------------------------
sudo nano /boot/config.txt
# Add the line:
dtoverlay=disable-wifi
sudo reboot

# Temporary Wi-Fi disable (until reboot):
sudo ifconfig wlan0 down

# Re-enable Wi-Fi:
sudo ifconfig wlan0 up

8 - Enable automatic system updates
-----------------------------------
sudo apt update
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades  # enable and press yes

# Configure automatic upgrades:
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
# Ensure Allowed Origins include (to be adjusted according to your preferences):
Unattended-Upgrade::Allowed-Origins {
  "${distro_id}:${distro_codename}-security";
  "${distro_id}:${distro_codename}-updates";
};

# Optional automatic reboot and cleanup (to be adjusted according to your preferences):
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::SyslogEnable "true";
Unattended-Upgrade::Verbose "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";

# Set update frequency (to be adjusted according to your preferences):
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
# Set:
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";

# Check timers:
systemctl list-timers '*apt*'

# Test unattended-upgrades dry run:
sudo unattended-upgrades --dry-run --debug

# Check if reboot needed:
cat /var/run/reboot-required || true

# View upgrade logs:
journalctl -u unattended-upgrades -n 200 --no-pager

9 - Install ClamAV Antivirus (optional)
---------------------------------------
sudo apt install clamav clamav-daemon
sudo freshclam

# Configure if needed:
sudo nano /etc/clamav/clamd.conf

sudo systemctl start clamav-daemon
sudo systemctl enable clamav-daemon

# Scan whole system:
clamscan -r /

# Scan specific file or folder:
clamscan /path/to/file_or_folder

10 - Set automatic reboot periodically (optional)
-------------------------------------------------
sudo crontab -e
# Add the line:
0 5 * * 2,5 /sbin/reboot
# Explanation:
# 0 5 - at 5:00 AM
# * * - every day of the month
# 2,5 - Tuesday(2) and Friday(5)

11 - Watchdog Timer (optional)
------------------------------
# Raspberry Pi hardware watchdog to auto reboot on crash
# See dedicated documentation folder for setup instructions

12 - Optional Monitoring Tools Installation
-------------------------------------

# Install htop for interactive process monitoring
sudo apt install htop

# Run htop to view CPU, memory, and process usage
htop

# Install iotop for real-time disk I/O monitoring
sudo apt install iotop

# Run iotop (requires sudo) to monitor disk activity
sudo iotop

# Install vnstat for network traffic monitoring
sudo apt install vnstat

# Enable and start vnstat daemon (auto-starts on boot)
sudo systemctl enable vnstat
sudo systemctl start vnstat

# View network traffic summary
vnstat

# For real-time network monitoring
vnstat -l

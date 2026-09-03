# Personal Linux & Security CLI Reference

## 1. Service & Firewall Operations
* **Check service status:** `sudo systemctl status <service>`
* **Restart a service:** `sudo systemctl restart <service>`
* **Enable + start immediately:** `sudo systemctl enable --now <service>`
* **Check firewall rules:** `sudo ufw status verbose`

## 2. SSH Management & Hardening
* **Test SSH config syntax (typo check):** `sudo sshd -t`
* **Dump active SSH config (truth check):** `sudo sshd -T`
* **Copy SSH key to remote server:** `ssh-copy-id -i ~/.ssh/<key.pub> <user>@<IP>`
* **Force test key failure (bypass keys):** `ssh -i /dev/null -o PubkeyAuthentication=no <user>@<IP>`

## 3. Log Parsing & Threat Hunting
* **View SSH logs (last 20 lines):** `sudo journalctl -u ssh -n 20`
* **Follow live SSH logs in real time:** `sudo journalctl -u ssh -f`
* **Search logs for pattern:** `sudo journalctl -u ssh | grep "Invalid user"`

## 4. Fail2ban IPS Control
* **Check jail status & banned IPs:** `sudo fail2ban-client status sshd`
* **Unban a host IP:** `sudo fail2ban-client set sshd unbanip <IP>`
* **Manually ban an IP:** `sudo fail2ban-client set sshd banip <IP>`

## 5. File Permissions & Ownership
* **Set file permissions to user read/write only (600):** `chmod 600 <file>`
* **Set directory permissions to user access only (700):** `chmod 700 <directory>`

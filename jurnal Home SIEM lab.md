# Home SIEM Lab — Wazuh: Build Journal

I document every step and every difficulty I ran into while building this lab.

## 1. Installing Ubuntu and connecting over SSH

I installed Ubuntu on VirtualBox, then tried to connect with SSH.

- **Problem:** `Connection refused`, because the SSH server was not running.
- **Cause:** the OpenSSH server package was not installed.
- **Solution:** install the OpenSSH server and check its status:

```bash
sudo apt install openssh-server -y
sudo systemctl status ssh
```

## 2. Installing the Wazuh server

I ran the Wazuh server installation command, but the installation hung with no progress.

- **Problem:** the installation stopped making progress.
- **Cause:** a slow internet connection.
- **Diagnosis:** I ran `top` and `ps aux` to check whether the installation was still running, then ran a connection speed test and saw downloads at only 11 KB/s.
- **Solution:** I changed the VM network from Bridged Adapter (Wi-Fi) to NAT Network + Port Forwarding.

## 3. SSH brute-force simulation (Kali → agent)

I connected to the agent over SSH from Kali. SSH limited me to 3 attempts per connection, so I ran the command repeatedly, entering a wrong password each time.

I then checked the Wazuh dashboard: it showed 32 authentication failures. At first I thought there were no high-level alerts, but there was a level 10 alert with the description "syslog: User missed the password more than one time" and rule ID 2502.

## 4. Nmap scan detection test

I tested whether an Nmap scan would be detected, and found that Wazuh, as a HIDS (Host-based Intrusion Detection System), does not detect network scanning activity coming from outside the host. For that you need a NIDS (Network Intrusion Detection System) component, such as Suricata.

## 5. File Integrity Monitoring test

I tested whether Wazuh detects modifications to `/etc/passwd`: I modified the file, restarted the agent and checked the dashboard, but nothing appeared.

Commands used:

```bash
echo "# test FIM" | sudo tee -a /etc/passwd
sudo systemctl restart wazuh-agent
sudo sed -n '114p' /var/ossec/etc/ossec.conf   # check the config
sudo nano /var/ossec/etc/ossec.conf            # added realtime="yes" on the <directories> line
```

- **Problem:** the FIM alert was missing entirely.
- **Cause:** the service stopped exactly while the event was being sent to the server. The local scan succeeded, but synchronization was interrupted.
- **Solution:** I dropped scan-on-restart and enabled real-time monitoring, which detects changes instantly, without a restart.

## What I learned

- FIM works on checksums: Wazuh computes a fingerprint of every monitored file and compares it with one stored locally. If they differ, the file has changed.
- There are 2 verification modes: periodic and real-time.

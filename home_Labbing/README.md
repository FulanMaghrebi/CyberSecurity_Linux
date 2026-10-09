# Homelab – Important Commands

This repository contains important Linux commands, configurations, and installation instructions for managing a home server.

---

## 1. Power Management

### 1.1 Disable Suspend on Lid Close

By default, Linux may suspend a laptop when its lid is closed.

To keep the home server running even when the laptop lid is closed, modify the systemd login manager configuration.

**Step 1: Open the configuration file**

```bash
sudo nano /etc/systemd/logind.conf
```

**Step 2: Modify the configuration**

Find the following settings under `[Login]`, remove the leading `#` if present, and set their values to `ignore`:

```ini
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

**Configuration explanation:**

| Setting                               | Description                                                         |
| ------------------------------------- | ------------------------------------------------------------------- |
| `HandleLidSwitch=ignore`              | Prevents suspension when closing the lid.                           |
| `HandleLidSwitchExternalPower=ignore` | Prevents suspension when the laptop is connected to external power. |
| `HandleLidSwitchDocked=ignore`        | Prevents suspension when the laptop is docked.                      |

**Step 3: Save the configuration**

In Nano:

1. Press `Ctrl + O` to save.
2. Press `Enter` to confirm.
3. Press `Ctrl + X` to exit.

**Step 4: Apply changes**

Restart the login manager:

```bash
sudo systemctl restart systemd-logind
```

Alternatively, reboot the server:

```bash
sudo reboot
```

> **Note:** Restarting systemd-logind may affect active sessions. A reboot will interrupt your SSH connection until the server starts again.

---

## 2. CasaOS Installation

### 2.1 Overview

CasaOS is a web-based home server management platform that provides an intuitive interface for managing Docker containers, applications, and files.

### 2.2 Installation

Install CasaOS using the official installation script:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

**Command explanation:**

| Command     | Description                               |
| ----------- | ----------------------------------------- |
| `curl`      | Downloads data from a URL.                |
| `-f`        | Fails on HTTP errors.                     |
| `-s`        | Enables silent mode.                      |
| `-S`        | Displays errors even in silent mode.      |
| `-L`        | Follows HTTP redirects.                   |
| `\|`        | Pipes the downloaded script to Bash.      |
| `sudo bash` | Executes the script with root privileges. |

> **Security Note:** The installation script is executed with root privileges. Review the script before execution.

### 2.3 Access

After installation, open the CasaOS web interface in your browser:

```text
http://<SERVER-IP>
```

Replace `<SERVER-IP>` with the IP address of your server.

### 2.4 Verification

Check the installed CasaOS version:

```bash
casaos -v
```

### 2.5 Update CasaOS

CasaOS can be updated through its web interface or via the command line:

```bash
curl -fsSL https://get.casaos.io/update | sudo bash
```

### 2.6 Uninstall CasaOS

To uninstall CasaOS:

```bash
sudo casaos-uninstall
```

> **Network Note:** CasaOS requires access to external download servers during installation. Ensure that your firewall allows the necessary outgoing HTTPS connections.

---

## 3. Firewall Configuration (UFW)

### 3.1 Overview

UFW (Uncomplicated Firewall) is a firewall management tool for Linux. It allows administrators to control incoming and outgoing network traffic by configuring firewall rules.

### 3.2 Installation

Install UFW using APT:

```bash
sudo apt update
sudo apt install ufw
```

### 3.3 Basic Configuration

Set the default firewall policies:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**Explanation:**

| Command                  | Description                                            |
| ------------------------ | ------------------------------------------------------ |
| `default deny incoming`  | Blocks incoming connections unless explicitly allowed. |
| `default allow outgoing` | Allows outgoing connections by default.                |

**Important: Allow SSH before enabling the firewall!**

```bash
sudo ufw allow 22/tcp
```

This allows incoming SSH connections on TCP port 22.

Enable UFW:

```bash
sudo ufw enable
```

Check firewall status:

```bash
sudo ufw status verbose
```

> **Warning:** Incorrect firewall configurations can interrupt SSH access. Always ensure the SSH port is allowed before enabling UFW.

### 3.4 Managing Firewall Rules

**Allow incoming connections:**

```bash
sudo ufw allow 80/tcp
```

**Deny incoming connections:**

```bash
sudo ufw deny 80/tcp
```

**Allow a specific IP address:**

```bash
sudo ufw allow from 192.168.1.100
```

**Allow SSH from a specific IP address:**

```bash
sudo ufw allow from 192.168.1.100 to any port 22 proto tcp
```

**Allow SSH from an entire subnet:**

```bash
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
```

Replace the example IP addresses and subnet with your actual network configuration.

### 3.5 Managing Outgoing Connections

By default, UFW allows outgoing connections.

**Block outgoing connections by default:**

```bash
sudo ufw default deny outgoing
```

**Allow outgoing HTTP connections:**

```bash
sudo ufw allow out 80/tcp
```

**Allow outgoing HTTPS connections:**

```bash
sudo ufw allow out 443/tcp
```

**Allow outgoing DNS requests over UDP:**

```bash
sudo ufw allow out 53/udp
```

**Restore outgoing connections:**

```bash
sudo ufw default allow outgoing
```

> **Warning:** Blocking outgoing connections can prevent software updates, DNS resolution, downloads and other network services from functioning. Configure the required exceptions before applying this policy. Changes to firewall defaults can also interrupt remote sessions.

### 3.6 Managing UFW

**Check firewall status:**

```bash
sudo ufw status
```

**Display numbered firewall rules:**

```bash
sudo ufw status numbered
```

**Delete a firewall rule by its number:**

```bash
sudo ufw delete 2
```

**Reload firewall rules:**

```bash
sudo ufw reload
```

**Disable the firewall:**

```bash
sudo ufw disable
```

**Enable the firewall:**

```bash
sudo ufw enable
```

**Enable logging:**

```bash
sudo ufw logging on
```

### 3.7 CasaOS Firewall Configuration

To allow access to the CasaOS web interface from the local network, assuming CasaOS uses port 80:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 80 proto tcp
```

This allows devices within the specified subnet to access CasaOS.

> **Security Note:** CasaOS uses Docker to manage applications. Docker-published container ports may bypass UFW rules. Additional Docker firewall configuration may be necessary to enforce network isolation.

---

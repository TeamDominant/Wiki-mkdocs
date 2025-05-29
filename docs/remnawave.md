# Remnawave

## Setup used

The setup, which was used in our case:

- OS: Debian 12
- Nodes: 1

## Installation

As soon as we log into the system after purchasing a VPS, we execute the following commands:

### Updating OS
```bash
apt update && apt upgrade -y && apt autoclean -y && apt clean -y && apt autoremove -y && reboot
```
to install all updates  
if your system is minimized by default, unminimize it by adding `unminimize`:
```bash
unminimize && apt update && apt upgrade -y && apt autoclean -y && apt clean -y && apt autoremove -y && reboot
```

### Domain Configuration
The script supports two methods of domain configuration: via Cloudflare or using ACME with your hosting provider.

### DNS Records Setup for Panel + Node on the Same Server

| Type  | Name              | Content          | Proxy status  |
| ----- | ----------------- | ---------------- | ------------- |
| A     | example.com       | your_server_ip   | DNS only      |
| CNAME | panel.example.com | example.com      | DNS only      |
| CNAME | sub.example.com   | example.com      | DNS only      |
| CNAME | node.example.com  | example.com      | DNS only      |

!!! note
    The node.example.com record is not mandatory for a selfsteal node — you can also use example.com for selfsteal.

### DNS Records Setup for Panel and Node on Separate Servers

| Type  | Name              | Content                 | Proxy status  |
| ----- | ----------------- | ----------------------- | ------------- |
| A     | example.com       | your_server_ip          | DNS only      |
| CNAME | panel.example.com | example.com             | DNS only      |
| CNAME | sub.example.com   | example.com             | DNS only      |
| A     | node.example.com  | your_server_ip_vps_node | DNS only      |

### Installation Guidelines
#### 1. Single Server Setup:
   - Select the option "Install Remnawave components", then choose "Install panel and node on one server". Once the process is complete, the script will automatically restart the panel and provide all the necessary login details.
#### 2. Dual Server Setup
   - Start with the first server and select the option "Install Remnawave components," then choose "Install panel only." Wait for the script to complete the setup and provide the login details for the panel.
   - Log into the control panel, navigate to Nodes → Management, select the desired node, and click the "Important Information". In the pop-up window, you’ll see an icon to copy the certificate — click on it.
   - Proceed to the second server and initiate the node installation. When prompted, paste the certificate you copied earlier.
   - Upon completion, you’ll see a message confirming that the node has been successfully connected.

### Server Setup

To set up the server, run this command on it:

```
bash <(curl -Ls https://raw.githubusercontent.com/eGamesAPI/remnawave-reverse-proxy/refs/heads/main/install_remnawave.sh)
```

Follow the instructions shown during the installation method you've selected.

### SSH

```bash
nano /etc/ssh/sshd_config
```

All you need to is uncomment line 14 and change your port. You can change the port to the one you like the most, just make sure it's not used by other services. Final result looks like:

```bash title="sshd_config" linenums="14" hl_lines="1"
Port 222
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

Then, you need to reload services:

!!! warning
    Make sure your ufw is **NOT** running, so you can connect after you decide to reconnect. If you forgot to do add new port to ufw, then access your VPS by using VNC and make changes.

```bash
systemctl daemon-reload
```

```bash
systemctl restart ssh
```

## Optional configuration

### Installing Xanmod kernel

1. Check the version you need to install
```bash
wget https://dl.xanmod.org/check_x86-64_psabi.sh
chmod +x check_x86-64_psabi.sh
./check_x86-64_psabi.sh
```

2. Register the PGP key
```bash
wget -qO - https://dl.xanmod.org/archive.key | sudo gpg --dearmor -vo /etc/apt/keyrings/xanmod-archive-keyring.gpg
```

3. Add the repo
```bash
echo 'deb [signed-by=/etc/apt/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-release.list
```

4. Update and install the version checked suggested you to install
```bash
sudo apt update && sudo apt install linux-xanmod-x64v3
```

5. Reboot

### Ufw

```bash
nano /etc/ufw/sysctl.conf
```

Change the value in the following string from `0` to `1`, so services can't two way ping your server

!!! warning
    This step is optional and should be only done if you really need to do so. In 99% cases this affects nothing.

```bash title="sysctl.conf" hl_lines="4"
# Ignore bogus ICMP errors
net/ipv4/icmp_echo_ignore_broadcasts=1
net/ipv4/icmp_ignore_bogus_error_responses=1
net/ipv4/icmp_echo_ignore_all=1
```

Press ++ctrl+x++, then ++y++, and hit ++enter++ to save and exit.

Restart the ufw:

```bash
ufw disable && ufw enable
```
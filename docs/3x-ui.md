# 3x-ui

!!! danger
    This page is no longer supported. Please, migrate to Remnawave.

## Setup used

The setup was used in my case:

- OS: Ubuntu 24.04.1
- 3x-ui Version: 2.5.6
- VPS: [Aéza](https://aeza.net/?ref=572663)

## Installation

As soon as we log into the system after purchasing a VPS, we execute the following commands:

1. 
```bash
apt update && apt upgrade -y
```
to install all updates  
if your system is minimized by default, unminimize it
```bash
unminimize && apt update && apt upgrade -y && apt autoclean -y && apt clean -y && apt autoremove -y && reboot
```

2. 
```bash 
openssl req -x509 -keyout /etc/ssl/certs/3x-ui.key -out /etc/ssl/certs/3x-ui.pem -newkey rsa:4096 -sha256 -days 3650 -nodes -new
```
where `.pem` is the public key, and `.key` is the private key.
Just keep pressing Enter, no need to fill anything in.

3. 
```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```
to install the panel itself

4. `Would you like to customize the Panel Port settings? (If not, a random port will be applied)`
we answer with ++n++.

5. Click on the link in the "Access URL" line to access our panel. Enter the credentials generated for us after the installation.

6. Go to Panel Settings, then to the Authentication section, and change the login details. In the General section, specify the path to the already created Public & Private keys: 
```bash title="public key"
/etc/ssl/certs/3x-ui.pem
```
```bash title="private key"
/etc/ssl/certs/3x-ui.key
```
Scroll up, click Save, and restart the panel.

## Securing and little tweaks

### Xanmod Kernel

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

5. If you didn't, apply the bbr config via `x-ui` or something. No need to do it, if you've done previously 

6. Reboot

### Ssh

Changing ssh port is required since ISPs scan for open ports and trying to detect such VPS machine, which you use as proxy.

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

```bash
systemctl daemon-reload
```

```bash
systemctl restart ssh
```

!!! warning
    Make sure your ufw is **NOT** running, so you can connect after you decide to reconnect. If you forgot to do add new port to ufw, then access your VPS by using VNC and make changes.

### Fail2ban

```bash
apt install fail2ban -y && apt install ufw -y
```

Right after the packages got installed, create and edit fail2ban jail config
```bash
touch /etc/fail2ban/jail.local && nano /etc/fail2ban/jail.local
```

Copy and paste the following ++ctrl+shift+v++

```bash title="jail.local"
[sshd]
enabled = true
chain   = INPUT
action  = iptables-allports
bantime = 1209600
maxretry = 1
logpath = /var/log/auth.log
```

Press ++ctrl+x++, then ++y++, and hit ++enter++ to save and exit.

In the terminal, type:

```bash
x-ui
```

Type 20 (IP Limit Management) and press ++enter++. Then press 1 to install and type ++y++.

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

```bash
apt install net-tools
```

```bash
netstat -ntlp | grep LISTEN
```
Open the ports you need (SSH, 3x-ui, 443, and any others if you have additional services).
```bash
ufw disable && ufw allow 222/tcp && ufw allow 443/tcp && ufw allow {panel_port} && ufw enable
```

### BBR

1. In the terminal, type
```bash
x-ui
```

2. Type 23 (Enable BBR) and select 1 (Enable BBR).

## SSL Management

!!! warning
    This step is only required if you don't want to use the least secure connection. If you don't really care, move to the next step: [Reality](https://wiki.amdcloud.kz/3x-ui/#reality).

First, let's open open `80/tcp` for some time:

```bash
ufw allow 80/tcp
```

Next, move the **x-ui** panel:

```bash
x-ui
```

and then select **SSL Management (18)**. Press **1** to **Get SSL**.

!!! warning
    On this step you may face the 404 error because speedtest-cli is not available for **Ubuntu 24.04**. Remove the repo from apt repos by using [this guide](https://askubuntu.com/questions/43345/how-to-remove-a-repository).

Enter your domain, default port - 80, wait until the certificate is created and then agree to set certificate for the panel.

Leave port `80/tcp` is not secure, so let's make a crontab for it:

```bash
crontab -e
```

```bash title="crontab"
0 23 * * * ufw allow 80/tcp && "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null && ufw deny 80/tcp
```

## Setting up
### Reality

!!! note 
    Easy, but less secure. Doesn't require SSL Management to be set up.  

- Protocol: `vless`
- Port: `443`
- Client: `Enabled`
- Email = `email` or `username`
- Security: `Reality`
- uTLS: `chrome/firefox`
- Dest & SNI: 
   * Sweden / Finland: `teamdocs.su:443`
   * Germany: `wikiportal.su:443`
   * Moscow: `docscenter.su:443`

If port `443` is used, then:

- Port: custom or default
- uTLS: `chrome/firefox/random`
You are supposed to test it yourself because it works differently on each VPS, ISP, and even OS.  

Click on the QR code icon next to the created client, then click the QR code to copy it to the clipboard, and paste it into Nekoray/Streisand.

### TLS

Install nginx first:

```bash 
apt install nginx
```

and then change its default config:

```bash
nano /etc/nginx/sites-enabled/default
```

Make the changes to file:

```bash
# Default server configuration
#
server {
        listen 127.0.0.1:8080 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
```

Test the nginx by using:
```bash
nginx -t
```

```bash
systemctl reload nginx
```

Settings inside the 3x-ui panel for inbound:  

- Port: `443`
- Add fallback. Dest: `8080`  
- uTLS: `None` (?)
- Security: `TLS`  
- ALPN: `http/1.1`  
- Set cert from panel  
- Sniffing: `Enabled` (?)  

Create a webpage for correct work. Let's ask an AI to create a webpage for us.

```bash
nano /var/www/html/index.html
```

Simply paste any html code and save it.  

After its all done we need to close port `80/tcp`:

```bash
ufw deny 80/tcp
```

### cortez24rus's script

Follow the instructions on the repo [page](https://github.com/cortez24rus/xui-reverse-proxy).

and run the script itself after you've done the requirements.

```bash
bash <(curl -Ls https://raw.githubusercontent.com/cortez24rus/xui-reverse-proxy/refs/heads/main/reverse_proxy.sh)
```

## Clean-up

Finally, clean up, reboot, and you're ready to use it.
```bash
apt update && apt upgrade -y && apt autoclean -y && apt clean -y && apt autoremove -y && reboot
```
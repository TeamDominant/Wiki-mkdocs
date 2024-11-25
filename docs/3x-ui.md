# 3x-ui

## Setup used

The setup was used in my case:

- OS: Ubuntu 22.04.5
- 3x-ui Version: 2.4.6
- VPS: [Aeza](https://aeza.net/?ref=572663)

## Installation

As soon as we log into the system after purchasing a VPS, we execute the following commands:

1. 
```bash
apt update && apt upgrade -y
```
to install all updates

2. 
```bash 
openssl req -x509 -keyout /etc/ssl/certs/3x-ui.key -out /etc/ssl/certs/3x-ui.pem -newkey rsa:4096 -sha256 -days 3650 -nodes -new
```
where .pem is the public key, and .key is the private key.
Just keep pressing Enter, no need to fill anything in.

3. 
```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```
to install the panel itself

## Setting up

1. `Would you like to customize the Panel Port settings? (If not, a random port will be applied)`
we answer "n".

2. Click on the link in the "Access URL" line to access our panel. Enter the credentials generated for us after the installation.

3.Go to Panel Settings, then to the Authentication section, and change the login details. In the General section, specify the path to the already created Public & Private keys: /etc/ssl/certs/3x-ui.pem and /etc/ssl/certs/3x-ui.key. Scroll up, click Save, and restart the panel.

## Inbound creation

- Protocol: `vless`
- Port: `443`
- Client: `Enabled`
- Email = `email` or `username`
- Security: `Reality`
- uTLS: `chrome/firefox`
- Dest и SNI: choose a website with the lowest ping in the country where your VPS is located. You can check it by running the following command in your VPS terminal: ping domain.com. You can ask your VPS support for assistance or simply ask ChatGPT.

If port 443 is occupied, then:

- Port: custom or default
- uTLS: chrome/firefox/random
You are supposed to test it yourself because it works differently on each VPS, ISP, and even OS.
- Dest and SNI: you also need to test these yourself.

Click on the QR code icon next to the created client, then click the QR code to copy it to the clipboard, and paste it into Nekoray/Streisand.

Click on the QR code icon next to the created client, then click on the QR code to copy it to the clipboard and paste it into Nekoray/Streisand.

## Securing and little tweaks
### Fail2ban

```bash
apt install fail2ban -y && apt install ufw -y
```

After runnig `touch /etc/fail2ban/jail.local && nano /etc/fail2ban/jail.local`
Copy and paste (ctrl + shift + v):

```nginx
[sshd]
enabled = true
filter = sshd
action = iptables[name=SSH, port=ssh, protocol=tcp]
logpath = /var/log/auth.log
findtime = 600
maxretry = 3
bantime = 43200
```

Press ctrl + x, then y, and hit enter to save and exit.

### Ufw

1. 
```bash
nano /etc/ufw/before.rules
```
Look `-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT` and change `ACCEPT` to `DROP`

2. 
```bash
apt install net-tools
```
```bash
netstat -ntlp | grep LISTEN
```
Open the ports you need (SSH, 3x-ui, 443, and any others if you have additional services).
```bash
ufw allow 22/tcp && ufw allow 443 && ufw allow {panel_port} && ufw enable
```
ctrl + x, y and enter.

3. In the terminal, type 
```bash
x-ui
```

4. Type 20 (IP Limit Management) and press Enter. Then press 1 to install and type y.

### BBR

1. In the terminal, type
```bash
x-ui
```

2. Type 23 (Enable BBR) and select 1 (Enable BBR).

## Final

Finally, clean up, reboot, and you're ready to use it.
```bash
apt update && apt upgrade -y && apt autoclean -y && apt clean -y && apt autoremove -y && reboot
```
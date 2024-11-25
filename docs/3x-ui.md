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
где .pem - публичный ключ, а .key - приватный.  
Просто спамим enter, ничего заполнять не нужно.

3. 
```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```
to install the panel itself

## Setting up

1. `Would you like to customize the Panel Port settings? (If not, a random port will be applied)`
отвечаем "n".

2. Нажимаем на ссылку в строке Access URL, чтобы попасть в нашу панель. Вводим данные, которые нам были сгенерированы после установки.

3. Заходим в Panel Settings и после в раздел Authentication, меняем данные для входа. В разделе General указываем путь к уже созданным Public & Private ключам: /etc/ssl/certs/3x-ui.pem и /etc/ssl/certs/3x-ui.key, листаем вверх и жмём Save и Restart panel.

## Inbound creation

- Protocol: `vless`
- Port: `443`
- Client: `Enabled`
- Email = `email` or `username`
- Security: `Reality`
- uTLS: `chrome/firefox`
- Dest и SNI: подбираем сайт, с минимальным пингом, в стране, где Ваш VPS. Проверить можно вводом в терминал VPS: `ping domain.com`. Можете уточнить в поддержке VPS или просто спросить ChatGPT.

Если у вас занят 443 порт, то:

- Port: пользовательский или по умолчанию
- uTLS: chrome/firefox/random  
  You are supposed to test it youself, because it works differently on each VPS, IPS and even OS. 
- Dest and SNI: you also need to also check it yourself

Нажимаем на иконку QR-code возле создавшегося клиента, кликаем по QR-code, чтобы скопировать в буфер обмена и вставляем в Nekoray/Streisand.

## Securing and little tweaks
### Fail2ban

```bash
apt install fail2ban -y && apt install ufw -y
```

После `touch /etc/fail2ban/jail.local && nano /etc/fail2ban/jail.local`
Копируем и вставляем (ctrl + shift + v):

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

ctrl + x, y и enter, чтобы сохранить и выйти.

### Ufw

1. 
```bash
nano /etc/ufw/before.rules
```
ищем `-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT` и меняем `ACCEPT` на `DROP`

2. 
```bash
apt install net-tools
```
```bash
netstat -ntlp | grep LISTEN
```
Открываем порты, которые нам нужны (ssh, 3x-ui, 443 и остальные, если у Вас что-то еще помимо этого)
```bash
ufw allow 22/tcp && ufw allow 443 && ufw allow {panel_port} && ufw enable
```
ctrl + x, y и enter.

3. В терминале вводим 
```bash
x-ui
```

4. вводим 20 (IP Limit Management) и enter. После нажимаем 1 для установки и вводим y. 

### BBR

1. В терминале вводим 
```bash
x-ui
```

2. вводим 23 (Enable BBR) и выбираем 1 (Enable BBR).

## Final

Напоследок подчищаем, ребут и можем пользоваться.
```bash
apt update && apt upgrade -y && apt autoclean -y && apt clean -y && apt autoremove -y && reboot
```
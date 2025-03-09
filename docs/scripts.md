# Scripts

## Geo-check

```bash
wget -qO - "https://raw.githubusercontent.com/vernette/ipregion/refs/heads/master/ipregion.sh" | bash
```


## Speedtest:

```bash
sudo apt update && sudo apt install -y curl && curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash && sudo apt install -y speedtest && speedtest
```

## IP check for service blocks

```bash
bash <(curl -Ls IP.Check.Place) -l en
```

## :fire: Env info and tests with Russian ISPs

```bash
wget -qO- speedtest.artydev.ru | bash 
```

## :fire: Env info and tests with ISPs

```bash
wget -qO- bench.sh | bash
```

## Xanmod Kernel

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


## ICMP Limit reqs

Продолжая тему облегчения всем ддосов да и в принципе облегчения жизни серверу
Есть параметры ядра которые ограничивают кол-во ответов ICMP когда на сервер приходит любой запрос на не открытый порт(т.е. порт не закрыт фаерволом и его ни одно приложение не слушает), так вот, по дефолту рейтлимит 1000 пакетов в секунду, что очень много, что бы ограничить это нужно сделать следующее:
подключиться к серверу по ssh и дать блоком следующие команды, т.е. полностью их скопировать и полностью вставить

```bash
cat <<EOT >> /etc/sysctl.conf 
net.ipv4.icmp_msgs_burst=1
net.ipv4.icmp_msgs_per_sec=1
net.ipv4.icmp_ratelimit=1
net.ipv6.icmp.ratelimit=1
EOT
sysctl -p
```
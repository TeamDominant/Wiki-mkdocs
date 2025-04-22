# OpenWrt
## Installation
### [Installting OpenWrt](https://openwrt.org/docs/guide-user/installation/openwrt_x86#linux)
1. Boot into Archlinux with your bootable USB Drive.

2. As soon as you are in, download wget package with
```bash
pacman -Sy wget
```

3. Download the proper image you need. In our case the command looks like this
```bash
wget https://downloads.openwrt.org/releases/24.10.1/targets/x86/64/openwrt-24.10.1-x86-64-generic-ext4-combined-efi.img.gz
```

4. Unpack image (simply press tab right after writing gunzip)
```bash
gunzip openwrt-*.img.gz
```

5. Identify disk (to replace sdX in the following command below)
```bash
lsblk
```

6. Write image
```bash
dd if=openwrt-24.10.1-x86-64-generic-ext4-combined-efi.img bs=1M of=/dev/sdX
```

You are ready to boot into an OpenWrt now. You may have to hit enter a couple of times if the boot seems to hang, it'll drop you to the command prompt and complain that there's no password.

### Ethernet ports configuration

You'll want to modify your interfaces, to determine WAN port and others as LAN

```bash
vi /etc/config/network
```

Use ++S++ to enter INSERT mode.

```lan hl_lines="4"
config device
        option name 'br-lan'
        option type 'bridge'
        list ports 'eth0'
```
```wan  hl_lines="2"
config interface 'wan'
        option device 'eth1'
        option proto 'dhcp'
```
```wan6 hl_lines="2"
config interface 'wan6'
        option device 'eth1'
        option proto 'dhcpv6'
```

Press ++Esc++ to exit INSERT mode and save-exit with `:wq`

### [Partition](https://openwrt.org/docs/guide-user/installation/openwrt_x86#partition_layout) 

The x86 image is using the following partition layout (as seen from inside of the device):

- /dev/sda1 is a 16MB ext4 /boot partition where GRUB and the kernel are stored.
- /dev/sda2 is a 104MB partition containing the squashfs root filesystem and a read-write f2fs filesystem OR the ext4 root filesystem (depending on what image you have chosen).
- And also nowadays there is /dev/sda3, which is X MB partition.

### [Expanding root partition and filesystem](https://openwrt.org/docs/guide-user/advanced/expand_root)

OpenWrt doesn't use the whole partition space, so you need to fix it for future configurations.

1. Install packages
```bash
opkg update
opkg install parted losetup resize2fs
```

2. Download and run `expand-root.sh`
```bash
wget -U "" -O expand-root.sh "https://openwrt.org/_export/code/docs/guide-user/advanced/expand_root?codeblock=0"
. ./expand-root.sh
```

3. Reboot and check 
```bash
reboot
```

## AdGuard Home
### [Installation](https://openwrt.org/docs/guide-user/services/dns/adguard-home#installation)

Since 21.02, there is a official AdGuard Home package which can be installed through opkg.

The opkg package for 21.02 has also been confirmed to work on 19.07, but will require transferring the correct ipk through SSH or SCP and installing with opkg manually due to not being present in the 19.07 packages repository.

Required dependencies (ca-bundle) are automatically resolved and installed when using the official package. 

```bash
opkg update
opkg install adguardhome
```

To have AdGuard Home automatically start on boot and to start the service: 
```bash
service adguardhome enable
service adguardhome start
```

### [Setup](https://openwrt.org/docs/guide-user/services/dns/adguard-home#setup)

Create `script.sh` file and paste this code from OpenWrt wiki

```sh
# Get the first IPv4 and IPv6 Address of router and store them in following variables for use during the script.
NET_ADDR=$(/sbin/ip -o -4 addr list br-lan | awk 'NR==1{ split($4, ip_addr, "/"); print ip_addr[1]; exit }')
NET_ADDR6=$(/sbin/ip -o -6 addr list br-lan scope global | awk '$4 ~ /^fd|^fc/ { split($4, ip_addr, "/"); print ip_addr[1]; exit }')
echo "Router IPv4 : ""${NET_ADDR}"
echo "Router IPv6 : ""${NET_ADDR6}"
 
# 1. Move dnsmasq to port 54.
# 2. Set local domain to "lan".
# 3. Add local '/lan/' to make sure all queries *.lan are resolved in dnsmasq;
# 4. Add expandhosts '1' to make sure non-expanded hosts are expanded to ".lan";
# 5. Disable dnsmasq cache size as it will only provide PTR/rDNS info, making sure queries are always up to date (even if a device internal IP change after a DHCP lease renew).
# 6. Disable reading /tmp/resolv.conf.d/resolv.conf.auto file (which are your ISP nameservers by default), you don't want to leak any queries to your ISP.
# 7. Delete all forwarding servers from dnsmasq config.
uci set dhcp.@dnsmasq[0].port="54"
uci set dhcp.@dnsmasq[0].domain="lan"
uci set dhcp.@dnsmasq[0].local="/lan/"
uci set dhcp.@dnsmasq[0].expandhosts="1"
uci set dhcp.@dnsmasq[0].cachesize="0"
uci set dhcp.@dnsmasq[0].noresolv="1"
uci -q del dhcp.@dnsmasq[0].server
 
# Delete existing config ready to install new options.
uci -q del dhcp.lan.dhcp_option
uci -q del dhcp.lan.dns
 
# DHCP option 3: Specifies the gateway the DHCP server should send to DHCP clients.
uci add_list dhcp.lan.dhcp_option='3,'"${NET_ADDR}"
 
# DHCP option 6: Specifies the DNS server the DHCP server should send to DHCP clients.
uci add_list dhcp.lan.dhcp_option='6,'"${NET_ADDR}" 
 
# DHCP option 15: Specifies the domain suffix the DHCP server should send to DHCP clients.
uci add_list dhcp.lan.dhcp_option='15,'"lan"
 
# Set IPv6 Announced DNS
uci add_list dhcp.lan.dns="$NET_ADDR6"
 
uci commit dhcp
service dnsmasq restart
service odhcpd restart
exit 0
```

Execute the script
```bash
chmod +x script.sh
./script.sh
```

### [Setup AGH through the web interface](https://openwrt.org/docs/guide-user/services/dns/adguard-home#setup_agh_through_the_web_interface)

On first time setup the default web interface port is TCP 3000.

- Go to http://192.168.1.1:3000/ (If your router IP is not 192.168.1.1, change this accordingly)
- Setup the Admin Web Interface to listen on 192.168.1.1 at port 8080. (Changing the web interface port is optional)
- Set DNS server to listen on 192.168.1.1 at port 53.
- Create an user and choose a strong password.

### [Login AGH](https://openwrt.org/docs/guide-user/services/dns/adguard-home#login_agh)

- http://192.168.1.1:8080/ (or whatever listening port you set)

1. Go to DNS Settings and add Upstream DNS servers. Enter one server address per line. [Learn more](https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration#upstreams) about configuring upstream DNS servers. Here is a [list of known DNS providers](https://link.adtidy.org/forward.html?action=dns_kb_providers&from=ui&app=home) to choose from.
2. Add your Private reverse DNS servers such as `127.0.0.1:54` and `[::1]:54`

Feel free to change upstream DNS servers to whatever you like (Adguard Home supports DoH, DoT and DoQ out of the box), add the blacklists of your preference and enjoy ad-free browsing on all of your devices. 

## SQM
### [Preparation: Measure Your Current Speed and Latency](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm#preparationmeasure_your_current_speed_and_latency)

Before you can optimize your network, you need to know its current state.
- When your internet is quiet run a speed test from [Waveform](https://www.waveform.com/tools/bufferbloat) or [Speedtest](https://speedtest.net/). This is to determine your peak download/upload speeds, latency, and grade your bufferbloat.
- To maximize performance most devices will benefit from enabling Packet Steering under LuCI → Network → Interfaces → Global network options.
- If you are using this OpenWrt device as an [Extender, Repeater, or Bridge](https://openwrt.org/docs/guide-user/network/wifi/relay_configuration), test your upstream router (OpenWrt or otherwise) and determine if an issue is present there first.
- If you are on a [wireless AP](https://openwrt.org/docs/guide-user/network/wifi/wifiextenders/bridgedap), test your upstream router separately. If your AP wifi driver supports AQL limits (e.g. [mt76](https://openwrt.org/docs/techref/driver.wlan/mt76) does) adjust/reduce those seperately to improve wifi latency.

### Installation
Install `luci-app-sqm` (or `sqm-scripts` if you don't use LuCI) and read below.

In LuCI go to Network → SQM QoS:

1. In the **Basic Settings** tab:
    - Check the **Enable** box
    - Set the **Interface** to your internet (WAN) link in the dropdown. Check Network → Interfaces if you need to determine your WAN port.
    - Enter your **Download** and **Upload** speeds to 90% of the results you tested in Preparation
2. In the **Queue Discipline** tab:
    - Choose *cake* as the Queueing Discipline (or fq_codel, consider [note 3](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm#a_little_more_tuning))
    - Choose *piece_of_cake.qos* as the Queue Setup Script
    - Advanced Configuration may be left unchecked (see [note 4](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm#a_little_more_tuning) for advanced settings)
3. In the **Link Layer Adaptation** tab, select your link and overhead (setting mpu is optional see [note 2](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm#a_little_more_tuning)):
    - VDSL - choose **Ethernet**, and set overhead 34 (or 26 if you're not using PPPoE) (mpu 68). If the link is 100 Mbps Ethernet set overhead 42 (mpu 84).
    - DSL of any other type - choose **ATM**, and set overhead 44 (mpu 96).
    - DOCSIS Cable - choose **Ethernet**, and for rates < 760 Mbps set overhead 22 (mpu 64), for rates >= 760 Mbps set overhead 42 (mpu 84).
    - Fiber - choose **Ethernet**, and set overhead 44 (mpu 84).
    - Ethernet - choose **Ethernet**, and set overhead 44 (mpu 84).
    - If unsure - it's better to overestimate, choose **Ethernet**, and set overhead 44 (mpu 96).
4. Click **Save & Apply**.

# 靜宜系統GateWay建置

gw03 ubuntu 18.04
mas01 alpine 3.13.1
wka01 alpine 3.13.1
wka02 alpine 3.13.1
wka03 alpine 3.13.1
wka04 alpine 3.13.1
ds01 alpine 3.13.1


# GateWay事前準備 
- 取消 sudo 輸入密碼步驟
>$ sudo nano  /etc/sudoers
                   :::
%sudo   ALL=(ALL:ALL)  NOPASSWD:ALL
- 安裝wifi套件
>$ sudo apt update -y

>$ sudo apt install wpasupplicant

>$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev

update_config=1

ap_scan=1

fast_reauth=1

country=TW


network={
    ssid="輸入 Wifi 名稱" 
    psk="輸入 Wifi 密碼"
}
```
---



>$ sudo kill -9 $(ps -ef | grep wpa | awk '{print $2}')

>$ sudo wpa_supplicant -B -i XXXwifi網卡名稱 -D wext -c /etc/wpa_supplicant/wpa_supplicant.conf

>$ sudo dhclient

# 開機自動連線wpa_supplicant.service
$ sudo nano /etc/systemd/system/wpa_supplicant.service
```
[Unit]
Description=WPA supplicant
Before=network.target
After=dbus.service
Wants=network.target
IgnoreOnIsolate=true

[Service]
Type=dbus
BusName=fi.w1.wpa_supplicant1
ExecStart=/sbin/wpa_supplicant -u -s -D wext -c /etc/wpa_supplicant/wpa_supplicant.conf -i  XXXX網卡名稱

[Install]
WantedBy=multi-user.target  
Alias=dbus-fi.w1.wpa_supplicant1.service

```
# 開機自動連線dhclient.service
 
>$ sudo nano /etc/systemd/system/dhclient.service

!!重要
>$ sudo systemctl enable dhclient.service

>$ sudo reboot

```
[Unit]
Description= DHCP Client
Before=network.target
After=wpa_supplicant.service

[Service]
Type=simple
ExecStart=/sbin/dhclient XXXXXwifi網卡名稱

[Install]
WantedBy=multi-user.target

```

# 開機執行iptable 跟ip_forward

$ echo "sudo iptables -t nat -A POSTROUTING -s 192.168.30.0/24 -o 對外網卡名稱 -j MASQUERADE" >>.bashrc

$ echo 'echo 1 |sudo tee /proc/sys/net/ipv4/ip_forward'>>.bashrc

# 設定default gateway
>$ sudo nano /etc/netplan/XX-network-manager-all.yaml
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
      dhcp6: no
      addresses: [172.17.40.1/26]
      gateway4: 192.192.156.62
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]


```
# 設定 SSH 遠端管理系統
$ sudo nano /etc/ssh/ssh_config
                            :::
StrictHostKeyChecking no

[重要] StrictHostKeyChecking ask 修改為 no，並註解拿掉

$ sudo nano /etc/ssh/sshd_config
                           :::
PermitRootLogin no
# 是否允許 root 管理者以 SSH 登入
[重要] PermitRootLogin without-password 修改為 no

# 安裝Dnsmasq on Ubuntu 18.04
>$ sudo systemctl disable systemd-resolved

>$ sudo systemctl stop systemd-resolved

>$ sudo rm /etc/resolv.conf

>$ sudo nano /etc/resolv.conf
```
nameserver 127.0.0.1
nameserver 8.8.8.8
```
>$ sudo apt-get install dnsmasq

>$ sudo nano /etc/dnsmasq.conf
```
If you want to enable DNSSEC validation and caching, uncomment

#dnssec
Make any other changes you see relevant and restart dnsmasq when done:
```

# 設定Dnsmasq DHCP server(並將其設定為固定ip來使用k3s)
$ sudo nano /etc/dnsmasq.conf
```
dhcp-range=192.168.30.100,192.168.30.150,24h
dhcp-option=option:router,192.168.30.254
dhcp-option=option:dns-server,192.168.30.254
dhcp-option=option:netmask,255.255.255.0
dhcp-host=00:07:32:42:5e:3a,192.168.30.10
dhcp-host=00:07:32:4d:1e:46,192.168.30.20
dhcp-host=00:07:32:4d:1e:97,192.168.30.21
dhcp-host=00:07:32:42:70:c1,192.168.30.22
dhcp-host=00:07:32:4d:1e:17,192.168.30.23
dhcp-host=00:07:32:42:5e:c5,192.168.30.30

```
$ sudo systemctl restart dnsmasq
# 叢集教學環境系統環境設定流程
## 前製作業
- 在GateWay電腦加目錄解壓縮cnt.zip

>$ unzip cnt.zip
> $ tree cnt

![](https://i.imgur.com/AYAqtZ5.png =45%x)
- 進入cnt目錄
> $ cd cnt 
- 更改執行程式會使用到的變數
> $ nano config
```
#設定主機名稱
HN=gw01
#設定使用者名稱
UR=bigred
#設定執行程式時所在目錄
DR=~/cnt
#設定叢集電腦名稱
CLUSTER="mas01 wka01 wka02 wka03 wka04 ds01"
#抓取對外網卡名稱
GW=$(route -n | grep -e "^0.0.0.0 ")
GWIF=${GW##* }
#抓取對外IP
IPS=$(ifconfig $GWIF | grep 'inet ')
IP=$(echo $IPS | cut -d' ' -f2)
#設定程式裡面busybox httpd port號
HTTPORT=8888
```
## 執行startcnt程式
- startcnt 
- 1. GW安裝所需管理套件
- 2. 啟動 busybox httpd 
- 3. 以root身分登入到叢集中的主機，並從busybox網站執行prefly.sh程式
> $ nano startcnt

> $ ./startcnt
## 執行sysprep.sh程式
- sysprep.sh
- 1. 在GW產生公私鑰憑證
- 2. 發送公私鑰憑證給叢集電腦 (達成免密碼登入)
- 3. 增加 ssh server端(叢集電腦)環境變數(.ssh/environmenr)
> ./sysprep.sh
## 執行clusterinfo.sh
- clusterinfo.sh
- 1. 以第二管理者身分登入到叢集中的主機，並從busybox網站執行sysinfo.sh程式
- 2. 顯示叢及電腦的系統資訊
> $ ./clusterinfo
## alpine環境建置

- 安裝完後
$ vi /etc/ssh/sshd_config 
PermitRootLogin yes (予許使用ssh時root登入)

在GateWay裡面起busybox httpd ，讓alpine執行以下遠端程式(用curl curl GateWayIP:p

ort號/set/prefly.sh| sh)
````bash
#!/bin/bash
apk update &> /dev/null
apk upgrade &> /dev/null
[ $? = 0 ] && echo "system upgrade ok"
for ap in nano bash curl tree sudo grep procps
do
which $ap &>/dev/null
[ "$?" != "0" ] && apk add ${ap} &> /dev/null
a[ $? = 0 ] && echo "${ap} add ok"
done
#讓alpine可以sudo不用密碼
cat /etc/sudoers | grep -x "%wheel ALL=(ALL) NOPASSWD: ALL" &>/dev/null
[ $? != 0 ] && echo '%wheel ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers && echo "sudo Nopasswd OK"
#讓alpine ssh時不用確認是否收取server的公鑰
cat /etc/ssh/ssh_config |grep -x "StrictHostKeyChecking no" &>/dev/null
[ $? != 0 ] && echo 'StrictHostKeyChecking no' >> /etc/ssh/ssh_config
#予許client端可以使用本機的.ssh/environment環境變數
cat /etc/ssh/sshd_config | grep -x "PermitUserEnvironment yes" &>/dev/null
[ $? != 0 ] && echo 'PermitUserEnvironment yes' >> /etc/ssh/sshd_config
#停用ipv6功能(需要重啟)
ifconfig |grep inet6 &>/dev/null
[ $? == 0 ]  && echo "net.ipv6.conf.all.disable_ipv6 = 1" >>/etc/sysctl.conf && echo "ivp6 stop ok"
#重啟sshd&ssh
/etc/init.d/sshd restart
#重啟sysctl
/etc/init.d/sysctl restart
#更改alpin登入歡迎語
echo "Welcom TO Cloud Native Trainer" > /etc/motd
#建立第二管理者帳號bigred
if [ ! -d  /home/bigred ];then
  adduser -s /bin/bash -h /home/bigred -D bigred
  addgroup bigred wheel
  echo -e "bigred\nbigred\n" | passwd bigred &> /dev/null
  echo "bigred ready"
else
echo "bigred exist"
fi
```

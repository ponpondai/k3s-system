# 叢集教學環境系統環境設定流程
## 前製作業
- 在GateWay電腦加目錄解壓縮cnt.zip

>$ unzip cnt.zip
> $ tree cnt

![](https://i.imgur.com/AYAqtZ5.png )
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

在GateWay裡面起busybox httpd ，讓alpine執行以下遠端程式(用curl curl GateWayIP:port號/set/prefly.sh| sh)


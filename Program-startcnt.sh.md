# startcnt
```bash=
#!/bin/bash
#使用config設定
source config
# 檢視GW的linux作業系統、主機名稱、使用者、所在路徑是否正確
cat /etc/os-release | grep 'NAME="Ubuntu"' &>/dev/null
[ "$?" != "0" ] && echo "the system is not Ubuntu" && exit 1
[ `hostname` != "${HN}" ] && echo "wrong hostname" && exit 1
[ `whoami` != "${UR}" ] && echo "wrong user" && exit 1
[ `pwd` != "${DR}" ] && echo "pls move to cnt" && exit 1
# 更新軟體套件清單
echo "`hostname` is updating"
sudo apt update &>/dev/null
echo "${HN} update ok"
# 安裝所需管理套件
which sshpass &>/dev/null
[ "$?" != "0" ] && sudo apt-get install sshpass &>/dev/null && echo "sshpass install ok"
which ./busybox &>/dev/null
[ "$?" != "0" ] && wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 &>/dev/null && echo "busybox install ok"$
# 啟動 busybox httpd
ps aux | grep -v grep | grep "busybox httpd -p ${HTTPORT}" &>/dev/null
if [ "$?" = "0" ];then
  echo "busybox httpd started"
else
  ./busybox httpd -p ${HTTPORT} -h www
fi
# 以root身分登入到叢集中的主機，並從網站下載所需執行prefly
echo "prefly is beginning"
for n in $CLUSTER
do
  nc -w 1 -z $n 22 &>/dev/null
  if [ $? = 0 ] ; then
    sshpass -p "root" ssh root@${n} exit &>/dev/null
    if [ $? = 0 ] ; then
      sshpass -p "root" ssh root@${n} grep bigred /etc/passwd &>/dev/null
      if [ $? != 0 ] ; then
        sshpass -p "root" ssh -q root@${n} "apk add curl" &>/dev/null
        sshpass -p "root" ssh root@${n} "curl ${IP}:${HTTPORT}/set/prefly.sh| sh" &>/dev/null
        sshpass -p "root" ssh root@${n} '/etc/init.d/sshd restart'
        echo "${n} system prefly ok"
      else
        echo "$n bigred exist"
```
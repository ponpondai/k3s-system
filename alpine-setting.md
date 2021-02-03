# alpine環境建置

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
[ $? = 0 ] && echo "${ap} add ok"
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
````
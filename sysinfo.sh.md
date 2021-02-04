# sysinfo(觀察系統資訊)
```bash=
#!/bin/bash
#抓取對外網卡名稱
gw=$(route -n | grep -e "^0.0.0.0 ")
GWIF=${gw##* }
#抓取網卡IP
ips=$(ifconfig $GWIF | grep 'inet ')
IP=$(echo $ips | cut -d' ' -f2)
NETID=${IP%.*}
#抓GateWayIp
GW=$(route -n | grep -e '^0.0.0.0' | tr -s \ - | cut -d ' ' -f2)
#
echo "[`hostname`]"
echo "--------------------------------------------------------"
os=$(cat /etc/os-release | grep -E "^NAME" | cut -d'=' -f 2)
vs=$(cat /etc/os-release | grep VERSION_ID | cut -d'=' -f 2)
echo "OS : $os"
echo "VERSION : $vs" 
cn=$(cat /proc/cpuinfo | grep 'model name' | head -n 1 | cut -d ':' -f2 | tr -s ' ')
echo -n "CPU : $cn (core: "
cn=$(cat /proc/cpuinfo | grep 'model name' | wc -l)
echo "$cn)"

m=$(free -mh | grep Mem:)
echo -n "Memory : "
echo $m | cut -d' ' -f2 | sed 's/.$//'


echo "IP Address : $IP"
echo "Default Gateway : $GW"
echo "$NETID"


java -version &> /tmp/java
[ "$?" != "0" ] && echo 'JAVA NOT FOUND' || cat /tmp/java
echo ""
```
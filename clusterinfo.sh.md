# clusterinfo.sh(叢集系統資訊檢視)
```bash=
#!/bin/bash
source config 
[ `hostname` != "${HN}" ] && echo "wrong hostname" && exit 1
[ `whoami` != "${UR}" ] && echo "wrong user" && exit 1
[ `pwd` != ${DR} ] && echo "pls move to cnt2" && exit 1
for n in ${CLUSTER}
do
  nc -w 1 -z $n 22 &>/dev/null
  if [ "$?" == "0" ];then
    ssh $n "curl -s http://${IP}:${HTTPORT}/sysinfo.sh | bash"
  fi
done

```
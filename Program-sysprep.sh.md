# sysprep.sh
## GateWay產生公私鑰並達成免密碼登錄
```bash=
#!/bin/bash
#載入設定檔(config)
source ~/cnt/config
#判斷主機名稱、使用者、執行位置是否正確
[ `hostname` != "${HN}" ] && echo "wrong hostname" && exit 1
[ `whoami` != "${UR}" ] && echo "wrong user" && exit 1
[ `pwd` != "${DR}" ] && echo "pls move to ${DR}" && exit 1
#產生公私鑰以及公鑰憑證
echo '' | ssh-keygen -t rsa -P '' &>/dev/null
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys

#傳送公私鑰以及公鑰憑證至叢集電腦
for n in ${CLUSTER}
do
#判斷叢集電腦是否存在，以及是否為alpine，是否已有私鑰
  nc -w 1 -z $n 22 &>/dev/null
  if [ "$?" == "0" ];then
    sshpass -p 'bigred' ssh -q $n cat /etc/os-release|grep 'NAME="Alpine Linux"' &>/dev/null
    if [ "$?" = "0" ];then  
      sshpass -p 'bigred' ssh $n ls ~/.ssh/id_rsa &>/dev/null
      if [ $? != 0 ];then
	 #刪除.ssh資料夾並再次建立，傳送公私鑰以及公鑰憑證並更改公私鑰權限為700才可使用，並再次刪除knownhost檔案，增加ssh環境變數
        sshpass -p 'bigred' ssh $n 'sudo rm -r ~/.ssh/ &>/dev/null'
        sshpass -p 'bigred' ssh $n 'mkdir -p ~/.ssh'
        sshpass -p 'bigred' scp -r ~/.ssh/ $n:~
        sshpass -p 'bigred' ssh $n 'chmod -R 700 .ssh/'
        sshpass -p 'bigred' ssh $n 'rm ~/.ssh/known_hosts'
        sshpass -p 'bigred' ssh $n 'echo "PATH=.:$PATH" > ~/.ssh/environment' &>/dev/null
        echo "$n system prepare ok"
      else
        echo "$n .ssh created"  
      fi
    else
      echo "${n} is not alpine" 
    fi
  else
    echo "can not connect to ${n}"
  fi
done
```

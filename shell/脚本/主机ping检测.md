# 

checkhost.sh

```shell
#!/bin/bash
#
#===============================
# Description: Check hosts from filelist
# Last Update: 2017.9.22
# Version: 1.0
#===============================
start=`date +%s`
HLIST=$(cat ./ipadds.txt)
uphosts=0
downhosts=0

for IP in $HLIST
do
    ping -c 3 -i 0.2 -w 3 $IP &> /dev/null
    if [ $? -eq 0 ];then
        echo "Host $IP is up"
        #统计up的ip数
        let uphost++
    else
        echo "Host $IP is down"
         #统计down的ip数
        let downhost++
    fi
done

stop=`date +%s`
echo "Up hosts:$uphosts."
echo "Down hosts:$downhosts."
echo "IP addresses ($uphosts hosts up) scanned in $[ stop - start ] seconds"
```


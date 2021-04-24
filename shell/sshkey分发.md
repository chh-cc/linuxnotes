相同密码expect分发

vim ssh_key_distrubution.sh

```shell
#!/bin/bash
SOURCE=root
IP_FILE=$SOURCE/expect_scripts/ip_file
REMOTE_USER='root'
PASSWORD=''
SSH_KEY_FILE=/root/.ssh/id_rsa.pub

if [ -f $IP_FILE ];then
    echo "$IP_FILE not exists..."
    exit 1
fi
if [ -s $IP_FILE ];then
    echo "$IP_FILE is wrong..."
    exit 1
fi
if ! which expect &> /dev/null;then
    yum install expect -y &> /dev/null
    if [ $? -ne 0 ];then
        echo "yum install expect failed..."
        exit 1
    fi
fi

for HOST_IP in $(cat $IP_FILE)
do
    expect "$EXPECT_SCRIPT" "$SSH_KEY_FILE" "$REMOTE_USER" "$HOST_IP" "$PASSWORD"
    if [ $? -eq 0 ];then
        echo "$HOST_IP distrubute successfully..."
    else
        echo "$HOST_IP distrubute failed..."
    fi
done
```

vim ssh_key_expect.exp

```shell
#!/usr/bin/expect 

if { $argc != 4 } {
	send_user "usage:please check parameters number...\n"
	exit
}

set timeout 30
set file [lindex $argv 0]
set user [lindex $argv 1]
set host [lindex $argv 2]
set password [lindex $argv 3]

spawn ssh-copy-id -i $file "-p2222 $user@$host"
expect {
   "yes/no"	{send "yes\r";exp_continue}
   "*password" 	{send "$password\r"}
}
expect eof
exit
```


## ssh 免密登陆

```
#!/bin/bash
SERVERS="192.168.1.241 192.168.1.242"
PASSWD="123456"

function sshcopyid
{
    expect -c "
        set timeout -1;
        spawn ssh-copy-id $1;
        expect {
            \"yes/no\" { send \"yes\r\" ;exp_contine; }
            \"password:\" { send \"$PASSWD\r\";exp_continue; }
        };
        expect eof;
    "
}

for server in $SERVERS
do
    sshcopyid $server

done
```

用来进行免密登陆的

## linux xargs 命令的使用及其exec 、管道的区别






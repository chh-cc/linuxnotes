

```shell
#!/bin/bash
# Description: Only support RedHat system

WORD_DIR=/data/project/nginx1.10
DAEMON=$WORD_DIR/sbin/nginx
CONF=$WORD_DIR/conf/nginx.conf
NAME=nginx

PID=$(awk -F'[; ]+' '/^[^#]/{if($0~/pid;/)print $2}' $CONF)
#如果$PID为空
if [ -z "$PID" ]; then
PID=$WORD_DIR/logs/nginx.pid
else
PID=$WORD_DIR/$PID
fi

stop() {
    #停止nginx
    $DAEMON -s stop
    #等待1秒
    sleep 1
    [ ! -f $PID ] && action "* Stopping $NAME" /bin/true || action "* Stopping $NAME" /bin/false
}

start() {
    #启动nginx
    $DAEMON
    sleep 1
    [ -f $PID ] && action "* Starting $NAME" /bin/true || action "* Starting $NAME" /bin/false
}

reload() {
    $DAEMON -s reload
}

test_config() {
    $DAEMON -t
}

case "$1" in
    start)
        #如果$PID文件不存在
        if [ ! -f $PID ];then
            start
        else
            echo "$NAME is running"
            exit 0
        fi
        ;;
    stop)
        if [ ! -f $PID ];then
            stop
        else
            echo "$NAME not running"
            exit 0
        fi
        ;;
    restart)
        if [ ! -f $PID ];then
            echo "$NAME is running"
            start
        else
            stop
            start
        fi
        ;;
    reload)
        reload
        ;;
    testconfig)
        test_config
        ;;
    status)
        [ -f $PID ] && echo "$NAME is running..." || echo "$NAME not running!"
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|reload|testconfig|status}"
        exit 1
        ;;
esac
```


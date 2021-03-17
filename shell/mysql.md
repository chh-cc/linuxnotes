### 全量备份

```shell
#!/bin/bash
#全量备份，只备份一次
#指定备份目录
backup_dir="/bak/mysql-xback"
#检查
[[ -d ${backup_dir} ]] || mkdir -p ${backup_dir}
if [[ -d ${backup_dir}/all-backup ]];then
    echo "全备份已存在"
    exit 1
fi
#命令，需要设置
innobackupex --defaults-file=/etc/my.cnf --user=back --password='123456' --no-timestamp ${backup_dir}/all-backup &> /tmp/mysql-backup.log
tail -n 1  /tmp/mysql-backup.log | grep 'completed OK!'
if [[ $? -eq 0 ]];then
    echo "all-backup" > /tmp/mysql-backup.txt
else
    echo "备份失败"
    exit 1
fi
```

### 增量备份

```shell
#!/bin/bash
#增量备份
#备份目录
backup_dir="/bak/mysql-xback"
#新旧备份
old_dir=`cat /tmp/mysql-backup.txt`
new_dir=`date +%F-%H-%M-%S`
#检查
if [[ ! -d ${backup_dir}/all-backup ]];then
    echo "还未全量备份"
    exit 1
fi
#命令
/usr/bin/innobackupex --user=back --password='123456' --no-timestamp --incremental --incremental-basedir=${backup_dir}/${old_dir} ${backup_dir}/${new_dir} &> /tmp/mysql-backup.log
tail -n 1  /tmp/mysql-backup.log | grep 'completed OK!'
if [[ $? -eq 0 ]];then
    echo "${new_dir}" > /tmp/mysql-backup.txt
else
    echo "备份失败"
    exit 1
fi
```

备份binlog

```shell
#!/bin/bash
#
# 注意：执行脚本前修改脚本中的变量
# 功能：cp方式增量备份
#
# 适用：centos6+
# 语言：中文
#
#使用：./xx.sh -uroot -p'123456'，将第一次增量备份后的binlog文件名写到/tmp/binlog-section中，若都没有，自动填写mysql-bin.000001
#过程：增量先刷新binlog日志，再查询/tmp/binlog-section中记录的上一次备份中最新的binlog日志的值
#      cp中间的binlog日志，并进行压缩。再将备份中最新的binlog日志写入。
#恢复：先进行全量恢复，再根据全量备份附带的time-binlog.txt中的记录逐个恢复。当前最新的Binlog日志要去掉有问题的语句，例如drop等。
#[变量]
#mysql这个命令所在绝对路径
my_sql="/usr/local/mysql/bin/mysql"
#mysqldump命令所在绝对路径
bak_sql="/usr/local/mysql/bin/mysqldump"
#binlog日志所在目录
binlog_dir=/usr/local/mysql/data
#mysql-bin.index文件所在位置
binlog_index=${binlog_dir}/mysql-bin.index
#备份到哪个目录
bak_dir=/bak/mysql-binback
#这个脚本的日志输出到哪个文件
log_dir=/tmp/mybak-binlog.log
#保存的天数，4周就是28天
save_day=10
#[自动变量]
#当前年
date_nian=`date +%Y-`
begin_time=`date +%F-%H-%M-%S`
#所有天数的数组
save_day_zu=($(for i in `seq 1 ${save_day}`;do date -d -${i}days "+%F";done))
#开始
/usr/bin/echo >> ${log_dir}
/usr/bin/echo "time:$(date +%F-%H-%M-%S) info:开始增量备份" >> ${log_dir}
#检查
${my_sql} $* -e "show databases;" &> /tmp/info_error.txt
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:登陆命令错误" >> ${log_dir}
    /usr/bin/cat /tmp/info_error.txt #如果错误则显示错误信息
    exit 1
fi
#移动到目录
cd ${bak_dir}
bak_time=`date +%F-%H-%M`
bak_timetwo=`date +%F`
#刷新
${my_sql} $* -e "flush logs"
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:刷新binlog失败" >> ${log_dir}
    exit 1
fi
#获取开头和结尾binlog名字
last_bin=`cat /tmp/binlog-section`
next_bin=`tail -n 1 ${binlog_dir}/mysql-bin.index`
echo ${last_bin} |grep 'mysql-bin' &> /dev/null
if [[ $? -ne 0 ]];then
    echo "mysql-bin.000001" > /tmp/binlog-section #不存在则默认第一个
    last_bin=`cat /tmp/binlog-section`
fi
#截取需要备份的binlog行数
a=`/usr/bin/sort ${binlog_dir}/mysql-bin.index | uniq | grep -n ${last_bin} | awk -F':' '{print $1}'`
b=`/usr/bin/sort ${binlog_dir}/mysql-bin.index | uniq | grep -n ${next_bin} | awk -F':' '{print $1}'`
let b--
#输出最新节点
/usr/bin/echo "${next_bin}" > /tmp/binlog-section
#创建文件
rm -rf mybak-section-${bak_time}
/usr/bin/mkdir mybak-section-${bak_time}
for i in `sed -n "${a},${b}p" ${binlog_dir}/mysql-bin.index  | awk -F'./' '{print $2}'`
do
    if [[ ! -f ${binlog_dir}/${i} ]];then
        /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:binlog文件${i} 不存在" >> ${log_dir}
        exit 1
    fi
    cp -rf ${binlog_dir}/${i} mybak-section-${bak_time}/
    if [[ ! -f mybak-section-${bak_time}/${i} ]];then
        /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:binlog文件${i} 备份失败" >> ${log_dir}
        exit 1
    fi
done
#压缩
if [[ -f mybak-section-${bak_time}.tar.gz ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:压缩包mybak-section-${bak_time}.tar.gz 已存在" >> ${log_dir}
    /usr/bin/rm -irf mybak-section-${bak_time}.tar.gz
fi
/usr/bin/tar -cf mybak-section-${bak_time}.tar.gz mybak-section-${bak_time}
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:压缩失败" >> ${log_dir}
    exit 1
fi
#删除binlog文件夹
/usr/bin/rm -irf mybak-section-${bak_time}
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:删除sql文件失败" >> ${log_dir}
    exit 1
fi
#整理压缩的日志文件
for i in `ls | grep "^mybak-section.*tar.gz$"`
   do
    echo $i | grep ${date_nian} &> /dev/null
        if [[ $? -eq 0 ]];then
            a=`echo ${i%%.tar.gz}`
            b=`echo ${a:(-16)}` #当前日志年月日
            c=`echo ${b%-*}`
            d=`echo ${c%-*}`
            #看是否在数组中，不在其中，并且不是当前时间，则删除。
            echo ${save_day_zu[*]} |grep -w $d &> /dev/null
            if [[ $? -ne 0 ]];then
                [[ "$d" != "$bak_timetwo" ]] && rm -rf $i
            fi
        else
            #不是当月的，其他类型压缩包，跳过
            continue
        fi
done
#结束
last_time=`date +%F-%H-%M-%S`
/usr/bin/echo "begin_time:${begin_time}   last_time:${last_time}" >> ${log_dir}
/usr/bin/echo "time:$(date +%F-%H-%M-%S) info:增量备份完成" >> ${log_dir}
/usr/bin/echo >> ${log_dir}
```

### 重写备份

```shell
#!/bin/bash
#xbak备份脚本
#每周六执行一次
#10 4 * * 6 /bin/bash /root/bin/mybak-rewrite.sh
#清理并备份
[[ -d /bak/xback ]] || mkdir -p /bak/xback
cd /bak/xback
rm -rf *.tar.gz
[[ -d bak/mysql-xback ]] || echo "bak-dir not found"
cd /bak/mysql-xback
tar -cf XtraBackup.tar.gz *
mv XtraBackup.tar.gz /bak/xback
rm -rf  /bak/mysql-xback/*
#全备份一次
bash /root/bin/mybak-all.sh
```



innobackup

```shell
#!/bin/sh


# 注意此脚本不适合按天做增量备份
# --no-timestamp:
# 这个选项不创建一个时间戳的目录来存储备份
# --use-memory:
# 通过使用更多的内存，准备过程可以加快速度。它依赖于您的系统上的免费或可用RAM，默认为100MB。
# 一般来说，进程的内存越多越好。进程中使用的内存的数量可以由多个字节来指定:
# --compress:
# 压缩选项
# --compress-threads:
# 压缩线程
# --incremental-basedir:
# 增量备份的基础路径(上次备份路径)
# --incremental:
# 增量备份选项

BAK_PATH=/data/database_backup
CURRENT_DATE=$(date +%Y%m%d)
LOG_FILE="mysql_back_info_${CURRENT_DATE}.log"
LOG_DIR='/var/log/mysql_back'
USER='root'
PASS='root'
CONF_FILE='/etc/my.cnf'
#SOCK_FILE='/data/db/mysql/var/mysql.sock'
SOCK_FILE='/var/lib/mysql/mysql.sock'
#DATA_DIR='/data/db/mysql/var'
DATA_DIR='/var/lib/mysql'
BASE_DIR=${BAK_PATH}/${CURRENT_DATE}
SCRIPT_DIR='/root/scripts'
BINLOG_DIR='/data/db/mysql_binlog'
# BINLOG_BIN='/usr/local/lnmp/mysql/bin/mysqlbinlog'

if [ "$UID" -ne 0 ];then
    echo "You must run as root"
    exit
fi
if [ ! -d ${LOG_DIR} ];then
    mkdir -p ${LOG_DIR}
fi

do_install(){
    yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm -y
    yum install percona-xtrabackup-24 -y
}

do_full_backup() {
    if [ ! -d ${BASE_DIR} ];then
        mkdir -p ${BASE_DIR}
    fi
    #全备
    innobackupex \
    --defaults-file=${CONF_FILE} \
    --user=${USER} \
    --password=${PASS} \
    --use-memory=4G \
    --compress \
    --compress-threads=4 \
    -S ${SOCK_FILE} \
    ${BASE_DIR} >> ${LOG_DIR}/${LOG_FILE} 2>&1
    
    num=$?
    if [ "$num" -eq 0 ];then
        echo "full backup OK."
    else
        echo "full backup FAIL"
        exit ${num}
    fi
    
    do_write_file
}

do_write_file(){
    if [ ! -d ${SCRIPT_DIR}/${CURRENT_DATE} ];then
        mkdir -p ${SCRIPT_DIR}/${CURRENT_DATE}
    fi

    ls -rt1 ${BASE_DIR} | tail -n 1 > ${SCRIPT_DIR}/${CURRENT_DATE}/back.lock
}

do_read_file() {
    echo $(cat ${SCRIPT_DIR}/${CURRENT_DATE}/back.lock)
}

do_inc_backup(){
    LAST_BACKUP_DIR=`do_read_file`
    if [ -z ${LAST_BACKUP_DIR} ];then
        echo "Not found last_backup_dir.data file"
        exit 1
    else
        innobackupex \
        --defaults-file=${CONF_FILE} \
        --user=${USER} \
        --password=${PASS} \
        -S ${SOCK_FILE} \
        --use-memory=4G \
        --compress \
        --compress-threads=4 \
        --incremental-basedir=${BASE_DIR}/${LAST_BACKUP_DIR} \
        --incremental \
        ${BASE_DIR} >> ${LOG_DIR}/${LOG_FILE} 2>&1

        num="$?"
        if [ "${num}" -eq 0 ];then
            echo "INC backup Success."
            do_write_file
        else
            echo "INC backup Failed."
            exit ${num}
        fi
    fi
}


do_write_list(){
    if [ -d ${BAK_PATH}/$1 ]; then
        if test -f ${SCRIPT_DIR}/list.lock;then
            echo "list.lock存在" >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
            exit 1
        fi

        ls -tr1  ${BAK_PATH}/$1 > ${SCRIPT_DIR}/list.lock
    else
        exit 1
    fi
}

do_merge_backup(){
    RECOVERY_INC_DIR=$1
    SUB_BACKUP_DIR=$2
    END_NR_NUM=$(grep -n "$RECOVERY_INC_DIR" ${SCRIPT_DIR}/list.lock | awk -F':' '{print $1}')
    if [ -f ${SCRIPT_DIR}/list.lock ];then
        for ((counter=1; counter<=${END_NR_NUM}; ++counter))
        do
            DIR_NAME=$(cat ${SCRIPT_DIR}/list.lock | awk 'NR=="'${counter}'"{print}')
            if [ ${counter} -eq 1 ];then
            # 取回全量备份
                innobackupex \
                --decompress \
                --parallel=4 \
                ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
                find  ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} -name "*.qp" -delete
                innobackupex \
                --apply-log \
                --redo-only \
                --use-memory=4G \
                ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
                # 定义全量备份目录变量
                FULL_BACKUP_DIR=${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME}
            elif [ ${counter} -eq ${END_NR_NUM} ];then
            # 合并最后一个增量，不需要--redo-only参数
                innobackupex \
                --decompress \
                --parallel=4 \
                ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
                find ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} -name "*.qp" -delete
                innobackupex \
                --apply-log \
                --use-memory=4G \
                ${FULL_BACKUP_DIR} \
                --incremental-dir=${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
            else
            # 合并非全量备份和非最后一次的增量备份
                innobackupex \
                --decompress \
                --parallel=4 \
                ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
                find ${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} -name "*.qp" -delete
                innobackupex \
                --apply-log \
                --use-memory=4G \
                --redo-only \
                ${FULL_BACKUP_DIR} \
                --incremental-dir=${BAK_PATH}/${SUB_BACKUP_DIR}/${DIR_NAME} \
                >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
            fi
        done
    fi
}

do_packaging_backup(){
    innobackupex \
    --defaults-file=${CONF_FILE} \
    --user=${USER} \
    --password=${PASS} \
    -S ${SOCK_FILE} \
    --stream=tar ./ | gzip -> ${BAK_PATH}/${CURRENT_DATE}_all.tar.gz
}

do_recovery(){
    SUB_BACKUP_DIR=$1
    RECOVERY_INC_DIR=$2
    do_merge_backup ${RECOVERY_INC_DIR} ${SUB_BACKUP_DIR}
    if [ "$?" -eq 0 ];then
        echo "Merge SUCC"
    else
        echo "Merge FAIL"
        exit 1
    fi
    # 关闭mysql服务
    service mysqld stop
    # 获取全量备份目录
    FULL_BACKUP_DIR=$(cat ${SCRIPT_DIR}/list.lock | awk 'NR==1{print}')
    # 回滚所有未提交的事务
    innobackupex \
    --apply-log \
    ${BAK_PATH}/${SUB_BACKUP_DIR}/${FULL_BACKUP_DIR} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1

    if [ $? -eq 0 ];then
        echo "开始回滚所有未提交的事务" >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
    else
        echo -e "开始回滚所有未提交的事务失败，请尝试手动执行命令: \ninnobackupex --apply-log ${BAK_PATH}/${SUB_BACKUP_DIR}/${FULL_BACKUP_DIR}" >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
        exit 1
    fi

    if test -d ${DATA_DIR}; then
        echo -e "${DATA_DIR}目录存在, 在回写时会失败，这里先退出，请手动删除数据目录并手动执行: \ninnobackupex --copy-back ${BAK_PATH}/${SUB_BACKUP_DIR}/${FULL_BACKUP_DIR}" >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
        exit 1
    fi
    # 拷回数据至数据目录
    innobackupex \
    --copy-back \
    ${BAK_PATH}/${SUB_BACKUP_DIR}/${FULL_BACKUP_DIR} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1

    if [ $? -eq 0 ];then
        chown -R mysql.mysql ${DATA_DIR}
        service mysqld start
    else
        exit 1
    fi
}

do_replay_binlog(){
    SUB_BACKUP_DIR=$1
    STOP_TIME=$2
    RECOVERY_INC_DIR=$3
    RECOVERY_NEX_DIR=$4
    RECOVERY_DAY=$5
    STOP_DATETIME="${RECOVERY_DAY} ${STOP_TIME}"

    if test -f ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_INC_DIR}/xtrabackup_binlog_info; then
         begin_file=$(cat ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_INC_DIR}/xtrabackup_binlog_info | awk '{print $1}')
         begin_position=$(cat ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_INC_DIR}/xtrabackup_binlog_info | awk '{print $2}')
     else
         exit 1
     fi

    if [ ! -z ${RECOVERY_NEX_DIR} ];then
        if test -f  ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_NEX_DIR}/xtrabackup_binlog_info.qp;then
            innobackupex \
            --decompress \
            --parallel=4 \
            ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_NEX_DIR} >> ${LOG_DIR}/recovery_info_${CURRENT_DATE}.log 2>&1
            find ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_NEX_DIR} -name "*.qp" -delete
        fi
        end_file=$(cat ${BAK_PATH}/${SUB_BACKUP_DIR}/${RECOVERY_NEX_DIR}/xtrabackup_binlog_info | awk '{print $1}')
    else
        end_file=${begin_file}
    fi
    mysqlbinlog ${BINLOG_DIR}/${begin_file} \
                ${BINLOG_DIR}/${end_file} \
                --start-position=${begin_position} \
                --stop-datetime="${STOP_DATETIME}"| mysql -uroot -p

    if [ "$?" -eq 0 ];then
        echo "recovery Success"
    else
        echo "recovery Failed."
        exit 1
    fi
}

do_recovery_main(){
    read -p "请准备好数据库管理员密码，后面要用到，这里请输入时间[2018-02-23 17-10-10]: " RECOVERY_DAY RECOVERY_TIME
    STOP_TIME=$(echo ${RECOVERY_TIME} | awk '{gsub("-", ":");print $1$2$3}')
    SUB_BACKUP_DIR=$(echo ${RECOVERY_DAY} | awk -F '-' '{print $1$2$3}')
    RECOVERY_H=$(echo ${RECOVERY_TIME} | awk -F'-' '{print $1}')
    RECOVERY_M=$(echo ${RECOVERY_TIME} | awk -F'-' '{print $2}')
    # 这里需要注意一下，这里方便测试备份时间间隔精确至分钟，如果增量按时备份，请在此处作相应修改, 比如按小时备份:
    # REGEX_STRING=${RECOVERY_DAY}_${RECOVERY_H}-[0-5][0-9]-[0-5][0-9]
    REGEX_STRING=${RECOVERY_DAY}_${RECOVERY_H}-${RECOVERY_M:0:1}[0-9]
    do_write_list ${SUB_BACKUP_DIR}
    RECOVERY_INC_DIR=$(cat ${SCRIPT_DIR}/list.lock | awk '/'${REGEX_STRING}'/ {print}')
    RECOVERY_NEX_DIR=$(cat ${SCRIPT_DIR}/list.lock | awk '/'${REGEX_STRING}'/ {nr[NR+1]; next};NR in nr')
    do_recovery ${SUB_BACKUP_DIR} ${RECOVERY_INC_DIR}
    while :
    do
        checkPidExists
        num=$?
        if [ ${num} -eq 0 ];then
            break
        fi
    done

    do_replay_binlog ${SUB_BACKUP_DIR} ${STOP_TIME} ${RECOVERY_INC_DIR} ${RECOVERY_NEX_DIR} ${RECOVERY_DAY}
    if [ $? -eq 0 ];then
        rm -rf ${SCRIPT_DIR}/list.lock
    fi
}

checkPidExists() {
    if [ -z `pidof mysqld` ];then
        return 1
    else
        return 0
    fi
}

case $1 in
    full)
    do_full_backup
    ;;
    inc)
    do_inc_backup
    ;;
    install)
    do_install
    ;;
    packaging)
    do_packaging_backup
    ;;
    recovery)
    do_recovery_main
    ;;
    *)
    echo "Usage: $0 <params: [full|inc|recovery|packaging]>"
    echo
    echo "full: 全量备份"
    echo "inc: 增量备份"
    echo "recovery: 恢复备份"
    echo "packaging: 打包备份"
    echo
    ;;
esac
```


crontab -u 用户名 -e               # 编辑周期任务



crontab格式：

分钟  小时    天  月  星期   命令或脚本

o minute: 区间为 0 – 59 
o hour: 区间为0 – 23 
o day-of-month: 区间为0 – 31 
o month: 区间为1 – 12. 1 是1月. 12是12月. 
o Day-of-week: 区间为0 – 7. 周日可以是0或7.



Crontab 示例

1. 在 12:01 a.m 运行，即每天凌晨过一分钟。这是一个恰当的进行备份的时间，因为此时系统负载不大。

1 0 * * * /root/bin/backup.sh

2. 每个工作日(Mon – Fri) 11:59 p.m 都进行备份作业。

59 11 * * 1,2,3,4,5 /root/bin/backup.sh

下面例子与上面的例子效果一样：

59 11 * * 1-5 /root/bin/backup.sh

3. 每5分钟运行一次命令

*/5 * * * * /root/bin/check-status.sh

4. 每个月的第一天 1:10 p.m 运行

10 13 1 * * /root/bin/full-backup.sh

5. 每个工作日 11 p.m 运行。

0 23 * * 1-5 /root/bin/incremental-backup.sh
# zabbix微信报警

1. 微信企业号注册

   企业微信注册地址：https://work.weixin.qq.com/，填写企业信息注册企业号

2. 通讯录添加运维部门及人员

   <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210729224826.png" alt="img" style="zoom:67%;" />

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210729224927.png" alt="img" style="zoom:67%;" />

3. 创建应用

   应用管理→创建应用

   <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210729225018.png" alt="img" style="zoom:67%;" />

   <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210729225033.png" alt="img" style="zoom:67%;" />

4. 获取企业CorpID，点击我的企业即可看到

   ![img](https://gitee.com/c_honghui/picture/raw/master/img/20210729225118.png)

5. 微信接口调试

   调用微信接口需要一个调用接口的凭证：Access_token通过CorpID和Secret可以获得Access_token

   微信企业号接口调试地址：https://qy.weixin.qq.com/cgi-bin/debug

   <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210729225154.png" alt="img" style="zoom:67%;" />

6. 获取微信告警工具

   ```shell
   mkdir /etc/zabbix/alertscripts/
   cd  /etc/zabbix/alertscripts/
   wget https://dl.cactifans.com/tools/zabbix_weixin.x86_64.tar.gz
   tar -xzvf zabbix_weixin.x86_64.tar.gz
   mv zabbix_weixin/weixin .
   chmod o+x weixin
   mv zabbix_weixin/weixincfg.json /etc/
   rm -rf zabbix_weixin.x86_64.tar.gz 
   rm -rf zabbix_weixin/
   ```

   修改配置文件

   ```shell
   vim /etc/weixincfg.json
   {
   "corp": {
           "corpid": "ww8b7dff43e65901aa",
           "secret": "3d6SqLI9MxCiBAUUtkf3LoMQs2K1ZeGdsYar0QaEj4I",
           "agentid": 1000002
       }
   }
   ```

   测试

   ```shell
   [root@zabbix-server alertscripts]# ./weixin hhh subject body
   ok
   ```

7. 添加报警媒介

   ![](https://gitee.com/c_honghui/picture/raw/master/img/20210729233433.png)

   三个参数：{ALERT.SENDTO} {ALERT.SUBJECT} {ALERT.MESSAGE}

8. 设置微信报警动作

9. 设置收件人

   ![img](https://gitee.com/c_honghui/picture/raw/master/img/20210729233525.png)

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210729233535.png)

测试：在监控端把agentd服务停止

结果:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210729233604.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210729233610.jpg" alt="img" style="zoom: 25%;" />
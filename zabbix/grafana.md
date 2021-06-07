# grafana

Grafana默认没有zabbix作为数据源，需要手动给zabbix安装一个插件，然后再添加进Grafana即可

安装grafana:

```shell
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-6.3.6-1.x86_64.rpm
yum localinstall grafana-6.3.6-1.x86_64.rpm
systemctl daemon-reload
systemctl start grafana-server.service
```

命令安装zabbix插件,但可能会出现版本冲突,建议手动安装插件:

```shell
# 获取可用插件列表
#grafana-cli plugins list-remote
#grafana-cli plugins install alexanderzobnin-zabbix-app
#systemctl restart grafana-server.servic

#手动安装插件
登陆https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/?tab=installation,因为这里我安装的是grafana6,所以选择3.12.0版本的插件,要根据grafana的版本选择插件版本
把下载好的插件上传到/var/lib/grafana/plugins,解压
unzip alexanderzobnin-grafana-zabbix-v3.12.0-1-gd0e8319.zip
systemctl restart grafana-server.servic
```

```text
修改图形为饼状，需要下载另一个grafana-piechart-panel 
https://grafana.com/plugins/grafana-piechart-panel
--------------------------------------------------
grafana-cli plugins install grafana-piechart-panel
---------------------------------------------------
安装其他图形插件
grafana-cli plugins install grafana-clock-panel
#钟表形展示
grafana-cli plugins install briangann-gauge-panel
#字符型展示
grafana-cli plugins install natel-discrete-panel
#服务器状态
grafana-cli plugins install vonage-status-panel
```

访问grafana，http://localhost:3000，默认用户名和密码：admin/admin

启动插件：

plugin->zabbix->enable

添加数据源：

"Data Sources"->"Add data source"

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210526172009.png)
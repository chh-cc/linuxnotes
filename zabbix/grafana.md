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

添加zabbix的API接口、认证信息：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210526172009.png)

创建DashBoard（仪表盘）：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210608221549.png" alt="image-20210608221542828" style="zoom:67%;" />

添加新的仪表盘：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210608221634.png" alt="image-20210608221633982" style="zoom:67%;" />

单击标题面板可打开一个菜单框。单击edit 选项面板将会打开额外的配置选项：

Genera（常规选择）：添加图形标题，图形宽度高度等

| 配置项目                | 说明               |
| ----------------------- | ------------------ |
| Title                   | 仪表板上的面板标题 |
| Span                    | 面板背景透明化     |
| Drilldown / detail link | 钻取/详细信息链接  |

Metrics（指标）

| 配置项目    | 说明                                                     |
| ----------- | -------------------------------------------------------- |
| Data Source | 数据来源，因为是zabbix插件，自然来自zabbix，支持使用变量 |
| Group       | zabbix中设置的主机群组，支持使用变量                     |
| Host        | zabbix中设置的主机名，支持使用变量                       |
| Application | zabbix中的应用集，可以在下拉菜单中自由选择               |
| Item        | zabbix中的监控项，支持正则匹配多个监控项                 |

Axes（坐标轴）

用于坐标轴和网格的显示方式，包括单位，比例，标签等

| 设置项目 | 说明                    |
| -------- | ----------------------- |
| Unit     | 单位                    |
| Mode     | Time\|Series\|Histogram |

Legend（图例）：图例展示

| 图例的参数   | 说明                                           |
| ------------ | ---------------------------------------------- |
| Show         | 显示查询的参数名称                             |
| As Table     | 以表格是形似显示                               |
| To the right | 在右边显示                                     |
| Total        | 返回所有度量查询值的总和                       |
| Current      | 返回度量查询的最后一个值                       |
| Min          | 返回最小的度量查询值                           |
| Max          | 返回最大的度量查询值                           |
| Avg          | 返回所有度量查询的平均值                       |
| Decimals     | 控制Legend值的多少，以小数显示悬浮工具提示(图) |

Display（显示样式）

Time range（时间范围）

仪表盘变量：

单纯的手动去添加一个个监控图,只能显示一个主机的所有监控图形，若要查看不同主机的所有监控图形，就要通过变量的方式去实现。我们要设置的变量包括group，host，application和iteam。

仪表盘模板可以让你创建一个交互式和动态性的仪表板，它是Grafana里面最强大的、最常用的功能之一。创建的仪表盘模板参数，可以在任何一个仪表盘中使用。
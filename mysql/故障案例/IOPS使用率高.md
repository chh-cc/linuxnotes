原因:

- 实例**内存**满足不了缓存数据或排序等需要,导致产生大量物理IO
- 查询执行效率低,扫描过多数据行

解决:

方法一:

生成实例当前诊断报告（推荐方式）

在左侧导航栏中，选择***\*自治服务\** > \**诊断报告\****。

在 DMS 中生成当前的实例诊断报告，查看其中的 SQL优化、会话列表、慢 SQL 汇总部分，建议应用 SQL 优化给出的意见。

方法二：

dms->实例信息->实例会话 或 show full processlist;查看正在运行的查询，通过sql窗口优化功能优化查询
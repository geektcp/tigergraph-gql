# 概述
```
图数据库目前发展到第三代
第一代以neo4j为代表
第二代以Amazon Neptune为代表
第三代以tigergraph，arangodb为代表
```

# 名词解释
## 基本概念
```
gadmin                  管理命令，类似mysqladmin
gbar                    备份和存储命令，backup and resotre
GSQL Shell              用于执行sql语句，类似mysql数据库的mysql命令
GraphStudio UI          可视化控制台


GPE                     图数据库计算引擎
Graph Store             内置的一个基于内存的数据存储组件
DICT	                数据分片，类似arangodb的内部的分片机制
GSE                     存储引擎，GSE通过GPE接收操作数据，对Graph Store进行增删改查

GSQL Language           图数据库SQL，类似arangodb的aql
GSQL                    一个内部接口，GSQL Shell调用了这个接口执行图SQL
HA                      tigergraph支持高可用
IDS		                GSE的内部组件，用于把顶点表、边表转换成图
IUM		                安装，升级，维护
MultiGraph              复杂图架构，支持多个子图
Native Parallel Graph   并行图架构，支持并行存储和分析，高并发，高伸缩性
Nginx                   tigergraph的前端
REST++,RESTPPrest api   增加、删除和查询，没有编辑
SSO		
TigerGraph Platform
TigerGraph System

Kakfa
Zookeeper
```
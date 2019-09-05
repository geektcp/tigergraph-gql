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

## 边的方向性
```
tiger的边分为有向边，无相边。
有向边和无相边的区别是：界面上有向边有明显箭头，无向边就是直线。

有向边又有单向边，双向边。
无向边由于指定了边的起点类型和终点类型，而任何顶点的类型只能有一个，无所无向边其实还是有方向的。

只有起点和终点类型都是*的时候，才是真正的无向边。

单向边无法反向遍历。

无相边和双向边都可以反向遍历。
无向边的反向遍历结果相同。
双向边的反向遍历结果中边的类型不同。
```

## 双向边和有向边的本质差别
```
双向边其实无向边效果差不多。
双向边和有向边的本质差别在于查询方向。双向边可以指定两个方向查询，有向边根本不能反方向查询。
原因是tiger的查询语法范式定死了格式。
SELECT t FROM vSetVarName:s – ((eType1|eType2):e) -> (vType1|vType2):t

第一个横杠-前面必须是顶点集合（冒号后面是别名，别名可以省略）
依据这个范式，最精简的写法是：
source = SELECT s FROM source:s -()-> ;

举例说明：
source = SELECT t FROM source:s -(:e)-> :t
上面的写法无法写成（因为dest是顶点集合，不是顶点类型）：
dest = SELECT t FROM :s -(:e)-> dest:t
```

## tiger和arango边的方向性
```
arangodb里面的边都是有向边，但是遍历方向是可以指定的，也是支持反向遍历。
FOR vertext[] IN []
FOR v,e,p in[1..3]
OUTBOUND|INBOUND|ANY startVertex [EDGES]
FILTER ...

tiger的GSQL里面要实现arangodb的AQL中的设定方向的效果，最好的方式是创建边的时候就定下来这条边是单向边还双向边。如果是单向边，创建边的时候就定好了的出的还是进的，即OUTBOUND还是INBOUND的。
如果创建边的时候指定双向边，那么遍历就是ANY方式。
在tiger的query里面不能再直接指定边的方向性了。


另外，如果都是双向边，这是遍历默认相当于arangodb的ANY方向，如何指定方向呢？
```
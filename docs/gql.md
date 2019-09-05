# GSQL

## 语法
```
声明顶点
VERTEX<person> p1;
startpoint = to_vertex("c4", "company");
```

# 定义元组
```
(必须在累加器前面定义，参考C++的语法规则)：
TYPEDEF TUPLE <VERTEX v, EDGE e> CHILD;
```

声明累加器
```
OrAccum @@found  = false;
OrAccum @notSeen = true;
ListAccum<VERTEX> @childsVetex;
ListAccum<EDGE> @childsEdges;
```

赋值顶点
```
（和声明分开）：
startpoint = to_vertex("c4", "company");
```

## filter关键字
```
跟arangodb的filter不同，这不是一个通用过滤器，只能在伴随neighbors和neighborAttribute这两个函数时过滤,
而且只能在ACCUM语句块中使用：
s.neighbors().filter(true),
v.@diffCountry += v.neighborAttribute("worksFor", "company", "id")
.filter(v.locationId != company.country),
```

## JSONARRAY转LIST
```
JSONARRAY arr;
arr = filterObject.getJsonArray("filterEdgeType");
j = 0;
WHILE(j<arr.size()) DO
@@list += arr.getString(j);
j+=1;
END;
```

## ACCUM关键字
```
用于一阶累加

POST-ACCUM关键字
用于二阶累加
```

```
一个语法糖（官方文档没有明确解释，但有这个用法，见《运算符、函数和表达式->表达式声明》）：
二阶累加要调用一阶累加的变量时，加单引号即可。例如
  V = SELECT s
            FROM Start:s -(:e)-> :t
            ACCUM t.@received_score += s.@score/(s.outdegree() +1),
	                t.@test += 2,
	                test1 = s.@score
            POST-ACCUM// s.@score = (1.0-damping) + damping * s.@received_score,
                s.@received_score = 0,
	              s.@score = 11,
                @@maxDiff += abs(s.@score - s.@score'),
	              test2 = s.@score',
	              test3 = s.@score;

POST-ACCUM 里面的s.@score' 其实就是 ACCUM的变量s.@score
```

## 语法约束汇总
- 1、PRINT语句不能打印局部累加器；

- 2、仅当我们只选择了顶点集变量或顶点表达式集合中的某一部分的时候，PRINT才会打印出顶点的详细信息。 而当打印单个顶点时(即从一个变量或累加器获得数据，且它的数据类型是VERTEX)，只打印顶点id。

- 3、局部变量只能在ACCUM，POST-ACCUM或UPDATE SET子句中声明，并且其作用域也仅限于该子句。 局部变量只能是基本类(例如INT，FLOAT，DOUBLE，BOOL，STRING，VERTEX)，且必须在同一语句中声明的同时初始化局部变量

- 4、在局部变量的作用域内，同一级的子句中不能声明另一个相同名称的局部变量。 但是，一个新的与该变量同名的变量可以在低级别子句中(例如又嵌套了一层SELECT或UPDATE语句)使用。 即低级别的声明只在该级别工作。

- 5、声明顶点集变量时，可以选择将一组顶点类型指定给顶点集变量。 如果未明确指定顶点集变量的类型，则系统通过顶点集值隐含的类型来输出。 选择的类型可以是ANY（表示所有类型），下划线“_” (等效于ANY)或任何被明确说明的顶点类型。

- 6、在顶点集变量声明中，类型说明符跟在变量名后面，并且应该用括号括起来，如:vSetName(type)

- 7、通常赋值语句可以放在变量声明后的任何地方。然而，它仍然有两个限制。该限制适用于“内层”语句，即被包含在一个上层的语句块中的语句。

- 8、SELECT或UPDATE语句的正文内不允许使用全局累加器赋值，在ACCUM或POST-ACCUM子句中允许对全局变量赋值，但只有退出该子句后该值才生效。 因此，如果同一全局变量有多个赋值语句，则只有最后一个赋值语句才会生效。
ACCUM子句中不允许使用顶点属性赋值。 但是，允许对边属性赋值。 这是因为ACCUM子句是由边的集合迭代而来。

- 9、使用通用的VERTEX作为元素类型的任何累加器都不能由LOADACCUM()初始化。

- 10、在ACCUM子句中不允许更新顶点的属性值，因为这样的更新动作是并行运行的，同一个值可能会多次更新，从而导致输出的结果不确定。 如果顶点属性值的更新取决于边的属性值，请使用附属于顶点的累加器来保存对应的值，然后在POST-ACCUM子句中更新顶点的属性值。

- 11、元组必须在查询中的任何其他语句之前被首先定义。

- 12、申明一个顶点变量不要复制，单独一行再赋值。

- 13、evaluate传递的表达式字符串不能包含逗号。
影响是：tiger支持基础条件的复杂组合过滤条件。不过这些复杂条件要写死在存储过程里面。

- 14、tiger的逻辑运算符NOT里面不能使用顶点或者边的type字段，只能使用属性，其他运算符没有限制。
官方：NOT运算符不能与.type属性选择器结合使用。 要检查边或顶点类型是否不等于给定类型，请使用！=运算符

- 15、累加器的关键字都是区分大小写的，比如SetAccum不能写成SETACCUM。

- 16、to_vertex_set允许对应的顶点不存在，to_vertex则会导致存储过程执行失败，查不出数据。

- 17、本地导入数据的语句CREATE LOADING JOB里面，加载边时，边表的表结构中的起点和终点，必须是确定，不能通配符*，如果创建边表使用了通配符*，那么无法用CREATE LOADING JOB方式导入数据。

- 10、每个查询都被视为一个事务（transaction）。 因此，在整个查询操作完成（确认完成）之前，对于图数据的修改不会生效。因此，某个查询中的数据更改操作不会影响同一查询中的任何其他语句。


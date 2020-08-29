## MongoDB

#### 一、软件安装：

- 官网下载MongoDB，https://www.mongodb.com/

- 下载最新 4.0.6 版本，按照软件安装步骤进行安装

> 软件安装完成后会自动创建MongoDB的服务，不需要再手动配置。安装时可能出现服务启动未成功的提示，点击忽略，重启电脑后MongoDB的服务启动成功。

#### 二、MongoDB的基本操作：

##### 1、MongoDB的数据结构与关系型数据库结构对比：

| 关系型数据库术语/概念 | MongoDB术语/概念 |            解释/说明            |
| :-------------------: | :--------------: | :-----------------------------: |
|       Database        |     Database     |             数据库              |
|         Table         |    Collection    |          数据库表/集合          |
|          Row          |     Document     |         数据记录行/文档         |
|        Column         |      Field       |         数据列/数据字段         |
|         Index         |      Index       |              索引               |
|      Table joins      |        -         |      表关联/MongoDB不支持       |
|      Primary Key      |    Object ID     | 主键/MongoDB自动将_id设置为主键 |

##### 2、MongoDB 中的数据类型：

|   数据类型   |    说明    |                             解释                             |        举例         |
| :----------: | :--------: | :----------------------------------------------------------: | :-----------------: |
|     Null     |    空值    |                  表示空值或者未定义的 对象                   |     {"x" :null}     |
|   Boolean    |   布尔值   |               真 或 者 假 ： true 或 者 false                |     {"x" :true}     |
|   Integer    |    整数    | 整型数值。 用于存储数 值。 根据你所采用的服务 器， 可分为 32 位或 64 位。 |                     |
|    Double    |   浮点数   |                        双精度浮点值。                        |  {"x":3.14,"y":3}   |
|    String    |   字符串   |                         UTF-8 字符串                         |                     |
|    Symbol    |    符号    | 符号。 该数据类型基本上 等同于字符串类型， 但不 同的是， 它一般用于采用 特殊符号类型的语言。 |                     |
|   ObjectID   |  对象 ID   |                对象 ID。 用于创建文档 的 ID。                |  {"id":ObjectId()}  |
|     Date     |    日期    |     日期时间。 用 UNIX 时 间格式来存储当前日期 或时间。      | {"date":new Date()} |
|  Timestamp   |   时间戳   |                   从标准纪元开始的毫秒 数                    |                     |
|   Regular    | 正则表达式 |      文档中可以包含正则表 达式， 遵循 JavaScript 的语法      |  {"foo":/testdb/i}  |
|     Code     |    代码    |                 可 以 包 含 JavaScript 代码                  | {"x":function(){}}  |
|  Undefined   |   未定义   |                            已废弃                            |                     |
|    Array     |    数组    |                       值的集合或者列表                       | {"arr": ["a","b"]}  |
| Binary Data  |   二进制   |                     用于存储二进制数据。                     |                     |
|    Object    |  内嵌文档  |             文档可以作为文档中某 个 key 的 value             | {"x":{"foo":"bar"}} |
| Min/Max keys | 最小/大值  |  将一个值与 BSON（二进制的JSON）元素的最低值和最高值相比。   |                     |

##### 3、基本指令：

> 在MongoDB中数据库和集合都不需要手动创建，当我们创建文档时，如果文档所在的集合或数据库不存在会自动创建数据库和集合。

```shell
show dbs / show databases  --显示当前的所有数据库
```

```shell
use 数据库名			  --进入到指定的数据库
```

```shell
db						--查看当前所处的数据库
```

```shell
show collections		--查看数据库中的所有集合
```


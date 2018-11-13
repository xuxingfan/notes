# JSON格式数据转化为数据库表数据
- 数据库表存储json数据

为了提高系统响应速度，我们把一些数据统计的工作提前统计好，浓缩成json格式存到数据库，例如以下功能
1. 按月、季度、年统计机场吞吐量和中转量
2. 按月、季度、年统计各航司旅客量和中转量
3. 按月、季度、年统计城市中转量和中转量

- 数据库存json的有点
1. 提前按用户需求统计好结果，提高系统响应速度
2. 统计好结果后，可以把基础历史数据删除，节省空间
3. 不同类型的统计结果可以存储到同一张表，用type区分，避免每种统计建一张表。
4. JSON格式可以快速反序列化到实体，开发和展示方便。

- JSON格式逆向转化为表数据
由于我们的数据量比较大，统计生成的json文本也相对较大。我们无法用肉眼验证统计的准确性。所以在验证数据的准确性和查错的时候，有以下两种解决方式：
1. 写JAVA单元测试，把JSON格式数据反向解析成java实体，使用代码逻辑验证
2. 借助postgresql把JSON数据转化为表数据，使用sql语句的聚合函数验证

方法2相对快捷方便，下面介绍如何实现：

按城市\流向\同航司-不同航司 统计每天的中转旅客数json如下:

```json
[{
	"flowTo": "3",
	"flightDate": "2018-11-03",
	"deptTotalNum": 1339,
	"cityGroup": "TSN,LNJ",
	"transferTotalNum": 2,
	"deptCity": "TSN",
	"destCity": "LNJ",
	"sameFlightNum": 0,
	"differFlightNum": 2
}, {
	"flowTo": "1",
	"flightDate": "2018-11-03",
	"deptTotalNum": 79,
	"cityGroup": "CMB,CSX",
	"transferTotalNum": 1,
	"deptCity": "CMB",
	"destCity": "CSX",
	"sameFlightNum": 1,
	"differFlightNum": 0
}, {
	"flowTo": "2",
	"flightDate": "2018-11-03",
	"deptTotalNum": 147,
	"cityGroup": "BFJ,BKK",
	"transferTotalNum": 6,
	"deptCity": "BFJ",
	"destCity": "BKK",
	"sameFlightNum": 0,
	"differFlightNum": 6
}, {
	"flowTo": "3",
	"flightDate": "2018-11-03",
	"deptTotalNum": 197,
	"cityGroup": "KWL,XIC",
	"transferTotalNum": 1,
	"deptCity": "KWL",
	"destCity": "XIC",
	"sameFlightNum": 1,
	"differFlightNum": 0
}
#......此处省略20万 json
]
```
当我们需要验证数据是否正确时(上面所有中转数据之和==当天实际中转人数)，我们需要累加每项的结果。

- 使用postgresql自带的函数json_to_recordset把json转化为表数据。

```sql
select count_time,c.* from
t_transfer_countinfo,
json_to_recordset(data_content::json)as c(
    "flowTo" text,
	"flightDate" text,
	"deptTotalNum" int,
	"cityGroup" text,
	"transferTotalNum" int,
	"deptCity" text,
	"destCity" text,
	"sameFlightNum" int,
	"differFlightNum" int
)
```
结果如下：

![image](https://note.youdao.com/yws/api/personal/file/67B633446F1A418EA68ABB7D2E327B97?method=download&shareKey=2de580d2574ed92553fe12cc49500a13)

得到结果之后，我们就可以使用count,sum,max等聚合函数进行下一步分析。

备注:目前9.5以后的postgresql支持json以及jsonb格式,具体自行网上搜索。但是由于json和jsonb格式有最大长度限制，我们系统统计的内容相对较大不能使用jsonb格式存储，只能存text格式。使用text存储时，同样可以使用json_to_recordset转化,只是需要使用::json转一下。

## druid数据库连接查看
druid是一个为监控而生的数据库连接池,功能十分强大。api地址[https://github.com/alibaba/druid/wiki/Druid连接池介绍](https://github.com/alibaba/druid/wiki/Druid%E8%BF%9E%E6%8E%A5%E6%B1%A0%E4%BB%8B%E7%BB%8D)，网上可以搜到很多相关的配置。本次主要了解在非web工程下，如何看数据库的连接情况。

##### 使用shell脚本查看druid数据库连接
shell脚本如下(druidStat.sh)：

```shell
#!/bin/sh
export JAVA_HOME="/opt/app/java/jdk/jdk1.8.0_144"
"$JAVA_HOME/bin/java" -Dfile.encoding="UTF-8" -cp "druid-1.1.5.jar:$JAVA_HOME/lib/tools.jar" com.alibaba.druid.support.console.DruidStat  $@
```
###### 如何使用以上脚本？
1. 从脚本中可以看到我们首先需要下载druid-1.1.5.jar包上传到服务器
2. 把上面的脚本保存为 druidStat.sh,和jar包放在同一个目录下
3. 在终端运行sh druidStat.sh -help，我们就可以看到脚本api如下

```shell
Usage: druidStat -help | -sql -ds -act [-detail] [-id id] <pid> [refresh-interval]

参数: 
  -help             打印此帮助信息
  -sql              打印SQL统计数据
  -ds               打印DataSource统计数据
  -act              打印活动连接的堆栈信息
  -detail           打印统计数据的全部字段信息
  -id id            要打印的数据的具体id值
  pid               使用druid连接池的jvm进程id
  refresh-interval  自动刷新时间间隔, 以秒为单位

说明: 
  -sql,-ds,-act参数中要至少指定一种数据进行打印, 可以
    组合使用, 比如 -sql -ds 一起的话就打印两种统计数据
  -id id可以跟 -sql 或-ds组合, 比如  -sql -id 5 或 -ds -id 1086752
  pid必需指定, refresh-interval可选, 如不指定,则打印数据后退出
  pid和refresh-interval参数必需放在命令行的最后, 否则解析会出错

例子: 
  打印3983进程的sql 统计数据.
      >druidStat -sql 3983
  打印3983进程的ds统计数据.
      >druidStat -ds 3983
  打印3983进程的sql的id为10的详细统计数据.
      >druidStat -sql -id 10 -detail 3983
  打印3983进程的当前活动连接的堆栈信息
      >druidStat -act 3983
  打印3983进程的ds,sql,和act信息
      >druidStat -ds -sql -act 3983
  每隔5秒自动打印ds统计数据
      >druidStat -ds 3983 5
```

根据api我们就可以对数据库连接进行查看

首先使用ps -ef|grep "TransferBasicData-Passenger"查看java程序进程，然后使用druidStat -ds ${pid}就可以查看数据连接了
![druid.png](https://note.youdao.com/yws/api/personal/file/55FFCA862A744ADB9D223EC787EBE612?method=download&shareKey=8e80216649d6dfddad2f811a3ae797b9)

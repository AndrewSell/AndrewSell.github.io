# hive优化举例
1. 通过explain或者explain extended来查看执行计划：
```
select * from aa7;##普通select语句
```
 只不过extended可以打印hql语句的执行顺序。
2. stage相当于一个，mapreduce任务（可以是一个子查询，一个抽样，一个合并，一个limit），hive默认每次只执行一个stage，但是没有依赖关系的可以并行执行。

 一个hive的hql语句包含一个或者多个stage，多个之间的依赖越复杂，表明任务越复杂，执行效率越低。
3. limit的限制：
 + hive.limit.row.max.size=100000;#最大limit的row
 + hive.limit.optimize.limit.file=10;#最多操作10个文件
 + hive.limit.optimize.enable=false;#开启优化（建议true）
 + hive.limit.optimize.fetch.max=50000;#最大拉取

4. join设置
 + 永远小表驱动大表
 + 大表标识（```/*+STREAMTABLE(bt)*/```）
 + 开启map端的join
 + ..convert..
 + ..smalltable.filesize
 + join的on只支持等值连接
5. 本地模式
 + hive查询数据还是依靠hadoop，hadoop有单机版，分布式，HA分布式
 + hive.exec.mode.local.auto=false;#本地模式（建议true）
 + hive.exec.mode.local.auto.inputbyte.max=134217728;#超过128m不启用本地模式
 + hive.exec.mode.local.auto.input.file.max=4;#本地模式下最大操作文件数
6. parallel并行设置
 + hive没有相互依赖的任务可以并行执行
 + hive.exec.parallel=false;#建议开启
 + hive.exec.parallel.thread.number=8;#最大线程数量8
7. strict 严格模式
 + hive的严格模式将会阻止5类查询

8. 调整mapper或者reducer的个数：
 + mapred.max.split.size=256000000;#分片256m
 + dfs.block.size=;#调整hdfs的分块大小
 + hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
 + 直接设置mapper的个数（mapred-site.xml）
 + 直接设置reducer的个数
 > mapred.reduce.

    > hive.exec.reducers.byte.per.reducer=256000000;#单个reduce的处理数量256m

    > hive.exec.reducers.max=1009;#reduce的数量1009

    > map.reducer.

9. JVM重用
 + mapreduce.job.jvm.numtasks=1;
 + mapred.job.reuse.jvm.num.tasks=1;

10. 数据倾斜
 + 由于key的分布不均匀造成的数据向一个方向偏的现象。
 + 数据倾斜的原因：

       1. 数据本身倾斜
       2. hql语句：
        + join 容易造成;
        + group by也容易造成
 + 解决方案：

  ```
  1. 找到倾斜的原因，找到造成倾斜的的key，
  可以单独将这个key提取出来进行计算，然后通过ubion合并进来
  可以将key拼接随机数，然后将其分散到各个不同的节点执行。
  2. 设置属性：
      1. hive.map.aggr=true;
      2. hive.optimize.skewjoin=false;#建议开启
      3. hive.groupby.skewindata=false;
```
11. job数量
 + 一般是一个查询，子查询、limit等产生一个job。可以通过语句来控制job
  ```
   select * from aa7 a7
     where a7.id in(
     select id
     from aa9
    where id >1
    limit 1
    );
```
12. 添加索引
13. 创建分区
14. 多个mr中的group by放到单个reduce中

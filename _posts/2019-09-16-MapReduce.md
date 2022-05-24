---
title: "MapReduce"
share: false
categories:
  - 后端技术研究
tags:
  - 大数据
  - 分布式
  - 批处理
---

## 为什么需要 MapReduce
1. 海量数据受硬件资源限制，无法单机处理

2. 将单机程序扩展到分布式集群运行，将极大地增加程序的复杂度和开发难度

3. 引入 MapReduce 框架， 可将大部分开发工作聚焦于业务逻辑的开发，把分布式计算的复杂性交给框架处理

## MapReduce 特点
1. 易于编程。简单地实现接口即可完成一个分布式程序，而整个过程与串行程序的编写完全一致

2. 良好的扩展性。当计算资源不足时，可直接通过增加机器的方式来扩展其计算能力

3. 高容错性。MapReduce 设计的初衷就是使程序能够部署在廉价的 PC 机器上，这就要求它具有很高的容错性。例如其中一台机器挂了，它可以把上面的计算任务转移到另外一个节点上面上运行，不至于这个任务运行失败，而且这个过程不需要人工参与，而完全是由 Hadoop 内部完成的

4. 适合 PB 级以上海量数据的离线处理

5. 不适用实时计算。MapReduce 无法像 Mysql 一样，在毫秒或者秒级内返回结果

6. 不适用流式计算。流式计算的输入数据时动态的，而 MapReduce 的输入数据集是静态的，不能动态变化。这是因为 MapReduce 自身的设计特点决定了数据源必须是静态的

7. 不适用 DAG（有向图）计算。多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce 并不是不能做，而是使用后，每个MapReduce 作业的输出结果都会写入到磁盘，会造成大量的磁盘IO，导致性能非常的低下

## MapReduce 核心
![title](https://i.loli.net/2019/09/22/uoCyOM5NWgTBfZF.jpg)
* MapReduce将复杂的、运行于大规模集群上的并行计算过程高度地抽象到了两个函数：Map和Reduce

* 编程容易，不需要掌握分布式并行编程细节，也可以很容易把自己的程序运行在分布式系统上，完成海量数据的计算

* MapReduce采用“分而治之”策略，一个存储在分布式文件系统中的大规模数据集，会被切分成许多独立的分片（split），这些分片可以被多个Map任务并行处理

* MapReduce设计的一个理念就是“计算向数据靠拢”，而不是“数据向计算靠拢”，因为，移动数据需要大量的网络传输开销
* MapReduce框架采用了Master/Slave架构，包括一个Master和若干个Slave。Master上运行JobTracker，Slave上运行TaskTracker

函数 | 输入 |  输出 |  说明  
-|-|-|-
Map | <k1, v1> | List(<k2, v2>) | 1. 将小数据集进一步解析成一批<key,value>对，输入Map函数中进行处理<br>2. 每一个输入的<k1,v1>会输出一批<k2,v2>。<k2,v2>是计算的中间结果|
Reduce | <k2, List(v2)> | <k3, v3> | 输入的中间结果<k2,List(v2)>中的List(v2)表示是一批属于同一个k2的value

## MapReduce JAVA实战
实现一个统计每行特定首字母个数的 MapReduce 模型
1. 创建一个 maven 项目在 pom.xml dependencies中引入相关依赖

```xml
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-core</artifactId>
            <version>1.2.1</version>
        </dependency>
        
```

2. 编写对应的 Map 和 Reduce

```java
public class MapReduce {
    public static class HotSearchMap extends Mapper<Object, Text, Text, IntWritable> {
        // Mapper<Object, Text, Text, IntWritable> 即 Mapper<k1, v1, k2, v2>

        @Override
        protected map(Object key, Text value, Context context) throws IOException, InterruptedException {
            String currentLine = value.toString();
            if (currentLine.charAt(0) == 'A') {
                context.write(new Text("A"), new IntWritable(1));
            } else if (currentLine.charAt(0) == 'B') {
                context.write(new Text("B"), new IntWritable(1));
            } else {
                context.write(new Text("None"), new IntWritable(1));
            }
        }
    }

    public static class HotSearchReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
        // Reducer<Text, IntWritable, Text, IntWritable> 即 Mapper<k2, v2, k3, v3>

        @Override
        protected void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>) {
            int count = 0;
            for(IntWritable intWritable: values) {
                count += intWritable.get();
            }
            context.write(key, new IntWritable(count));
        }
    }

    public static void main(String[] args) throws Exception {
        Job job = Job.getInstance();

        job.setJarByClass(MapReduce.class);
        job.setMapperClass(HotSearchMap.class);
        job.setReducerClass(HotSearchReduce.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(""));
        FileOutputFormat.setOutputPath(job, new Path(""));
        job.waitForCompletion(true);
    }
}

```

3. 在 FileInputFormat, FileOutputFormat 里填写输入文件与输出文件夹的路径
运行后获得如下4个文件
![title](https://i.loli.net/2019/09/21/C1U38u4zRqgWMTE.png)
    * _SUCCESS 标识程序成功执行
    * part-r-00000 为实际输出文件
    * ._SUCCESS.crc 与 .part-r-00000.crc 为校验文件

* 注意事项：windows引入相关依赖后运行可能出现如下错误
```
java.io.IOException: Failed to set permissions of path: \tmp\hadoop-Administrator\mapred\staging\Administrator-519341271\.staging to 0700
```
* 解决方法：
修改/hadoop-1.0.2/src/core/org/apache/hadoop/fs/FileUtil.java里面的checkReturnValue，注释掉,重新编译打包hadoop-core-1.0.2.jar 或[直接下载替换](https://download.csdn.net/download/yan456jie/9484165)

```java
    private static void checkReturnValue(boolean rv, File p, FsPermission permission) 
        throws IOException {  
        /** 
        if (!rv) { 
            throw new IOException("Failed to set permissions of path: " + p + " to " + String.format("%04o", permission.toShort())); 
        } 
        **/  
    }

```

## 参考文献
[1] http://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf

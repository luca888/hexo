### 从0到1学会Apache Flink-运行Flink程序
---

######运行Flink自带的example:
>cd flink-1.7.1

> ./bin/flink run examples/streaming/WordCount.jar

![](https://i.imgur.com/7hbEWTD.png)

查看运行结果:
> cat log/flink-*-taskexecutor-*.out

![](https://i.imgur.com/k0vRuio.png)

<!--more-->

######编写SocketWindowWordCount程序:
---
在maven项目中创建SocketWindowWordCount.java:

    package com.flink.stream;

	import org.apache.flink.api.common.functions.FlatMapFunction;
	import org.apache.flink.api.common.functions.ReduceFunction;
	import org.apache.flink.streaming.api.datastream.DataStream;
	import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
	import org.apache.flink.streaming.api.windowing.time.Time;
	import org.apache.flink.util.Collector;

	public class SocketWindowWordCount {

    public static void main(String[] args) throws Exception {

        // get the execution environment
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // get input data by connecting to the socket
        String host = "172.16.101.182";
        int port = 9000;
        DataStream<String> text = env.socketTextStream(host, port);

        // parse the data, group it, window it, and aggregate the counts
        DataStream<WordWithCount> windowCounts = text
                .flatMap(new FlatMapFunction<String, WordWithCount>() {
                    @Override
                    public void flatMap(String value, Collector<WordWithCount> out) {
                        for (String word : value.split("\\s")) {
                            out.collect(new WordWithCount(word, 1L));
                        }
                    }
                })
                .keyBy("word")
                .timeWindow(Time.seconds(5), Time.seconds(1))
                .reduce(new ReduceFunction<WordWithCount>() {
                    @Override
                    public WordWithCount reduce(WordWithCount a, WordWithCount b) {
                        return new WordWithCount(a.word, a.count + b.count);
                    }
                });

        // print the results with a single thread, rather than in parallel
        windowCounts.print().setParallelism(1);

        env.execute("Socket Window WordCount");
    }

    // Data type for words with count
    public static class WordWithCount {

        public String word;
        public long count;

        public WordWithCount() {}

        public WordWithCount(String word, long count) {
            this.word = word;
            this.count = count;
        }

        @Override
        public String toString() {
            return word + " : " + count;
        }
    }
	}

项目结构:

![](https://i.imgur.com/YFOq21Q.png)

######运行示例:
---
现在，我们将运行此Flink应用程序。它将从套接字读取文本，并且每5秒打印一次前5秒内每个不同单词的出现次数，即处理时间的翻滚窗口，只要文字漂浮在其中。

* 首先，我们使用netcat来启动本地服务器

>nc -l 9000

* 提交Flink

项目打包后把jar包上传到```flink-1.7.1/examples/streaming/``` 目录下:

![](https://i.imgur.com/ZXL9zOL.png)

另开一个窗口,提交任务:
>cd flink-1.7.1

>./bin/flink run -c com.flink.stream.SocketWindowWordCount examples/streaming/flink-1.0-SNAPSHOT.jar 

-c:指定运行的任务

在nc窗口输入:

![](https://i.imgur.com/fP0UAKq.png)

查看运行结果:

>cat log/flink-*-taskexecutor-*.out

![](https://i.imgur.com/u1duXIe.png)

######用flink-ui的方式提交任务:
---
* 首先，我们使用netcat来启动本地服务器

> nc -l 9000

访问localhost:8081:

![](https://i.imgur.com/u65Sbb7.png)

选择 submit new job > add new 添加项目jar:

![](https://i.imgur.com/9HkTLH9.png)

upload > 选中项目  >填写 entry class 

![](https://i.imgur.com/VkUaGWh.png)

submit :

![](https://i.imgur.com/ptFCnvq.png)





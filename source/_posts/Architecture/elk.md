title: 使用ELK管理日志
date: 2016-11-04  
tags:
    - original
categories:
    - Architecture
---

# Logstash

## 简介
* 官网： https://www.elastic.co/products/logstash

<!-- more -->

## 检查java环境

* 需要java8以上的环境支撑

``` sh
$ java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```

## 下载安装

``` sh
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-5.0.0.tar.gz
$ tar zxvf logstash-5.0.0.tar.gz
$ cd logstash-5.0.0/
$ ./bin/logstash -e 'input { stdin { } } output { stdout {} }'
Sending Logstash logs to /home/nlp/logstash-5.0.0/logs which is now configured via log4j2.properties.
The stdin plugin is now waiting for input:
[2016-11-03T16:05:11,070][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>500}
[2016-11-03T16:05:11,091][INFO ][logstash.pipeline        ] Pipeline main started
[2016-11-03T16:05:11,133][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
hello world
2016-11-03T08:05:47.176Z iZ25ueoepxdZ hello world
```

## 自定义配置文件
* 将日志文件输出到elasticsearch

``` sh
$ vim test.conf
input {
    file {
        path => ["/alidata/logs/web/web-info.log"]
        start_position => "beginning"
    }
}

output {
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
    }
}
```

# Elasticsearch

## 简介
* 官网：https://www.elastic.co/products/elasticsearch

## 下载安装

``` sh
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.0.tar.gz
$ tar zxvf elasticsearch-5.0.0.tar.gz
$ cd elasticsearch-5.0.0/
$ ./bin/elasticsearch
$ curl http://localhost:9200
{
  "name" : "R9IzZP9",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "TGY12_FFSQCfp42g5NZ1VQ",
  "version" : {
    "number" : "5.0.0",
    "build_hash" : "253032b",
    "build_date" : "2016-10-26T04:37:51.531Z",
    "build_snapshot" : false,
    "lucene_version" : "6.2.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 后台运行

``` sh
$ ./bin/elasticsearch -d -p es.pid  // 进程id写到es.pid文件中
```

# Kibana

## 简介
* 官网：https://www.elastic.co/products/kibana

## 下载安装

``` sh
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-5.0.0-linux-x86_64.tar.gz
$ tar xvf kibana-5.0.0-linux-x86_64.tar
$ cd kibana-5.0.0-linux-x86_64/
$ vim config/kibana.yml
server.host: "*.205.*.30" // 外网ip地址，不然只能本机才能访问
elasticsearch.url: "http://localhost:9200"  // 集成elasticsearch
```

* 用浏览器打开查看： http://*.205.*.30:5601

## 后台运行

``` sh
$ nohup ./bin/kibana > nohup.log 2>&1 &
```


<br>
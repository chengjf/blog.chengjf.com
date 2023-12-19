+++
title = '记一次测试环境修改ES的mapping的问题'
date = 2019-06-22T13:40:36+08:00
draft = false
comment = true
tags = ["ES", "ElasticSearch"]
categories = "编程技术"
+++

本周开始做搜索功能优化，这边的搜索功能主要用的是ES。代码写好提测给测试同学的时候，测试同学说搜不出东西，我排查了下，是ES的索引的mapping同线上不一致，线上的mapping增加了三个分词插件，分别是ik，NGram，pinyin，搜索的时候对这些插件扩展字段进行的搜索。

现在就是要把测试环境的索引mapping修改成同线上一致的，我采用的是下面的方案：

1. 新建一个原索引（index）的备份索引（index_bak）,这个备份索引采用线上一致的settings和mappings，可以直接将线上的settings和mappings复制到json文件里。

```shell
curl -XPUT 'http://127.0.0.1:9200/index_bak' -d '@create_index_bak.json'
```

2. 将数据从原索引reindex到备份索引。

```shell
curl -XPOST 'http://127.0.0.1:9200/_reindex' -d'
 {
   "source": {
     "index": "index"
   },
   "dest": {
     "index": "index_bak"
   }
 }
 '  
 ```

3. 删除原索引

```shell
curl -XDELETE 'http://127.0.0.1:9200/index'
```

4. 同新建备份索引一样，重新新建一个索引，同原来的索引同名
```shell
curl -XPUT 'http://127.0.0.1:9200/index' -d '@create_index_bak.json'
```
5. 将数据从备份索引reindex到新建的索引
```shell
curl -XPOST 'http://127.0.0.1:9200/_reindex' -d'
 {
   "source": {
     "index": "index_bak"
   },
   "dest": {
     "index": "index"
   }
 }
 '
 ```   
6. 删除备份索引

```shell
curl -XDELETE 'http://127.0.0.1:9200/index_bak'
```
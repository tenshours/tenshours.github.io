---
layout: post
title: "ElasticSearch基础操作"
date: 2019-4-2
tags: elastic
comments: true
---

#### 查询

1. 如果不想展示source中的内容，设置参数 _source=false
2. 如果展示/不展示指定的field，_source_include=id,value   _source_exclude=id
3. 如果mapping中，property设置了store属性为false，那么search的时候结果不会有该属性
4. 用url查询数据的参数 https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html



#### 删除

1. 根据查询条件删除数据，_delete_by_query

#### 更新

~~~json
POST gavin/user/3/_update
{
  "script": {
    "source": "ctx._source.age += params.value", 
    "lang": "painless",
    "params": {"value": 10}
  }
}
~~~

可以使用script来更新数据
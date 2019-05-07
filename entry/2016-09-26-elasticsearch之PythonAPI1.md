---
layout: post
title: elasticsearch Python API 笔记(一)
category: elasticsearch
tags: [elasticsearch]
---
- Update API
    - 添加新的field,或修改已有的field
        - RESTFul API：

        ```
        curl -XPOST 'localhost:9200/index/type/id/_update -d '{
            "script": "ctx._source.field_name = \"new_field_value\""
        }'
        ```
        - Python API：

        ```
        from elasticsearch import Elasticsearch
        es = Elasticsearch()
        # 以上两行代码为公用代码，下面的Python代码中省略
        
        body = { "doc": { "ctx._source.new_field_name": "new_filed_value"}}
        es.update(index=myIndex, doc_type=myType, id=myId, body=body)
        ```

    - 移除已有的filed
        - RESTFul API:

        ```
        curl -XPOST 'locahost:9200/index/type/id/_update -d '{
            "script": "ctx._source.remove(\"filed_name\")"
        }
        ```
        - Python API:
        
        ```
        body = { "script": { "ctx._source.remove(\"filed_name)" }}
        es.update(index=myIndex, doc_type=youType, id=yourId, body=body)
        ```

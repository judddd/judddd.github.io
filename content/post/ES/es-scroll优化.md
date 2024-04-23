---
title: es-scroll优化
link: es-scroll优化
categories:
- ES
date: 2024-04-16T18:20:09+08:00
date_modified: 2024-04-16T18:20:09+08:00
---

## 官网
https://elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results

## scroll上下文调整
scroll=1m
默认1min

## scorll参数监控
```
# 集群
search.max_open_scroll_context 默认为500
search.max_keep_alive 默认为1d
GET /_nodes/stats/indices/search

# 索引
GET _cat/indices?h=index,health,status,store.size,cds,fm,sm,sc,pri,pri.store.size,rep,docs.deleted,indexing.index_failed,search.query_current,segments.index_writer_memory,searchOpenContexts
```

## 清理scroll
```
DELETE /_search/scroll
{
  "scroll_id" : [
    "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==",
    "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAABFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAAAxZrUllkUVlCa1NqNmRMaUhiQlZkMWFBAAAAAAAAAAIWa1JZZFFZQmtTajZkTGlIYkJWZDFhQQAAAAAAAAAFFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAABBZrUllkUVlCa1NqNmRMaUhiQlZkMWFB"
  ]
}

DELETE /_search/scroll/_all
```

## slice切片
slice(doc) = floorMod(hashCode(doc._id), max))
```
GET /my-index-000001/_search?scroll=1m
{
  "slice": {
    "id": 0,                      
    "max": 2                      
  },
  "query": {
    "match": {
      "message": "foo"
    }
  }
}
GET /my-index-000001/_search?scroll=1m
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "query": {
    "match": {
      "message": "foo"
    }
  }
}

```
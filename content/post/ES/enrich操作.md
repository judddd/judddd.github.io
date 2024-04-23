---
title: enrich操作
link: enrich操作
categories:
- ES
date: 2024-04-16 11:51:41 +0800
date_modified: 2024-04-16 11:51:41 +0800
---

# enrich操作

## 定义policy
```
PUT _enrich/policy/enrich-city
{
  "match": {
    "indices": "city-index",
    "match_field": "city",
    "enrich_fields": [
      "district"
    ]
  }
}

PUT _enrich/policy/enrich-district
{
  "match": {
    "indices": "district-index",
    "match_field": "district",
    "enrich_fields": ["place"]
  }
}
```

## 执行policy
```
PUT _enrich/policy/enrich-district/_execute
PUT _enrich/policy/enrich-city/_execute
```

## ingest pipeline添加
```
PUT _ingest/pipeline/enrich-city
{
  "description": "Enrich demo",
  "processors": [
    {
      "enrich": {
        "description": "demo",
        "policy_name": "enrich-city",
        "field": "city",
        "target_field": "enrich-city",
        "max_matches": "1",
        "ignore_failure": true
      }
    },
      {
      "enrich": {
        "description": "demo1",
        "policy_name": "enrich-district",
        "field": "enrich-city.district",
        "target_field": "enrich-place",
        "max_matches": "5",
        "ignore_failure": true
      }
    }
  ]
}
```

## 插入文档
```
POST country-index/_doc?pipeline=enrich-city
{
  "city":"cq"
}
```

## 转换后
<img src="/images/2024/04/16/enrich.png" alt="" title="enrich.png" border="0" width="2414" height="910" />
---
layout: post
title: 009 Apache Druid - Transform Data
categories: Druid
---

# Mục đích bài viết
Trong bài viết này chúng ta sẽ cùng tìm hiểu về cách biến đổi dữ liệu khi nhập dữ liệu.

# Dữ liệu mẫu
Sử dụng dữ liệu mẫu trong file `quickstart/tutorial/transform-data.json`
```json
{"timestamp":"2018-01-01T07:01:35Z","animal":"octopus",  "location":1, "number":100}
{"timestamp":"2018-01-01T05:01:35Z","animal":"mongoose", "location":2,"number":200}
{"timestamp":"2018-01-01T06:01:35Z","animal":"snake", "location":3, "number":300}
{"timestamp":"2018-01-01T01:01:35Z","animal":"lion", "location":4, "number":300}
```
# Load data
Load data bằng ingestion spec `quickstart/tutorial/transform-index.json`
```json
{
  "type" : "index_parallel",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "transform-tutorial",
      "timestampSpec": {
        "column": "timestamp",
        "format": "iso"
      },
      "dimensionsSpec" : {
        "dimensions" : [
          "animal",
          { "name": "location", "type": "long" }
        ]
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "number", "fieldName" : "number" },
        { "type" : "longSum", "name" : "triple-number", "fieldName" : "triple-number" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "week",
        "queryGranularity" : "minute",
        "intervals" : ["2018-01-01/2018-01-03"],
        "rollup" : true
      },
      "transformSpec": {
        "transforms": [
          {
            "type": "expression",
            "name": "animal",
            "expression": "concat('super-', animal)"
          },
          {
            "type": "expression",
            "name": "triple-number",
            "expression": "number * 3"
          }
        ],
        "filter": {
          "type":"or",
          "fields": [
            { "type": "selector", "dimension": "animal", "value": "super-mongoose" },
            { "type": "selector", "dimension": "triple-number", "value": "300" },
            { "type": "selector", "dimension": "location", "value": "3" }
          ]
        }
      }
    },
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/tutorial",
        "filter" : "transform-data.json"
      },
      "inputFormat" : {
        "type" :"json"
      },
      "appendToExisting" : false
    },
    "tuningConfig" : {
      "type" : "index_parallel",
      "maxRowsPerSegment" : 5000000,
      "maxRowsInMemory" : 25000
    }
  }
}
```
Để ý trong đoạn này có thông tin `transformSpec`
```json
{
    "transformSpec": {
        "transforms": [
          {
            "type": "expression", // Sử dụng biểu thức để biến đổi
            "name": "animal", // Tên field sau khi biến đổi, nếu trùng với trường trước khi biến đổi thì sẽ overide
            "expression": "concat('super-', animal)" // Thêm prefix field animal bằng "super-"
          },
          {
            "type": "expression", // Sử dụng biểu thức để biến đổi
            "name": "triple-number", // Tên field sau khi biến đổi, nếu khác với field gốc thì sẽ tạo field mới
            "expression": "number * 3" // Nhân 3 lên
          }
        ],
        "filter": { // Điều kiện lọc sau khi đã biến đổi dữ liệu
          "type":"or",
          "fields": [
            { "type": "selector", "dimension": "animal", "value": "super-mongoose" },
            { "type": "selector", "dimension": "triple-number", "value": "300" },
            { "type": "selector", "dimension": "location", "value": "3" }
          ]
        }
    }
}
```
Submit task
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/transform-index.json http://localhost:8081/druid/indexer/v1/task
```
Dùng dsql query được kết quả như mong muốn: 
```sh
dsql> select * from "transform-tutorial";
┌──────────────────────────┬────────────────┬───────┬──────────┬────────┬───────────────┐
│ __time                   │ animal         │ count │ location │ number │ triple-number │
├──────────────────────────┼────────────────┼───────┼──────────┼────────┼───────────────┤
│ 2018-01-01T05:01:00.000Z │ super-mongoose │     1 │        2 │    200 │           600 │
│ 2018-01-01T06:01:00.000Z │ super-snake    │     1 │        3 │    300 │           900 │
│ 2018-01-01T07:01:00.000Z │ super-octopus  │     1 │        1 │    100 │           300 │
└──────────────────────────┴────────────────┴───────┴──────────┴────────┴───────────────┘
Retrieved 3 rows in 0.03s.
```

Tạm kết: Như vậy qua bài viết này chúng ta đã biết cách biến đổi dữ liệu khi nhập để phù hợp với mục đích thống kê của mình. Happy hacking!!!

# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
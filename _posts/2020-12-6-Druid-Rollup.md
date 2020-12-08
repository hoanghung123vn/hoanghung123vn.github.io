---
layout: post
title: 006 Apache Druid - Roll-up In Druid
categories: Druid
---

# Mục đích bài viết
Trong quá trình Ingestion, roll-up là kỹ thuật giúp bạn tổng hợp trước dữ liệu ngay lúc nhập dữ liệu, từ đó giảm lượng dữ liệu cần lưu trữ.

# Chuẩn bị dữ liệu
Dữ liệu của chúng ta có dạng như sau:
```json
{"ts":"2018-01-01T01:01:35Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":10, "bytes":1000, "cost": 1.4}
{"ts":"2018-01-01T01:01:51Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":20, "bytes":2000, "cost": 3.1}
{"ts":"2018-01-01T01:01:59Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":30, "bytes":3000, "cost": 0.4}
{"ts":"2018-01-01T01:02:14Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":40, "bytes":4000, "cost": 7.9}
{"ts":"2018-01-01T01:02:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":50, "bytes":5000, "cost": 10.2}
{"ts":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
{"ts":"2018-01-01T02:33:14Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":100, "bytes":10000, "cost": 22.4}
{"ts":"2018-01-01T02:33:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":200, "bytes":20000, "cost": 34.5}
{"ts":"2018-01-01T02:35:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":300, "bytes":30000, "cost": 46.3}
```
File dữ liệu này nằm tại vị trí `quickstart/tutorial/rollup-data.json`.

Chúng ta sẽ nhập dữ liệu bằng một ingestion spec như dưới đây, file này đặt tại vị trí `quickstart/tutorial/rollup-index.json`.
```json
{
  "type" : "index_parallel",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "rollup-tutorial",
      "dimensionsSpec" : {
        "dimensions" : [ // Các dimentions là các trường được chọn sẽ lưu trữ trong Druid
          "srcIP",
          "dstIP"
        ]
      },
      "timestampSpec": {
        "column": "timestamp",
        "format": "iso"
      },
      "metricsSpec" : [ // Các mestrics là các trường tổng hợp được lựa chọn
        { "type" : "count", "name" : "count" }, // Đếm số dòng được kết hợp lại sau khi roll-up
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" }, // Tổng số package
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" } // Tổng số bytes
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "week", // Quy định dữ liệu trong khoảng thời gian này sẽ nằm trong cùng một segment
        "queryGranularity" : "minute", // Quy định mức độ chi tiết của quá trình roll-up, tất cả các bản ghi trong cùng 1 phút sẽ được tổng hợp lại, timstamp sẽ được làm tròn theo phút
        "intervals" : ["2018-01-01/2018-01-03"], // Ngoài khoảng thời gian này dữ liệu sẽ được bỏ qua, chỉ áp dụng với batch ingestion
        "rollup" : true // Bật chế độ roll-up
      }
    },
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/tutorial",
        "filter" : "rollup-data.json"
      },
      "inputFormat" : {
        "type" : "json"
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
# Load data
Thực hiện ingestion bằng một HTTP request: 
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/rollup-index.json http://localhost:8081/druid/indexer/v1/task
```
Sử dụng dsql để truy vấn ta được kết quả như sau:
```sh
$ bin/dsql
Welcome to dsql, the command-line client for Druid SQL.
Type "\h" for help.
dsql> select * from "rollup-tutorial";
┌──────────────────────────┬────────┬───────┬─────────┬─────────┬─────────┐
│ __time                   │ bytes  │ count │ dstIP   │ packets │ srcIP   │
├──────────────────────────┼────────┼───────┼─────────┼─────────┼─────────┤
│ 2018-01-01T01:01:00.000Z │  35937 │     3 │ 2.2.2.2 │     286 │ 1.1.1.1 │
│ 2018-01-01T01:02:00.000Z │ 366260 │     2 │ 2.2.2.2 │     415 │ 1.1.1.1 │
│ 2018-01-01T01:03:00.000Z │  10204 │     1 │ 2.2.2.2 │      49 │ 1.1.1.1 │
│ 2018-01-02T21:33:00.000Z │ 100288 │     2 │ 8.8.8.8 │     161 │ 7.7.7.7 │
│ 2018-01-02T21:35:00.000Z │   2818 │     1 │ 8.8.8.8 │      12 │ 7.7.7.7 │
└──────────────────────────┴────────┴───────┴─────────┴─────────┴─────────┘
Retrieved 5 rows in 1.18s.

dsql>
```
Ta thấy kết quả của các record với timestamp `2018-01-01T01:01` đã được tổng hợp lại từ 3 bản ghi thành 1 bản ghi duy nhất.
```sh
┌──────────────────────────┬────────┬───────┬─────────┬─────────┬─────────┐
│ __time                   │ bytes  │ count │ dstIP   │ packets │ srcIP   │
├──────────────────────────┼────────┼───────┼─────────┼─────────┼─────────┤
│ 2018-01-01T01:01:00.000Z │  35937 │     3 │ 2.2.2.2 │     286 │ 1.1.1.1 │
└──────────────────────────┴────────┴───────┴─────────┴─────────┴─────────┘
```

# Kết quả

Tạm kết: Trong bài này chúng ta đã cùng tìm hiểu roll-up trong Druid và cách thức hoạt động của nó.

# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
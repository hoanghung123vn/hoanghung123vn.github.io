---
layout: post
title: 003 Apache Druid - Druid Ingestion
categories: Druid
---
# Mục đích bài viết
Trong bài viết này chúng ta sẽ tìm hiểu cách viết một file spec để ingestion.

# Chuẩn bị dữ liệu
Giả sử dữ liệu của chúng ta có dạng như sau:
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
Lưu vào file ingestion-tutorial-data.json trong thư mục quickstart/ của thư mục gốc Druid

# Viết file spec
Định nghĩa type của ingestion, nó có nhiều giá trị khác nhau tùy theo kiểu ingestion, ví dụ với native batch ingestion thì sẽ có giá trị `index_parallel`, streaming ingestion với kafka thì sẽ có giá tri là `kafka`.

Định nghĩa data schema bên trong spec: Đây là thành phần quan trọng nhất của file spec.

Tạo file `ingestion-tutorial-index.json` trong thư mục `quickstart/` với nội dung: 
```json
{
    "type": "index_parallel",
    "spec": {
        "dataSchema": {}
    }
}
```

Thành phần đầu tiên của `dataSchema` là `dataSource` - tên của datasource, tiếp theo là `timestampSpec` chứa mô tả trường sẽ làm timestamp trong dữ liệu của mình, nên trong dư liệu của Druid nhất định phải có một trường thời gian. Đặt vào file `json` như sau:
```json
{
    "type": "index_parallel",
    "spec": {
        "dataSchema": {
            "dataSource": "ingestion-tutorial",
            "timestampSpec": {
                "format": "iso",
                "column": "ts"
            }
        }
    }
}
```
Kiểu dữ liệu trong Druid bao gồm 4 loại String, Long, Float, Double tương ứng với các kiểu dữ liệu trong SQL <a href="http://druid.apache.org/docs/latest/querying/sql.html#data-types">chi tiết</a>.

`granularitySpec` chứa mô tả về mức độ chi tiết của dữ liệu, trong này có mô tả về `rollup`, một tính năng cho phép chúng ta xử lý trước dữ liệu tại thời điểm nhập.

`demensionSpec` mô tả các trường sẽ được nhập vào Druid. `metricsSpec` mô tả các chỉ số sẽ được tổng hợp khi nhập, mô tả này chỉ xuất hiện khi `rollup` set bằng `true`.

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "rollup" : true
  }
}
```
`Demensions` và `Metrics` mô tả bởi `type` và `name` với giá trị mặc định của `type` là `string`. Ví dụ với `demension` chỉ cần mô tả với "srcIP" cũng đủ để hiểu có `name` là `srcIp`, `type` là `string`. Các trường lựa chọn để làm metrics dựa theo kiểu dữ liệu và ý nghĩa của nó trong từng context nhất định. Trong ví dụ này dữ liệu đang là các gói tin trong luồng mạng, các metrics được lựa chọn đó là số packets, số bytes, chi phí cost, và thêm 1 trường count để đếm số dòng đã được tổng hợp khi tiến hành `rollup`.

Dữ liệu được `rollup` như thế nào sẽ dựa vào mô tả trong `granularitySpec`: 
```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform", // Tất cả các segments (phân đoạn) đều có kích thước khoảng thời gian đồng nhất.
    "segmentGranularity" : "HOUR", // Kích thước khoảng thời gian của mỗi segments.
    "queryGranularity" : "MINUTE", // Chỉ định dữ liệu sẽ được rollup như thế nào, trong trường hợp này các bản ghi giống nhau nhưng chỉ lệch về timestamp trong cùng 1 phút sẽ được hợp nhất lại làm 1 bản ghi sau khi nhập.
    "intervals" : ["2018-01-01/2018-01-02"], // Chỉ có trong batch ingestion, chỉ có dữ liệu trong khoảng thời gian này mới được nhập.
    "rollup" : true
  }
}
```

Định nghĩa đầu vào datasource
```json
    "ioConfig" : {
      "type" : "index_parallel", // Kiểu nhập batch native.
      "inputSource" : {
        "type" : "local", // Kiểu đầu vào từ locla.
        "baseDir" : "quickstart/", // Thư mục chứa các file cần nhập.
        "filter" : "ingestion-tutorial-data.json" // Ký tự đại diện để lọc các file cần nhập, cho phép nhập nhiều file đồng thời.
      },
      "inputFormat" : {
        "type" : "json" // Định dạng file nhập vào
      }
    }
```

Thêm vào các tùy chỉnh
```json
    "tuningConfig" : {
      "type" : "index_parallel",
      "maxRowsPerSegment" : 5000000 // Số hàng tối đa trên mỗi segments.
    }
```

Cuối cùng chúng ta sẽ được 1 file spec như thế này
```json
{
  "type" : "index_parallel",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "ingestion-tutorial",
      "timestampSpec" : {
        "format" : "iso",
        "column" : "ts"
      },
      "dimensionsSpec" : {
        "dimensions": [
          "srcIP",
          { "name" : "srcPort", "type" : "long" },
          { "name" : "dstIP", "type" : "string" },
          { "name" : "dstPort", "type" : "long" },
          { "name" : "protocol", "type" : "string" }
        ]
      },
      "metricsSpec" : [
        { "type" : "count", "name" : "count" },
        { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
        { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
        { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "MINUTE",
        "intervals" : ["2018-01-01/2018-01-02"],
        "rollup" : true
      }
    },
    "ioConfig" : {
      "type" : "index_parallel",
      "inputSource" : {
        "type" : "local",
        "baseDir" : "quickstart/",
        "filter" : "ingestion-tutorial-data.json"
      },
      "inputFormat" : {
        "type" : "json"
      }
    },
    "tuningConfig" : {
      "type" : "index_parallel",
      "maxRowsPerSegment" : 5000000
    }
  }
}
```

Tùy vào từng kiểu nhập, sẽ có thêm các mô tả và giá trị tương ứng khác nhau.
# Submit task ingestion
Submit task bằng cách gọi API
```shell
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/ingestion-tutorial-index.json http://localhost:8081/druid/indexer/v1/task
```

Tạm kết: Trong bài viết này, chúng ta đã biết được cách viết một spec ingestion, các mô tả và ý nghĩa của chúng. Bài viết sau chúng ta sẽ cùng tìm hiểu cách Query dữ liệu trong Druid.

# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
---
layout: post
title: 004 Apache Druid - Writing Spec Ingestion
categories: Druid
---
# Mục đích bài viết
Khái niệm nhập dữ liệu trong Druid được gọi là ingestion. Trong bài viết này chúng ta sẽ cùng tìm hiểu các cách nhập dữ liệu trong Druid.
Druid cung cấp 2 phương thức để nhập dữ liệu đó là batch ingestion và streaming ingestion.

# Batch ingestion
Nhập dữ liệu hàng loạt là cách thức nhập dữ liệu từ file ở local file system (file có thể ở dạng nén).

1.Nhập thông qua web console

Tại màn hình web console, trên thanh tiêu đề chọn Load data, tiếp tục chọn Load disk sau đó click Connect data

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-01.png)

Nhập các giá trị như sau: 

- Base directory - thư mục ở local fs: quickstart/tutorial/

- File filter - ký tự đại diện, cho phép ingestion nhiều file: wikiticker-2015-09-12-sampled.json.gz

Nhấn Apply, sau đó dữ liệu từ file chưa được xử lý sẽ được load vào, phía trên console sẽ xuất hiện trình tự các bước nhập dữ liệu, ví dụ như Connect -> Parse data -> Parse time -> Transform -> Filter -> Configuration schema -> Partition -> Tune -> Publish, các bước này chính là để chúng ta điều chỉnh các tham số của việc nhập file, tất cả các cấu hình này cuối cùng sẽ được ghi vào file spec ingestion ở tab Edit spec. Bạn có thể bỏ qua tất cả các bước trước thay vào đó bằng việc viết một file spec cho việc ingestion (sẽ có một bài viết riêng cho việc viết file spec này)

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-02.png)

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-03.png)

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-04.png)

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-05.png)

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-06.png)

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-07.png)

Đây chính là file spec cuối cùng của chúng ta:

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-08.png)

Sau khi nhấn Submit, 1 task ingestion sẽ được thực thi, sau một khoảng thời gian, trạng thái của task chuyển sang SUCCESS cho thấy dữ liệu đã được nhập xong. Giờ đây bạn có thể Query vào dữ liệu đã được nhập. 

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-batch-data-loader-09.png)

2.Nhập thông qua command line

Druid cung cấp cho chúng ta một số script helper, ví dụ cho việc nhập dữ liệu <code>bin/post-index-task</code>

Tại thư mục gốc của Druid, chạy lệnh POST một ingestion task đến Overlord. Thư mục quickstart/tutorial chứa file spec của chúng ta:

```shell
bin/post-index-task --file quickstart/tutorial/wikipedia-index.json --url http://localhost:8081
```

Kết quả thu được giống như sau: 

```shell
Beginning indexing data for wikipedia
Task started: index_wikipedia_2018-07-27T06:37:44.323Z
Task log:     http://localhost:8081/druid/indexer/v1/task/index_wikipedia_2018-07-27T06:37:44.323Z/log
Task status:  http://localhost:8081/druid/indexer/v1/task/index_wikipedia_2018-07-27T06:37:44.323Z/status
Task index_wikipedia_2018-07-27T06:37:44.323Z still running...
Task index_wikipedia_2018-07-27T06:37:44.323Z still running...
Task finished with status: SUCCESS
Completed indexing data for wikipedia. Now loading indexed data onto the cluster...
wikipedia loading complete! You may now query your data
```

3.Nhập thông qua HTTP request

Druid cung cấp cho chúng ta một bộ API để tương tác với nó, để tạo 1 ingestion task, chạy lệnh sau: 
```shell
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/wikipedia-index.json http://localhost:8081/druid/indexer/v1/task
```
Kết quả thu được
```json
{ "task":"index_wikipedia_2018-06-09T21:30:32.802Z" }
```

# Streaming ingrestion
Chúng ta sẽ thử nhập dữ liêu online với 1 hệ thống Message Queue phổ biến hiện nay đó là Kafka.

Đầu tiên chúng ta cần chạy dịch vụ Kafka, download Kafka:
```shell
curl -O https://archive.apache.org/dist/kafka/2.6.0/kafka_2.13-2.6.0.tgz
tar -xzf kafka_2.13-2.6.0.tgz
cd kafka_2.13-2.6.0
```

Chạy Zookeeper: 
```shell
bin/zookeeper-server-start.sh config/zookeeper.properties
```

Start Kafka: 
```shell
bin/kafka-server-start.sh config/server.properties
```
Tạo một topic wikipedia: 
```shell
bin/kafka-topics.sh --create --topic wikipedia --bootstrap-server localhost:9092
```

Load data vào Kafka: 
```shell
cd quickstart/tutorial
gunzip -c wikiticker-2015-09-12-sampled.json.gz > wikiticker-2015-09-12-sampled.json
```

Submit 1 ingestion task thông qua HTTP vơi file spec wikipedia-kafka-supervisor json như sau: 
```json
{
  "type": "kafka",
  "spec" : {
    "dataSchema": {
      "dataSource": "wikipedia",
      "timestampSpec": {
        "column": "time",
        "format": "auto"
      },
      "dimensionsSpec": {
        "dimensions": [
          "channel",
          "cityName",
          "comment",
          "countryIsoCode",
          "countryName",
          "isAnonymous",
          "isMinor",
          "isNew",
          "isRobot",
          "isUnpatrolled",
          "metroCode",
          "namespace",
          "page",
          "regionIsoCode",
          "regionName",
          "user",
          { "name": "added", "type": "long" },
          { "name": "deleted", "type": "long" },
          { "name": "delta", "type": "long" }
        ]
      },
      "metricsSpec" : [],
      "granularitySpec": {
        "type": "uniform",
        "segmentGranularity": "DAY",
        "queryGranularity": "NONE",
        "rollup": false
      }
    },
    "tuningConfig": {
      "type": "kafka",
      "reportParseExceptions": false
    },
    "ioConfig": {
      "topic": "wikipedia",
      "inputFormat": {
        "type": "json"
      },
      "replicas": 2,
      "taskDuration": "PT10M",
      "completionTimeout": "PT20M",
      "consumerProperties": {
        "bootstrap.servers": "localhost:9092"
      }
    }
  }
}
```

Chạy với lệnh sau:

```shell
curl -XPOST -H'Content-Type: application/json' -d @quickstart/tutorial/wikipedia-kafka-supervisor.json http://localhost:8081/druid/indexer/v1/supervisor
```

Nếu như thành công, kết quả của bạn sẽ trông như sau:
```json
{ "id":"wikipedia" }
```

Tạm kết: Như vậy chúng ta đã biết được các cách nhập dữ liệu khác nhau trong Druid, bài sau chúng ta sẽ cùng tìm hiểu cách viết một Spec Ingestion file để nhập dữ liệu trực tiếp. Happy hacking!!!
# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
---
layout: post
title: 007 Apache Druid - Compacting Segments
categories: Druid
---

# Mục đích bài viết
Trong bài viết này chúng ta sẽ cùng tìm hiểu về cách giảm số lượng `segments` trong dữ liệu của Druid. Việc giảm số segment có thể giúp giảm bộ nhớ lưu trữ và quá trình xử lý.

# Load data
Thực hiện ingestion bằng một HTTP request: 
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/compaction-init-index.json http://localhost:8081/druid/indexer/v1/task
```
Sau khi task thực hiện thành công, ta thấy hiện tại đang có 51 segments: 
![alt](https://druid.apache.org/docs/latest/assets/tutorial-compaction-02.png)

Bởi vì `tuningConfig` của chúng ta đang để `maxRowsPerSegment` là `1000`.

Query lấy số record của bảng: 
```sh
dsql> select count(*) from "compaction-tutorial";
┌────────┐
│ EXPR$0 │
├────────┤
│  39244 │
└────────┘
Retrieved 1 row in 1.38s.
```

Ta thấy số hàng hiện giờ là 39244

# Giảm số lượng segments
Giảm số lượng segments bằng cách submit 1 task có task spec `quickstart/tutorial/compaction-keep-granularity.json` có nội dung:

```json
{
  "type": "compact",
  "dataSource": "compaction-tutorial",
  "interval": "2015-09-12/2015-09-13",
  "tuningConfig" : {
    "type" : "index_parallel",
    "maxRowsPerSegment" : 5000000,
    "maxRowsInMemory" : 25000
  }
}
```
Thực hiện `submit`

```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/compaction-keep-granularity.json http://localhost:8081/druid/indexer/v1/task
```
51 segment ban đầu sẽ được đánh dấu là `unused` và sau đó sẽ được xóa. Theo mặc định, Druid Coordinator sẽ không đánh dấu `unsued` cho đến khi quy trình Coordinator chạy được ít nhất 15p cho nên sau khi submit xong bạn sẽ thấy có tất cả 75 segments bao gồm cả segmens cũ và mới. Sau 15p, số segments giảm xuống còn 24.

# Tiếp tục giảm số lượng segments
Sau khi tùy chỉnh số lượng record tối đa trên mỗi `segment` để giảm số lượng `segments`, ta có thể chỉnh thêm giá trị `segmentGranularity` để giảm tiếp số lượng `segment`.

Task spec `quickstart/tutorial/compaction-day-granularity.json`: 
```json
{
  "type": "compact",
  "dataSource": "compaction-tutorial",
  "interval": "2015-09-12/2015-09-13",
  "segmentGranularity": "DAY",
  "tuningConfig" : {
    "type" : "index_parallel",
    "maxRowsPerSegment" : 5000000,
    "maxRowsInMemory" : 25000,
    "forceExtendableShardSpecs" : true
  }
}
```
Submit bằng HTTP request:

```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/compaction-day-granularity.json http://localhost:8081/druid/indexer/v1/task
```

Sau khi `submit`, số `segments` hiện giờ là 25, sau 15p số `segments` hiện tại còn lại 1.

Tạm kết: Như vậy qua bài viết này chúng ta đã biết giảm số lượng `segment` với Druid để tối ưu lưu trữ và chi phí xử lý. Happy hacking!!!

# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
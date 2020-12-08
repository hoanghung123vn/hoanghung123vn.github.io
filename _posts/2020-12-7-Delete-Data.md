---
layout: post
title: 008 Apache Druid - Delete Data
categories: Druid
---

# Mục đích bài viết

Trong bài viết này chúng ta sẽ cùng tìm hiểu về cách xóa dữ liệu hoàn toàn trong Druid. Dể xóa được dữ liệu hoàn toàn trong Druid, bạn cần phải thực hiện 2 bước, một là đánh dấu các segments cần xóa thành `unused`, hai là chạy một kill task để xóa các `unused` segments.

# Load data

Thực hiện ingestion bằng một HTTP request: 
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/deletion-index.json http://localhost:8081/druid/indexer/v1/task
```

# Đánh dấu `unused` segments theo khoảng thời gian
Đánh dấu tất cả các segments trong khoảng `2015-09-12T18:00:00.000Z/2015-09-12T20:00:00.000Z` thành `unsused`": 
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d '{ "interval" : "2015-09-12T18:00:00.000Z/2015-09-12T20:00:00.000Z" }' http://localhost:8081/druid/coordinator/v1/datasources/deletion-tutorial/markUnused
```
Sau khi chạy lệnh này, segment cho giờ 18 và 19 sẽ trở thành `unsued` nhưng chúng ta sẽ vẫn thấy chúng trong deep storage.
![alt](https://druid.apache.org/docs/latest/assets/tutorial-deletion-02.png)

# Đánh dấu `unused` segments theo segment ids

Sử dụng file segment ids `quickstart/tutorial/deletion-disable-segments.json` bao gồm các segment ids muốn đánh dấu: 
```json
{
  "segmentIds":
  [
    "deletion-tutorial_2015-09-12T13:00:00.000Z_2015-09-12T14:00:00.000Z_2019-05-01T17:38:46.961Z",
    "deletion-tutorial_2015-09-12T14:00:00.000Z_2015-09-12T15:00:00.000Z_2019-05-01T17:38:46.961Z"
  ]
}
```

Đánh dấu bằng segment ids

```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/deletion-disable-segments.json http://localhost:8081/druid/coordinator/v1/datasources/deletion-tutorial/markUnused
```

Sau khi chạy segments có id 13 và 14 đã được đánh dấu `unsued`.

# Run kill task
Chạy kill task với spec `quickstart/tutorial/deletion-kill.json`
```json
{
  "type": "kill",
  "dataSource": "deletion-tutorial",
  "interval" : "2015-09-12/2015-09-13"
}```

Với HTTP request:
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/deletion-kill.json http://localhost:8081/druid/indexer/v1/task
```

Sau khi chạy chúng ta thấy các segment `unused` trong khoảng thời gian `2015-09-12/2015-09-13` sẽ bị xóa cả trong deep storage.

Tạm kết: Như vậy qua bài viết này chúng ta đã biết các cách xóa dữ liệu hoàn toàn trong Druid. Happy hacking!!!

# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
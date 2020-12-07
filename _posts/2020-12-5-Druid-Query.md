---
layout: post
title: 005 Apache Druid - Query In Druid
categories: Druid
---

# Mục đích bài viết
Trong bài viết này chúng ta sẽ cùng tìm hiểu cách truy vấn dữ liệu trong Druid. Druid hỗ trợ 2 phương thức truy vấn là Druid SQL và Native query. 

# Druid SQL
Như tên gọi của nó, phương thức này cho phép chúng ta dùng SQL là một ngôn ngữ quen thuộc để truy vấn dữ liệu. Có nhiều cách để bạn thực hiện được truy vấn trong Druid. Giả sử bạn đã cài đặt thành công Druid.

1.Sử dụng Druid console: Mở trình duyệt của bạn ra [http://localhost:8888](http://localhost:8888). Chọn `Query` trên thanh tiêu đề bạn sẽ nhìn thấy Query view như sau: 
![alt text](https://druid.apache.org/docs/latest/assets/tutorial-query-01.png)

Lưu ý mỗi datasource bạn nhập vào được coi là một bảng. Bây giờ bạn có thể nhập bất cứ một câu truy vấn hợp lệ trong SQL. Click vào `Run` hoặc `Crtl Enter` để chạy đoạn sql. 
![alt text](https://druid.apache.org/docs/latest/assets/tutorial-query-04.png)

Trên giao diện cung cấp một số tính năng như show các cột, lọc, groud by, aggregate các bạn tự tìm hiểu thêm.

Thực chất tất cả các câu lệnh SQL đều sẽ được chuyển về dạng `json` [native query](http://druid.apache.org/docs/latest/querying/querying.html) trước khi câu truy vấn được thực thi. Bạn có thể xem được native query bằng cách click vào `...` bên cạnh nút `Run`, chọn `Explain SQL Query`.
![alt text](https://druid.apache.org/docs/latest/assets/tutorial-query-06.png)

Có thể bạn quen thuộc với SQL hơn nhưng trong một số trường hợp native query có thể cải thiện hiệu năng truy vấn của bạn.

Để có thể xem giải thích của câu SQL bằng SQL bạn có thể chạy đoạn sql sau:
```sql
EXPLAIN PLAN FOR
SELECT
 "page",
 "countryName",
 COUNT(*) AS "Edits"
FROM "wikipedia"
WHERE "countryName" IS NOT NULL
GROUP BY 1, 2
ORDER BY "Edits" DESC
```

Ví dụ truy vấn bằng sql:
- Tìm số dòng bị xóa theo giờ trong dữ liệu wikipedia:
```sql
SELECT FLOOR(__time to HOUR) AS HourTime, SUM(deleted) AS LinesDeleted
FROM wikipedia WHERE "__time" BETWEEN TIMESTAMP '2015-09-12 00:00:00' AND TIMESTAMP '2015-09-13 00:00:00'
GROUP BY 1
```
Kết quả:

![alt text](https://druid.apache.org/docs/latest/assets/tutorial-query-07.png)


2.Sử dụng dsql

Chạy lệnh `bin/dsql` chúng ta sẽ được một giao diện trông giống như sql command line, bạn có thể viết câu truy vấn sql ở đây.
```sh
Welcome to dsql, the command-line client for Druid SQL.
Type "\h" for help.
dsql>
```

3.Thông qua HTTP request

Sử dụng file truy vấn `wikipedia-top-pages-sql.json`:
```json
{
  "query" : "SELECT page, COUNT(*) AS Edits FROM wikipedia WHERE \"__time\" BETWEEN TIMESTAMP '2015-09-12 00:00:00' AND TIMESTAMP '2015-09-13 00:00:00' GROUP BY page ORDER BY Edits DESC LIMIT 10"
}
```
Chạy lệnh: 
```sh
curl -X 'POST' -H 'Content-Type:application/json' -d @quickstart/tutorial/wikipedia-top-pages-sql.json http://localhost:8888/druid/v2/sql
```
Được kết quả: 
```json
[
  {
    "page": "Wikipedia:Vandalismusmeldung",
    "Edits": 33
  },
  {
    "page": "User:Cyde/List of candidates for speedy deletion/Subpage",
    "Edits": 28
  }
  // đã cắt bớt kết quả
]
```
# Native query 
Sử dụng lệnh sau để gửi một native query: 
```sh
curl -X POST '<queryable_host>:<port>/druid/v2/?pretty' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @<query_json_file>
```
hoặc bạn có thể viết trực tiếp native query dưới dạng `json` vào SQL console:

![alt text](https://druid.apache.org/docs/latest/assets/native-queries-01.png)

Có nhiều loại truy vấn [xem tại đây](http://druid.apache.org/docs/latest/querying/querying.html#available-queries). Ví dụ với loại truy vấn [timeseries](http://druid.apache.org/docs/latest/querying/timeseriesquery.html) có file `query_json_file` như sau:
```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "sample_dimension1", "value": "sample_value1" },
      { "type": "or",
        "fields": [
          { "type": "selector", "dimension": "sample_dimension2", "value": "sample_value2" },
          { "type": "selector", "dimension": "sample_dimension3", "value": "sample_value3" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" },
    { "type": "doubleSum", "name": "sample_name2", "fieldName": "sample_fieldName2" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "sample_divide",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "name": "postAgg__sample_name1", "fieldName": "sample_name1" },
        { "type": "fieldAccess", "name": "postAgg__sample_name2", "fieldName": "sample_name2" }
      ]
    }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000" ]
}
```
Kết quả đầu ra: 
```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> }
  },
  {
    "timestamp": "2012-01-02T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> }
  }
]
```

Có thể hủy query thông qua HTTP request nếu trong file query chúng ta chỉ rõ id của nó:
```sh
# DELETE /druid/v2/{queryId}
curl -X DELETE "http://host:port/druid/v2/abc123"
```

Tạm kết: Như vậy qua bài viết này chúng ta đã biết cách truy vấn dữ liệu với druid. Các bài viết sau sẽ tập trung vào các kỹ thuật tối ưu lưu trữ và truy vấn với Druid. Happy hacking!!!
<!-- ![alt text](http://bizweb.dktcdn.net/thumb/grande/dev/100/005/238/products/ls.png?v=1607057688903) -->
# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
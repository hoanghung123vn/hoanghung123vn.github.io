---
layout: post
title: 001 Apache Druid - Introduction To Druid
categories: Druid
---

# Mục đích bài viết
Mình đang tìm hiểu để làm một hệ thống thống kê truy cập trên một hệ thống website. Sản phẩm công ty mình đang làm là một nền tảng thương mại điện tử, tự động tạo ra trang web cho người dùng chỉ với vài bước đơn giản. Số lượng website rất lớn cho nên nhu cầu cũng là rất lớn. Lõi của một hệ thống báo cáo thống kê đương nhiên phải là một hệ quản trị cơ sở dữ liệu tốt rồi. Apache Druid là một CSDL được thiết kế với mục đích tối ưu lưu trữ, truy vấn nhanh, phân tán, xử lý song song, dễ dàng scale. Bài viết này sẽ giới thiệu về Druid, các key features của nó, khi nào nên sử dụng.

# Druid là gì ?
Trên trang chủ tài liệu có ghi một câu "Apache Druid is a high performance real-time analytics database" - một CSDL phân tích thời gian thực với hiệu năng cao. Bản chất nó vẫn là một hệ quản trị CSDL, vẫn có tạo bảng, nhập, truy vấn, cập nhật, xóa dữ liệu. Còn phân tích thời gian thực và hiệu năng cao chắc chắn phải là do thiết kế của nó rồi. Thêm nữa Druid chỉ hỗ trợ chạy trên môi trường linux và mac os thôi nhé.

# Thiết kế của Druid
Kiến trúc của Druid là sự kết hợp của các ý tưởng của các hệ thống data warehouses, timeseries databases, logsearch. Một số tính năng chính của Druid như:
1. Định dạng lưu trữ dạng cột: Druid sử dụng lưu trữ hướng cột, điều này làm cho việc truy vấn nhanh hơn do chỉ cần load các cột cần thiết cho truy vấn thôi, bên cạnh đó các dữ liệu trong cột là cùng loại nên sẽ tối ưu được việc lưu trữ, hỗ trợ scan và tổng hợp nhanh.
2. Hệ thống phân tán có thể scale: Thường được triển khai trên cluster từ 10 đến 100 servers.
3. Xử lý song song mạnh mẽ: Có thể xử lý song song truy vấn trên toàn bộ cluster.
4. Nhập dữ liệu real-time hoặc xử lý theo lô.
5. Tự phục hồi, tự cân bằng tải, dễ vận hành.
6. Cloud-native, kiến trúc chịu lỗi: Một bản copy dữ liệu lưu ở deep storage. Khi triển khai nhiều server, các replication đảm bảo tính sẵn sàng khi một số server bị fail.
7. Các chỉ mục cho việc lọc nhanh: Sử dụng chỉ mục bitmap nén <a href="https://roaringbitmap.org/">Roaring</a> hoặc <a href="https://arxiv.org/pdf/1004.0403">CONCISE</a> để tạo chỉ mục hỗ trợ việc lọc và tìm kiếm nhanh trên nhiều cột.
8. Partition dựa trên thời gian
9. Thuật toán xấp xỉ: Sử dụng thuật toán xấp xỉ tối ưu thời gian truy vấn và bộ nhớ. Trong trường hợp độ chính xác là quan trọng thì nó sử dụng thuật toán chính xác.
10. Tự động tổng hợp dữ liệu khi nhập: Có thể tổng hợp dữ liệu ở thời gian nhập cho tối ưu việc lưu trữ và tăng hiệu suất.

# Khi nào nên sử dụng Druid
* Nhu cầu insert lớn, update thấp
* Các câu truy vấn chủ yếu là tổng hợp và báo cáo ("group by"). Có thể có thêm các câu truy vấn tìm kiếm và quét.
* Mong muốn độ trễ so với thời gian thực của truy vấn từ 100ms đến vài giây.
* Dữ liệu có các thành phần thành phần thời gian.
* Dữ liệu thường có nhiều hơn một bảng nhưng các câu truy vấn chỉ truy cập vào một bảng phân phối lớn, các truy vấn có thể có nhiều hơn một bảng tra cứu nhỏ - đoạn này hơi khó hiểu đấy.
* Dữ liệu có các cột số lượng cao, cần đếm nhanh và xếp loại chúng.
* Muốn load dữ liệu từ Kafka, HDFS, flat file, hoặc đối tượng lưu trữ như AWS S3.

Tạm thời dừng lại ở đây nhé, đón xem series tìm hiểu Apache Druid của mình

# Tài liệu tham khảo
[1] <a href="http://druid.apache.org/docs/latest/design/index.html">http://druid.apache.org/docs/latest/design/index.html</a>
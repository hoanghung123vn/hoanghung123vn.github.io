---
layout: post
title: Introduction to Druid
---

# Mục đích bài viết
Mình đang tìm hiểu để làm một hệ thống thống kê truy cập trên một hệ thống website. Sản phẩm công ty mình đang làm là một nền tảng thương mại điện tử, tự động tạo ra trang web cho người dùng chỉ với vài bước đơn giản. Số lượng website rất lớn cho nên nhu cầu cũng là rất lớn. Lõi của một hệ thống báo cáo thống kê đương nhiên phải là một hệ quản trị cơ sở dữ liệu tốt rồi. Apache Druid là một CSDL được thiết kế với mục đích tối ưu lưu trữ, truy vấn nhanh, phân tán, xử lý song song, dễ dàng scale. Và đương nhiên em ấy là một open source, ngon bổ rẻ nên xài thôi. Bài viết này sẽ giới thiệu về Druid, các key features của nó, khi nào nên sử dụng.

# Druid là gì ?
Trên trang chủ tài liệu có ghi một câu "Apache Druid is a high performance real-time analytics database" - một CSDL phân tích thời gian thực với hiệu năng cao. Bản chất nó vẫn là một hệ quản trị CSDL, vẫn có tạo bảng, nhập, truy vấn, cập nhật, xóa dữ liệu. Còn phân tích thời gian thực và hiệu năng cao chắc chắn phải là do thiết kế của nó rồi.

# Thiết kế của Druid
Kiến trúc của Druid là sự kết hợp của các ý tưởng của các hệ thống data warehouses, timeseries databases, logsearch. Một số tính năng chính của Druid như:
1. Định dạng lưu trữ dạng cột: Druid sử dụng lưu trữ hướng cột, điều này làm cho việc truy vấn nhanh hơn do chỉ cần load các cột cần thiết cho truy vấn thôi, bên cạnh đó các dữ liệu trong cột là cùng loại nên sẽ tối ưu được việc lưu trữ, hỗ trợ scan và tổng hợp nhanh.
2. Hệ thống phân tán có thể scale: Thường được triển khai trên cluster từ 10 đến 100 servers
3. Xử lý song song mạnh mẽ: Có thể xử lý song song truy vấn trên toàn bộ cluster
4. Nhập dữ liệu real-time hoặc theo lô
5. Tự phục hồi, tự cân bằng tải, dễ vận hành

To be continous

# Tài liệu tham khảo
[1] http://druid.apache.org/docs/latest/design/index.html
---
layout: post
title: 001 Making Visit Statistical System
categories: ["Statistical System"]
---

# Mục đích bài viết
Tìm hiểu các thành phần của một hệ thống thống kê truy cập website thương mại điện tử. Một hệ thống thống kê truy cập cho ta thấy được một cái nhìn tổng quan về các hoạt động đang diễn ra trên trang web của mình.

# Shopify use case
1. Khung nhìn trực tuyến
Cho phép theo dõi hoạt động trên trang web của bạn theo thời gian thực. Các metric được lựa chọn như:
- Số người đang truy cập trực tuyến và hoạt động trong vòng 5p gần nhất.
- Tổng số phiên hoạt động trong ngày.
- Tổng số đơn hàng được đặt trong ngày.
- Tổng số lợi nhuận trong ngày.
- Hành vi của người dùng: Số người truy cập có thêm một hay nhiều món đồ vào giỏ hàng trong vòng 10p gần nhất.
- Hành vi của người dùng: Số người truy cập có thêm một hay nhiều món đồ vào giỏ hàng, sau đó đến trang checkout và submit các thông tin liên hệ trong vòng 10p gần nhất.
- Hành vi của người dùng: Số người truy cập có thêm một hay nhiều món đồ vào giỏ hàng, sau đó đến trang checkout, submit thông tin liên hệ và thực sự trả tiền cho món đồ trong vòng 10p gần nhất.
- Số trang của cửa hàng bạn được xem bởi người truy cập trong vòng 10p.

2. Sự khác biệt số liệu phân tích so với một số hệ thống phân tích khác
- Khác nhau ở cách xác định 1 page được reload và xác định người dùng duy nhất truy cập được đếm.
- Khác nhau ở cách 1 sesion được định nghĩa, bỏ qua các session của các con bots. 

# Tài liệu tham khảo
[1] 
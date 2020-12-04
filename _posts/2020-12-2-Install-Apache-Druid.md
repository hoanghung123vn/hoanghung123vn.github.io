---
layout: post
title:  002 Apache Druid - Install Apache Druid
---

# Mục đích bài viết
Tiếp tục series tìm hiểu Apache Druid, trong bài viết này chúng ta sẽ cùng tìm hiểu cách cài đặt Druid. Druid cung cấp 2 cách cài đặt, một là cài đặt trên một single machine, hai là cài đặt trên một cluster.

# Cài đặt trên single machine
1. Yêu cầu môi trường
- Hệ điều hành linux, mac os, or các unix-like khác
- Java 8 or later
2. Bước 1 - Install
- Download Druid
```shell
wget https://mirror.downloadvn.com/apache/druid/0.20.0/apache-druid-0.20.0-bin.tar.gz
tar -xzf apache-druid-0.20.0-bin.tar.gz
cd apache-druid-0.20.0
```
2. Bước 2 - Start server
- Druid cung cấp các script khác nhau để start một server tùy theo cấu hình server <a href="http://druid.apache.org/docs/latest/operations/single-server.html">chi tiết</a>. Ở ví dụ này chúng ta sẽ start 1 script cho micro server
```
./bin/start-micro-quickstart
```
- Output sẽ được như thế này
```
$ ./bin/start-micro-quickstart
[Fri May  3 11:40:50 2019] Running command[zk], logging to[/apache-druid-0.20.0/var/sv/zk.log]: bin/run-zk conf
[Fri May  3 11:40:50 2019] Running command[coordinator-overlord], logging to[/apache-druid-0.20.0/var/sv/coordinator-overlord.log]: bin/run-druid coordinator-overlord conf/druid/single-server/micro-quickstart
[Fri May  3 11:40:50 2019] Running command[broker], logging to[/apache-druid-0.20.0/var/sv/broker.log]: bin/run-druid broker conf/druid/single-server/micro-quickstart
[Fri May  3 11:40:50 2019] Running command[router], logging to[/apache-druid-0.20.0/var/sv/router.log]: bin/run-druid router conf/druid/single-server/micro-quickstart
[Fri May  3 11:40:50 2019] Running command[historical], logging to[/apache-druid-0.20.0/var/sv/historical.log]: bin/run-druid historical conf/druid/single-server/micro-quickstart
[Fri May  3 11:40:50 2019] Running command[middleManager], logging to[/apache-druid-0.20.0/var/sv/middleManager.log]: bin/run-druid middleManager conf/druid/single-server/micro-quickstart
```

- Tất cả các thông tin của Druid lưu trữ ở thư mục <code>var</code>, log file lưu ở <code>var/sv</code>. Khi bạn muốn reset CSDL về ban đầu thì chỉ cần xóa thư mục này là được.

3. Bước 3 - Open web console
- Druid cung cấp cho chúng ta một giao diện web để quản lý dịch vụ, mở browser <a href="http://localhost:8888">http://localhost:8888</a>.
![alt text](https://druid.apache.org/docs/latest/assets/tutorial-quickstart-01.png)
- Druid có thể mất một chút thời gian để khởi động, trước thời gian này khi vào web console có thể có lỗi, chờ một lúc và refresh lại browser của bạn.
- Như vậy là bạn có thể bắt đầu sử dụng Druid để thao tác với dữ liệu của bạn rồi.

# Cài đặt trên một cluster
Đối với những ai cuồng docker như mình thì chắc chắn cài đặt trên cluster với docker là lựa chọn số 1 rồi, bạn không cần phải care môi trường và cấu hình máy, mọi chuyện đã có docker lo. Chỉ vài bước đơn giản bạn đã có dịch vụ Druid chạy trên cluster của mình. Yêu cầu duy nhất tất nhiên là bạn cần phải có docker.
1. Bước 1 - Docker compose
- Thực hiện các lệnh sau để chuẩn bị chạy với docker compose:
```shell
cd ~ && mkdir -p druid
cd druid
# Docker compose file
wget https://raw.githubusercontent.com/apache/druid/master/distribution/docker/docker-compose.yml
# Environment file
wget https://raw.githubusercontent.com/apache/druid/master/distribution/docker/environment
# Create deep storage for cluster
mkdir storage
# Run docker compose
docker-compose up -d
```
2. Bước 2 - Open web console
- Mở browser <a href="http://localhost:8888">http://localhost:8888</a> và chúng ta có kết quả như cách cài đặt với single machine.

Note: Giờ đây các thành phần của Druid được phân tách ra thành các dịch vụ chạy độc lập, có thể scale dễ dàng.
![alt text](https://www.baeldung.com/wp-content/uploads/2020/06/Druid-Processes-768x453.jpg)

Tạm kết: Tiếp tục theo dõi series bài viết để biết cách nhập dữ liệu và truy vấn với Druid nhé. Happy hacking!!!
<!-- ![alt text](http://bizweb.dktcdn.net/thumb/grande/dev/100/005/238/products/ls.png?v=1607057688903) -->
# Tài liệu tham khảo
[1] <a href="http://druid.apache.org">http://druid.apache.org</a>
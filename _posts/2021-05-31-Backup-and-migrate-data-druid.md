---
layout: post
title: Backup and migrate data
categories: ["Druid Advanced"]
---

# Mục đích bài viết
Các bước cần thiết để thực hiện di chuyển deep storage của bạn từ local lên S3 hoặc HDFS.

# Tổng quan
Nếu như bạn đang chạy một cụm Druid có giá trị sử dụng local deep storage và có mong muốn chuyển sang hệ thống có khả năng sản xuất cao hơn như S3 hay HDFS, sau đây là các bước cần thiết cho sự di chuyển: 
- Sao chép các segments từ local storage sang deep storage mới.
- Exports các segments của Druid từ metadata.
- Ghi đè lại các thông số load trong dữ liệu segment đã export để ánh xạ vị trí lưu trữ sâu mới.
- Import lại các segment đã chỉnh sửa vào metadata.
# Note
Trong quá trình migrate, đảm bảo rằng các tiến trình khác coordinator đã được tắt để đảm bảo trạng thái của metadata không thay đổi. Khi di chuyển từ Derby metadata, coordinator sẽ vẫn cần thiết trong quá trình khởi tạo, vì nó lưu trữ cơ sở dữ liệu Derby.

# Các bước thực hiện
1. Sao chép segments từ deep storage cũ đến deep storage mới
    - Nếu như bạn sử dụng Derby metadata, Druid hỗ trợ một export metadata tool, khi export matadata thì tự động copy deep storage cũ vào deep storage mới luôn. Khi copy dữ liệu, thao tác ghi đè của tool yêu cầu cấu trúc thư mục của các segments không bị thay đổi. Đối với deep storage di chuyển là S3, chỉ định: 
        - `--s3bucket`, `-b`: Tên của S3 buket
        - `--s3baseKey`, `-k`: Key sử dụng S3 bucket
    

        Đối với local deep storage, chỉ định:
        - `--newLocalPath`, `-n`: Đường dẫn mới lưu trữ segments
    - Nếu như bạn sử dụng các metadata của các cơ sở dữ liệu khác Derby, Druid không hỗ trợ tool migrate, nếu là deep storage S3 thì có thể sử dụng copy to S3 object, nếu deep storage là local thì có thể copy trực tiếp các segment.
2. Export segment cùng với ghi đè load specs
- Khi sử dụng Derby là metadata, sử dụng tool export có sẵn của Druid. Bằng cách thiết lập các tham số di chuyển deep storage như trong bước 1, tool này sẽ export metadata ra file CSV mà sau đó chúng ta có thể import lại.
- Thực hiện export metadata bằng câu lệnh sau:
```bash
cd ${DRUID_ROOT}
mkdir -p /tmp/csv
java -classpath "lib/*" -Dlog4j.configurationFile=conf/druid/cluster/_common/log4j2.xml -Ddruid.extensions.directory="extensions" -Ddruid.extensions.loadList=[] org.apache.druid.cli.Main tools export-metadata --connectURI "jdbc:derby://localhost:1527/var/druid/metadata.db;" -o /tmp/csv
```

Trong ví dụ câu lệnh trên:

- `lib` là thư mục lib của Druid
- `extensions` là thư mục extensions của Druid
- `/tmp/csv` là thư mục output của file CSV được xuất ra

Nếu như bạn không sử dụng Derby là metadata thì bạn có thể export data ra file CSV bằng hỗ trợ riêng của từng loại cơ sở dữ liệu, ví dụ nếu bạn sử dụng Postgres là metadata thì có thể sử dụng psql để export bảng `druid_segments`:
```bash
psql -U druid -d druid
\copy druid_segments to segments.csv csv header;
```

Sau khi quá trình export thành công, bạn có thể tắt hoàn toàn cụm Druid.
3. Import lại metadata
- Tương ứng với từng loại cơ sở dữ liệu sẽ có các câu lệnh khác nhau để import lại metadata, chỉ bảng `druid_segments` là cần import data, nếu bạn sử dụng Postgres, sử dụng psql để import lại metadata của bạn:
```bash
psql -U druid -d druid
\copy druid_segments from segments.csv csv header;
```

Sau khi quá trình import hoàn tất, bạn có thể restart cụm Druid của mình để load lại deep storage mới.
# Kết luận
Trong bài này chúng ta đã đi qua các bước cần thiết để có thể migrate cũng như backup dữ liệu trong một cụm Druid đang chạy. Trong môi trường production điều này là rất cần thiết để chuyển từ một mô hình nhỏ sang một hệ thống có khả năng sản xuất cao hơn. Một lưu ý nữa là hiện tại tool export metadata chỉ mới hỗ trợ cơ sở dữ liệu Derby, những cơ sở dữ liệu còn lại thì bạn phải tự nghĩ cách thooiii :))
# Tài liệu tham khảo
[1] http://druid.apache.org/docs/latest/operations/deep-storage-migration.html

[2] http://druid.apache.org/docs/latest/operations/export-metadata.html#deep-storage-migration
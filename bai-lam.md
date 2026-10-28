1. Cấu hình client_max_body_size
Thêm vào bên trong server block trong /etc/nginx/sites-available/default:
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    server_name _;

    # Limit request body to 10 MB
    client_max_body_size 10M;

    # Các location hiện có giữ nguyên
}

- Giới hạn kích thước request body ở mức 10 MB.
- Nếu request vượt giới hạn, Nginx thường trả về lỗi 413 Request Entity Too Large.
- Đặt ở cấp server sẽ áp dụng cho các location thuộc server block đó.
2. Cấu hình Gzip
Thêm vào bên trong khối http { ... } của /etc/nginx/nginx.conf:
# Gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_comp_level 5;

gzip_types
    text/plain
    text/css
    text/javascript
    application/javascript
    application/json
    application/xml
    image/svg+xml;

Directive	Ý nghĩa
gzip on;	Bật nén Gzip.
gzip_vary on;	Thêm header Vary: Accept-Encoding.
gzip_min_length 1024;	Chỉ nén response có độ dài tối thiểu 1024 byte.
gzip_comp_level 5;	Mức nén 5, cân bằng giữa CPU và kích thước dữ liệu.
gzip_types	Xác định các loại nội dung bổ sung được nén.


Lưu ý: text/html được Nginx xử lý mặc định khi Gzip bật, nên không cần thêm vào gzip_types.
3. Zero-downtime reload
Sau khi sửa cấu hình, chạy:
sudo nginx -t && sudo systemctl reload nginx

- nginx -t: Kiểm tra cú pháp và tính hợp lệ của cấu hình.
- &&: Chỉ thực hiện lệnh tiếp theo nếu kiểm tra thành công.
- systemctl reload nginx: Yêu cầu Nginx nạp cấu hình mới mà không cần dừng toàn bộ dịch vụ.
Vì sao reload hạn chế downtime?
Khi reload, Nginx tạo các worker mới sử dụng cấu hình mới. Các worker cũ ngừng nhận kết nối mới nhưng có thể tiếp tục xử lý những kết nối hiện tại trước khi thoát.

Nhờ đó, Nginx thường có thể áp dụng cấu hình mới mà không làm gián đoạn đáng kể việc phục vụ request. Đây là cơ chế graceful reload, thường được gọi là zero-downtime reload.

Tuy nhiên, zero downtime không được bảo đảm tuyệt đối trong mọi tình huống. Nếu cấu hình sai, tài nguyên cạn kiệt hoặc dịch vụ phụ trợ gặp sự cố, vẫn có thể xảy ra gián đoạn.

Nguyên tắc vận hành: Kiểm tra bằng nginx -t trước, sau đó dùng reload thay vì restart khi chỉ cần áp dụng thay đổi cấu hình

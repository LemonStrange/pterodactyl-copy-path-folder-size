# Pterodactyl Copy Path + Folder Size

Patch bổ sung hai tiện ích cho Pterodactyl Panel 1.15.1:

- **Copy Path**: quản trị viên root có thể sao chép đường dẫn volume của thư mục hiện tại.
- **Size**: quản trị viên root có thể tính dung lượng thư mục trực tiếp trên từng dòng folder.

Patch đã được triển khai và kiểm thử trên panel Pterodactyl 1.15.1. Việc tính dung lượng có cache 5 phút, giới hạn thời gian 25 giây và không cần restart Minecraft server.

## Cấu trúc

~~~text
patches/
├── 01-folder-size-controller.patch
├── 02-folder-size-route.patch
├── 03-copy-path-ui.patch
└── 04-folder-size-ui.patch
helper/
└── ptero-folder-size
~~~

## Yêu cầu

- Pterodactyl Panel 1.15.1 hoặc mã nguồn tương thích.
- Node/Yarn để build frontend.
- sudo, du, nice, ionice; ssh chỉ cần khi volume nằm trên host khác.
- Chỉ tài khoản root_admin mới thấy hai nút này.

## Cài đặt

Sao lưu panel trước khi thay đổi:

~~~sh
cd /var/www/pterodactyl
sudo tar -czf /root/pterodactyl-panel-backup-$(date +%Y%m%d-%H%M%S).tar.gz \
  app/Http/Controllers/Api/Client/Servers/FileController.php \
  routes/api-client.php \
  resources/scripts/components/server/files/FileManagerContainer.tsx \
  resources/scripts/components/server/files/FileObjectRow.tsx
~~~

Áp dụng bốn patch:

~~~sh
cd /var/www/pterodactyl
git apply /path/to/pterodactyl-copy-path-folder-size/patches/01-folder-size-controller.patch
git apply /path/to/pterodactyl-copy-path-folder-size/patches/02-folder-size-route.patch
git apply /path/to/pterodactyl-copy-path-folder-size/patches/03-copy-path-ui.patch
git apply /path/to/pterodactyl-copy-path-folder-size/patches/04-folder-size-ui.patch
~~~

Cài helper và phân quyền:

~~~sh
install -o root -g root -m 0750 \
  /path/to/pterodactyl-copy-path-folder-size/helper/ptero-folder-size \
  /usr/local/sbin/ptero-folder-size
~~~

Cho phép panel gọi đúng một helper bằng sudo không cần mật khẩu:

~~~sh
printf '%s\n' \
  'www-data ALL=(root) NOPASSWD: /usr/local/sbin/ptero-folder-size *' \
  > /etc/sudoers.d/pterodactyl-folder-size
chmod 0440 /etc/sudoers.d/pterodactyl-folder-size
visudo -cf /etc/sudoers.d/pterodactyl-folder-size
~~~

Build và xóa cache Laravel. Các lệnh php artisan phải chạy dưới www-data:

~~~sh
cd /var/www/pterodactyl
sudo -u www-data yarn build:production
sudo -u www-data php artisan optimize:clear
~~~

Sau đó mở lại trang Files trên panel. Không cần restart Minecraft server.

## Cách hoạt động

Copy Path ghép:

~~~text
/var/lib/pterodactyl/volumes/<server-uuid>/<current-directory>
~~~

Endpoint GET /api/client/servers/{server}/files/folder-size chỉ cho root administrator gọi. Panel ưu tiên helper chạy trên host volume; nếu helper không thành công, controller dùng API Wings để duyệt thư mục con.

Helper kiểm tra UUID, chống thoát khỏi volume bằng realpath, chạy du -sb với timeout, nice và ionice, đồng thời hỗ trợ chuyển tiếp qua SSH khi volume không nằm trên host hiện tại.

## Kiểm tra nhanh

~~~sh
/usr/local/sbin/ptero-folder-size <server-uuid> /plugins
~~~

Kết quả là số byte dạng số nguyên. Trên giao diện:

1. Đăng nhập bằng tài khoản root administrator.
2. Vào **Files**.
3. Nút **Copy Path** xuất hiện cạnh đường dẫn hiện tại.
4. Nút **Size** xuất hiện ở các dòng thư mục.

## Rollback

~~~sh
cd /var/www/pterodactyl
git apply -R patches/04-folder-size-ui.patch
git apply -R patches/03-copy-path-ui.patch
git apply -R patches/02-folder-size-route.patch
git apply -R patches/01-folder-size-controller.patch
rm -f /usr/local/sbin/ptero-folder-size
rm -f /etc/sudoers.d/pterodactyl-folder-size
sudo -u www-data yarn build:production
sudo -u www-data php artisan optimize:clear
~~~

## Ghi chú bảo mật

Đường dẫn host chỉ được gửi vào clipboard của trình duyệt root administrator. Không nên bật tính năng này cho user thường nếu chưa rà soát chính sách bảo mật của máy chủ.

## Giấy phép

Patch này được phát hành theo MIT. Pterodactyl Panel và các thành phần gốc vẫn thuộc bản quyền và giấy phép tương ứng của dự án Pterodactyl.

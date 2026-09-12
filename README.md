# Folder Size for Pterodactyl

Bản vá nhỏ cho Pterodactyl Panel 1.15.1: thêm nút `Size` vào các thư mục trong File Manager.

Tính năng chỉ dành cho `root_admin`. Panel ưu tiên helper chạy trên node chứa volume, có fallback qua Wings API, cache kết quả 5 phút và giới hạn mỗi lần quét 25 giây. Không cần restart Minecraft server.

## Cấu trúc

```text
patches/
├── 01-folder-size-controller.patch
├── 02-folder-size-route.patch
└── 03-folder-size-ui.patch
helper/
└── ptero-folder-size
```

## Cài đặt

Patch này dành cho panel 1.15.1 hoặc mã nguồn tương thích. Nên sao lưu các file liên quan trước khi áp dụng:

```sh
cd /var/www/pterodactyl
sudo tar -czf /root/pterodactyl-panel-backup-$(date +%Y%m%d-%H%M%S).tar.gz \
  app/Http/Controllers/Api/Client/Servers/FileController.php \
  routes/api-client.php \
  resources/scripts/components/server/files/FileObjectRow.tsx
```

Áp dụng patch:

```sh
cd /var/www/pterodactyl
git apply /path/to/pterodactyl-folder-size/patches/01-folder-size-controller.patch
git apply /path/to/pterodactyl-folder-size/patches/02-folder-size-route.patch
git apply /path/to/pterodactyl-folder-size/patches/03-folder-size-ui.patch
```

Cài helper:

```sh
install -o root -g root -m 0750 \
  /path/to/pterodactyl-folder-size/helper/ptero-folder-size \
  /usr/local/sbin/ptero-folder-size

printf '%s\n' \
  'www-data ALL=(root) NOPASSWD: /usr/local/sbin/ptero-folder-size *' \
  | sudo tee /etc/sudoers.d/pterodactyl-folder-size >/dev/null
sudo chmod 0440 /etc/sudoers.d/pterodactyl-folder-size
sudo visudo -cf /etc/sudoers.d/pterodactyl-folder-size
```

Build frontend và xóa cache Laravel. Các lệnh `php artisan` phải chạy dưới `www-data`:

```sh
cd /var/www/pterodactyl
sudo -u www-data yarn build:production
sudo -u www-data php artisan optimize:clear
```

Mở **Files**, chọn một thư mục và bấm **Size**. Kết quả hiển thị theo định dạng dung lượng của panel.

## Helper

Có thể kiểm tra riêng bằng:

```sh
/usr/local/sbin/ptero-folder-size <server-uuid> /plugins
```

Helper kiểm tra UUID và đường dẫn nằm trong volume, bỏ qua symlink, giới hạn thời gian chạy và dùng `du -sb`. Tham số host chỉ dùng khi cần chuyển tiếp sang node khác.

## Gỡ bỏ

```sh
cd /var/www/pterodactyl
git apply -R /path/to/pterodactyl-folder-size/patches/03-folder-size-ui.patch
git apply -R /path/to/pterodactyl-folder-size/patches/02-folder-size-route.patch
git apply -R /path/to/pterodactyl-folder-size/patches/01-folder-size-controller.patch
sudo rm -f /usr/local/sbin/ptero-folder-size
sudo rm -f /etc/sudoers.d/pterodactyl-folder-size
sudo -u www-data yarn build:production
sudo -u www-data php artisan optimize:clear
```

## Giấy phép

MIT. Pterodactyl Panel và các thành phần gốc vẫn thuộc giấy phép tương ứng của dự án Pterodactyl.

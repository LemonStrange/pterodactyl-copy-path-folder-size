# Folder Size cho Pterodactyl

Bản vá này thêm nút `Size` vào danh sách thư mục của Pterodactyl Panel. Bấm vào nút để xem dung lượng ngay trên trang **Files**.

Đã kiểm tra trên Pterodactyl 1.15.1. Với bản khác, patch có thể cần chỉnh lại đôi chút.

## Trong repo

```text
patches/
├── 01-folder-size-controller.patch
├── 02-folder-size-route.patch
└── 03-folder-size-ui.patch
helper/
└── ptero-folder-size
```

## Cài đặt

Nên sao lưu các file liên quan trước khi bắt đầu:

```sh
cd /var/www/pterodactyl
sudo tar -czf /root/pterodactyl-panel-backup-$(date +%Y%m%d-%H%M%S).tar.gz \
  app/Http/Controllers/Api/Client/Servers/FileController.php \
  routes/api-client.php \
  resources/scripts/components/server/files/FileObjectRow.tsx
```

Áp dụng ba patch:

```sh
cd /var/www/pterodactyl
git apply --check /path/to/pterodactyl-folder-size/patches/01-folder-size-controller.patch
git apply --check /path/to/pterodactyl-folder-size/patches/02-folder-size-route.patch
git apply --check /path/to/pterodactyl-folder-size/patches/03-folder-size-ui.patch

git apply /path/to/pterodactyl-folder-size/patches/01-folder-size-controller.patch
git apply /path/to/pterodactyl-folder-size/patches/02-folder-size-route.patch
git apply /path/to/pterodactyl-folder-size/patches/03-folder-size-ui.patch
```

Cài helper:

```sh
sudo install -o root -g root -m 0750 \
  /path/to/pterodactyl-folder-size/helper/ptero-folder-size \
  /usr/local/sbin/ptero-folder-size

printf '%s\n' \
  'www-data ALL=(root) NOPASSWD: /usr/local/sbin/ptero-folder-size *' \
  | sudo tee /etc/sudoers.d/pterodactyl-folder-size >/dev/null

sudo chmod 0440 /etc/sudoers.d/pterodactyl-folder-size
sudo visudo -cf /etc/sudoers.d/pterodactyl-folder-size
```

Build lại frontend và xóa cache:

```sh
cd /var/www/pterodactyl
sudo -u www-data yarn build:production
sudo -u www-data php artisan optimize:clear
```

Sau đó vào **Files**, chọn một thư mục và bấm **Size**.

## Helper

Có thể chạy thử trực tiếp:

```sh
/usr/local/sbin/ptero-folder-size <server-uuid> /plugins
```

Helper dùng `du -sb`, kiểm tra đường dẫn trong volume, bỏ qua symlink và có giới hạn thời gian. Tham số host thứ ba chỉ cần khi volume nằm trên node khác.

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

MIT. Pterodactyl Panel và các thành phần gốc vẫn giữ giấy phép tương ứng của dự án Pterodactyl.

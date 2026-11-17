RUZX WEB — Website Auto Order

RUZX Store | Still Online While You Offline

Website toko digital RUZX Store dengan sistem katalog produk, checkout pelanggan, pembayaran Midtrans Snap, dan penyimpanan transaksi menggunakan MySQL.

Tech Stack

- Frontend: HTML, CSS, JavaScript
- Backend: Node.js, Express.js
- Database: MySQL
- Payment Gateway: Midtrans Snap
- Process Manager: PM2
- Web Server: Nginx
- Runtime Port: 3000

Fitur

- Katalog produk dinamis dari database MySQL.
- Form pembelian pelanggan.
- Pembuatan transaksi Midtrans Snap.
- Pembayaran melalui metode yang tersedia di akun Midtrans, termasuk QRIS jika diaktifkan.
- Webhook notifikasi pembayaran.
- Verifikasi status transaksi melalui Midtrans.
- Penyimpanan data order.
- Pemantauan status pesanan.

Struktur Folder

ruzx-web/
├── .env.example
├── .gitignore
├── database.sql
├── package.json
├── package-lock.json
├── public/
│   ├── index.html
│   ├── app.js
│   └── style.css
└── src/
    ├── db.js
    ├── midtrans.js
    └── server.js

1. Persiapan VPS

Rekomendasi:

- Ubuntu 22.04 atau 24.04 LTS
- RAM minimal 1 GB
- Node.js 22 LTS
- MySQL 8
- Nginx

Login VPS:

ssh root@IP_VPS

Update sistem:

apt update && apt upgrade -y

Install kebutuhan:

apt install -y curl git unzip nginx mysql-server

2. Install Node.js

curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

Periksa instalasi:

node -v
npm -v

Install PM2:

npm install -g pm2

3. Download Source Code

Buat direktori website:

mkdir -p /var/www/ruzx-web

Clone repository:

git clone https://github.com/rassakiii025-ui/website-auto-order- /tmp/ruzx-repository

Ekstrak ZIP ke direktori sementara:

mkdir -p /tmp/ruzx-extract
unzip /tmp/ruzx-repository/ruzx-web.zip -d /tmp/ruzx-extract

Salin source code ke direktori website:

cp -a /tmp/ruzx-extract/ruzx-web/ruzx-web/. /var/www/ruzx-web/

Masuk ke direktori:

cd /var/www/ruzx-web

Periksa:

ls -la

Pastikan terdapat "package.json", "database.sql", "public/", dan "src/".

4. Install Dependencies

npm install --omit=dev

5. Konfigurasi MySQL

Aktifkan MySQL:

systemctl enable --now mysql

Masuk ke MySQL:

mysql

Buat database dan user:

CREATE DATABASE ruzx_web
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

CREATE USER 'ruzx_user'@'localhost'
IDENTIFIED BY 'GANTI_PASSWORD_KUAT';

GRANT ALL PRIVILEGES ON ruzx_web.*
TO 'ruzx_user'@'localhost';

FLUSH PRIVILEGES;
EXIT;

Import database:

mysql -u ruzx_user -p ruzx_web < /var/www/ruzx-web/database.sql

Periksa tabel:

mysql -u ruzx_user -p ruzx_web

SHOW TABLES;
SELECT * FROM products;
EXIT;

Database menyediakan dua tabel:

- "products": informasi produk dan harga.
- "orders": informasi pelanggan, transaksi, dan status pembayaran.

6. Konfigurasi Environment

Buat file ".env":

nano /var/www/ruzx-web/.env

Isi:

PORT=3000
APP_URL=https://domainkamu.com

MIDTRANS_IS_PRODUCTION=false
MIDTRANS_SERVER_KEY=ISI_SERVER_KEY
MIDTRANS_CLIENT_KEY=ISI_CLIENT_KEY

DB_HOST=localhost
DB_USER=ruzx_user
DB_PASSWORD=PASSWORD_DATABASE
DB_NAME=ruzx_web
DB_PORT=3306

Gunakan Sandbox Key untuk pengujian.

Setelah akun dan metode pembayaran Production siap, ganti ke Production Key dan ubah:

MIDTRANS_IS_PRODUCTION=true

Lindungi konfigurasi:

chmod 600 /var/www/ruzx-web/.env

Jangan upload ".env" ke GitHub.

7. Konfigurasi Midtrans

Login:

https://dashboard.midtrans.com/

Langkah:

1. Selesaikan aktivasi akun dan verifikasi bisnis.
2. Buka menu pengaturan integrasi.
3. Ambil Server Key dan Client Key.
4. Masukkan key ke ".env".
5. Aktifkan metode pembayaran QRIS jika tersedia pada akun.
6. Pastikan konfigurasi Snap menggunakan environment yang sama dengan backend.

Notification URL:

https://domainkamu.com/api/notification

Endpoint callback yang digunakan project:

POST /api/notification

Backend mengambil status transaksi dari Midtrans dan memperbarui status order di MySQL.

Status order yang disimpan:

- "pending"
- "paid"
- "failed"
- "expired"

Dokumentasi:

https://docs.midtrans.com/

Catatan integrasi: "public/index.html" pada source awal masih menggunakan alamat Snap Sandbox dan Client Key yang ditulis langsung di HTML. Sebelum Production, sesuaikan script Snap dengan environment Production dan gunakan Client Key dari endpoint "/api/config". Jangan pernah menaruh Server Key di frontend.

8. Menjalankan Website

Tes aplikasi:

cd /var/www/ruzx-web
npm start

Jika berhasil, terminal akan menampilkan:

Ruzx Store berjalan di port 3000

Hentikan proses pengujian menggunakan "CTRL + C".

Jalankan dengan PM2:

pm2 start npm --name ruzx-web -- start

Simpan proses:

pm2 save
pm2 startup

Ikuti perintah startup tambahan yang ditampilkan PM2.

Periksa:

pm2 status

Log:

pm2 logs ruzx-web

9. Konfigurasi Nginx

Buat konfigurasi:

nano /etc/nginx/sites-available/ruzx-web

Isi:

server {
    listen 80;
    server_name domainkamu.com www.domainkamu.com;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

Aktifkan:

ln -s /etc/nginx/sites-available/ruzx-web /etc/nginx/sites-enabled/ruzx-web

Tes:

nginx -t

Restart:

systemctl reload nginx

10. Aktivasi HTTPS

Install Certbot:

apt install -y certbot python3-certbot-nginx

Aktifkan SSL:

certbot --nginx -d domainkamu.com -d www.domainkamu.com

Pastikan DNS domain sudah diarahkan ke IP VPS.

11. Firewall

ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable

Jangan membuka port MySQL ke publik.

12. API Endpoint

Method| Endpoint| Fungsi
GET| "/api/config"| Mengambil Client Key
GET| "/api/products"| Mengambil daftar produk
POST| "/api/transaction"| Membuat transaksi
POST| "/api/notification"| Menerima notifikasi Midtrans
GET| "/api/order/:order_id"| Mengecek status order

13. Pengujian

Checklist:

- [ ] Website bisa dibuka.
- [ ] Produk muncul dari MySQL.
- [ ] Form pembelian berjalan.
- [ ] Transaksi berhasil dibuat.
- [ ] QRIS tersedia pada halaman pembayaran.
- [ ] Notifikasi pembayaran diterima.
- [ ] Status order diperbarui.
- [ ] Website tetap berjalan setelah restart VPS.

Gunakan transaksi Sandbox sebelum beralih ke Production.

14. Maintenance

Restart aplikasi:

pm2 restart ruzx-web

Lihat log:

pm2 logs ruzx-web

Periksa status:

pm2 status

Restart Nginx:

systemctl restart nginx

Restart MySQL:

systemctl restart mysql

15. Batasan Versi Saat Ini

Versi source code ini sudah memiliki:

- Katalog produk.
- Checkout.
- Pembuatan transaksi Midtrans Snap.
- Notifikasi pembayaran.
- Pembaruan status order.

Fitur yang belum diimplementasikan:

- Pengiriman file produk otomatis setelah pembayaran.
- Dashboard admin untuk mengelola produk.
- Dashboard riwayat pesanan pelanggan.
- Sistem akun pelanggan.

Jangan menganggap pesanan otomatis terkirim hanya karena status pembayaran sudah "paid". Pengiriman produk masih membutuhkan implementasi tambahan.

---

RUZX WEB — RUZX Store

Still Online While You Offline.

Repository: https://github.com/rassakiii025-ui/website-auto-order-

Payment Gateway: Midtrans
Backend: Node.js + Express
Database: MySQL

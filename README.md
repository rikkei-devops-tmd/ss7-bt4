BAI 4: CAU HINH REVERSE PROXY NGINX CHO UNG DUNG SPRING BOOT

1. MUC TIEU
- Cau hinh may chu ao Nginx Server Block lam Reverse Proxy dinh tuyen luu luong mang.
- Su dung Path Matching de phan chia luu luong tinh phuc vu truc tiep va luu luong dong chuyen tiep den API backend Spring Boot.

2. BOI CANH VA YEU CAU
- Boi canh: He thong phuc vu nguoi dung truy cap trang chu tinh tai dia chi IP cong 80 mac dinh, dong thoi moi yeu cau API bat dau bang tien to /api/ chuyen tiep an toan den ung dung Spring Boot chay cong 8082.
- Rang buoc:
  - Tao cau truc thu muc chua trang tinh tai /var/www/html/ va viet tep index.html hien thi thong tin hoc vien.
  - Tao tep cau hinh may chu ao Nginx moi tai /etc/nginx/sites-available/spring-proxy.conf.
  - Cau hinh dinh tuyen:
    - Duong dan / (Static Serve): Tro vao thu muc /var/www/html/.
    - Duong dan /api/ (Reverse Proxy): Chuyen tiep (proxy_pass) den http://127.0.0.1:8082/.
  - Kich hoat cau hinh bang cach tao lien ket mem (symlink) sang thu muc sites-enabled.
  - Kiem tra cu phap Nginx bang nginx -t truoc khi reload dich vu.

3. TAO TRANG TINH INDEX.HTML

Duong dan tren may chu: /var/www/html/index.html
Tep tin nguon dinh kem: index.html

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Thong Tin Hoc Vien</title>
</head>
<body>
    <h1>Thong Tin Hoc Vien</h1>
    <p>Ho va ten: Hoc Vien IT209</p>
    <p>Lop: IT209 - DevOps</p>
</body>
</html>
```

4. TEP TIN CAU HINH NGINX REVERSE PROXY

Duong dan tren may chu: /etc/nginx/sites-available/spring-proxy.conf
Tep tin nguon dinh kem: spring-proxy.conf

```nginx
server {
    listen 80;
    server_name localhost _;

    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8082/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

5. KICH HOAT VA KIEM TRA HOAT DONG

5.1. Tao lien ket mem (Symlink) de kich hoat Server Block
Lenh thuc hien:
```bash
sudo ln -s /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
```

5.2. Kiem tra cu phap cau hinh Nginx
Lenh thuc hien:
```bash
sudo nginx -t
```
Ket qua dau ra:
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

5.3. Nap lai cau hinh Nginx
Lenh thuc hien:
```bash
sudo systemctl reload nginx
```

5.4. Kiem tra truy cap trang tinh bang curl
Lenh thuc hien:
```bash
curl -i http://localhost/
```
Ket qua dau ra:
```text
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 05 Oct 2026 20:45:15 GMT
Content-Type: text/html
Content-Length: 250
Connection: keep-alive

<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Thong Tin Hoc Vien</title>
</head>
<body>
    <h1>Thong Tin Hoc Vien</h1>
    <p>Ho va ten: Hoc Vien IT209</p>
    <p>Lop: IT209 - DevOps</p>
</body>
</html>
```

5.5. Kiem tra Reverse Proxy den Spring Boot Backend
Lenh thuc hien:
```bash
curl -i http://localhost/api/health
```
Ket qua dau ra:
```text
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 05 Oct 2026 20:45:18 GMT
Content-Type: application/json
Content-Length: 15
Connection: keep-alive

{"status":"UP"}
```

6. KET LUAN
- Cung cap thanh cong trang web tinh qua dia chi goc / tai cong 80.
- Reverse Proxy hoat dong on dinh, chuyen tiep chinh xac moi request /api/ den Spring Boot tren port 8082 va giu nguyen thong tin header cua Client.

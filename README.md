# BT5 - DOCKER COMPOSE: APP MONITOR & REALTIME ALERT SYSTEM

# I. Lý Thuyết

## 1. Docker là gì?

Docker là nền tảng mã nguồn mở cho phép đóng gói ứng dụng cùng toàn bộ môi trường chạy (source code, thư viện, dependency, cấu hình...) vào một đơn vị gọi là **Container**.

Container giúp ứng dụng chạy giống nhau trên mọi môi trường:

* Laptop cá nhân
* Máy chủ nội bộ
* Cloud Server
* VPS
* Data Center

### Docker Architecture

```text
+---------------------+
|     Docker Host     |
+---------------------+
|     Containers      |
|  App1  App2  App3   |
+---------------------+
|    Docker Engine    |
+---------------------+
|   Linux/Windows OS  |
+---------------------+
```

### Các khái niệm quan trọng

| Thành phần     | Ý nghĩa                          |
| -------------- | -------------------------------- |
| Image          | Bộ cài đặt của ứng dụng          |
| Container      | Instance được tạo từ Image       |
| Dockerfile     | File mô tả cách build Image      |
| Volume         | Lưu dữ liệu bền vững             |
| Network        | Cho phép container giao tiếp     |
| Registry       | Kho lưu trữ Image                |
| Docker Compose | Quản lý nhiều container cùng lúc |

---

# 2. Các keyword thường dùng trong docker-compose.yml

## 2.1 version

Xác định phiên bản cú pháp Docker Compose.

```yaml
version: "3.9"
```

---

## 2.2 services

Khai báo các container sẽ chạy.

```yaml
services:
  nginx:
    image: nginx
```

---

## 2.3 image

Image dùng để tạo container.

```yaml
image: nginx:latest
```

---

## 2.4 container_name

Đặt tên container.

```yaml
container_name: webserver
```

---

## 2.5 build

Build image từ Dockerfile.

```yaml
build:
  context: .
  dockerfile: Dockerfile
```

---

## 2.6 ports

Map port host và container.

```yaml
ports:
  - "80:80"
```

Cú pháp:

```text
HOST_PORT:CONTAINER_PORT
```

---

## 2.7 environment

Khai báo biến môi trường.

```yaml
environment:
  MYSQL_ROOT_PASSWORD: 123456
  MYSQL_DATABASE: demo
```

---

## 2.8 env_file

Đọc biến môi trường từ file.

```yaml
env_file:
  - .env
```

---

## 2.9 volumes

Mount dữ liệu.

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

Hoặc:

```yaml
volumes:
  - ./html:/usr/share/nginx/html
```

---

## 2.10 restart

Chính sách tự khởi động lại.

```yaml
restart: always
```

Các giá trị:

```text
no
always
on-failure
unless-stopped
```

Ví dụ:

```yaml
restart: unless-stopped
```

---

## 2.11 depends_on

Khai báo phụ thuộc.

```yaml
depends_on:
  - mariadb
```

---

## 2.12 command

Ghi đè lệnh chạy mặc định.

```yaml
command: python app.py
```

---

## 2.13 networks

Khai báo network.

```yaml
networks:
  - backend
```

---

## 2.14 hostname

Đặt hostname trong container.

```yaml
hostname: dbserver
```

---

## 2.15 privileged

Cấp quyền cao.

```yaml
privileged: true
```

---

# 3. Khai báo Network

## Tạo network

```yaml
networks:
  monitoring_net:
```

## Sử dụng

```yaml
services:
  grafana:
    networks:
      - monitoring_net
```

---

# 4. Khai báo Volume

## Tạo volume

```yaml
volumes:
  mariadb_data:
  influxdb_data:
```

## Sử dụng

```yaml
services:
  mariadb:
    volumes:
      - mariadb_data:/var/lib/mysql
```

---

# 5. Mount File và Mount Thư Mục

## Mount File

```yaml
volumes:
  - ./nginx.conf:/etc/nginx/nginx.conf
```

### Ý nghĩa

Chỉ mount 1 file.

---

## Mount Folder

```yaml
volumes:
  - ./html:/usr/share/nginx/html
```

### Ý nghĩa

Mount toàn bộ thư mục.

---

# 6. Ưu điểm khi triển khai ứng dụng bằng Docker

## 6.1 Môi trường đồng nhất

Developer và Server sử dụng cùng môi trường.

```text
Works on my machine
↓
Works everywhere
```

---

## 6.2 Dễ triển khai

```bash
docker compose up -d
```

Là toàn bộ hệ thống được khởi động.

---

## 6.3 Dễ backup

Backup bằng:

```bash
docker save
docker export
```

---

## 6.4 Tách biệt ứng dụng

Mỗi service chạy trong container riêng.

Ví dụ:

```text
Node-RED
MariaDB
InfluxDB
Grafana
Flask API
Nginx
```

Không ảnh hưởng lẫn nhau.

---

## 6.5 Dễ mở rộng

Có thể scale service.

```bash
docker compose up --scale api=3
```

---

## 6.6 Di chuyển dễ dàng

Có thể chuyển nguyên hệ thống sang máy khác.

---

# 7. Triển khai ứng dụng Docker lên máy chủ không có Internet

Giả sử ứng dụng đã test thành công trên laptop.

## Bước 1. Kiểm tra toàn bộ Image

```bash
docker images
```

Ví dụ:

```text
nodered/node-red
mariadb
influxdb
grafana/grafana
nginx
python-flask-api
```

---

## Bước 2. Export toàn bộ Image

```bash
docker save -o project_images.tar \
nodered/node-red \
mariadb \
influxdb \
grafana/grafana \
nginx \
python-flask-api
```

---

## Bước 3. Backup Source Code

```bash
zip -r project.zip .
```

Hoặc:

```bash
tar -czvf project.tar.gz .
```

---

## Bước 4. Copy sang máy chủ

Có thể dùng:

```text
USB
Ổ cứng ngoài
Mạng LAN nội bộ
```

---

## Bước 5. Import Image

```bash
docker load -i project_images.tar
```

---

## Bước 6. Giải nén source

```bash
tar -xzvf project.tar.gz
```

---

## Bước 7. Khởi động hệ thống

```bash
docker compose up -d
```

---

# 8. Xuất toàn bộ Container ra file nén

## Export từng container

```bash
docker export nodered > nodered.tar
docker export grafana > grafana.tar
docker export mariadb > mariadb.tar
```

---

## Nén lại

```bash
tar -czvf backup_containers.tar.gz *.tar
```

---

# 9. Xóa toàn bộ Container

## Dừng container

```bash
docker stop $(docker ps -aq)
```

## Xóa container

```bash
docker rm $(docker ps -aq)
```

---

# 10. Khôi phục Container từ file nén

## Giải nén

```bash
tar -xzvf backup_containers.tar.gz
```

---

## Import lại

```bash
docker import nodered.tar nodered_restore
docker import grafana.tar grafana_restore
docker import mariadb.tar mariadb_restore
```

---

## Chạy lại

```bash
docker run -d --name nodered nodered_restore
docker run -d --name grafana grafana_restore
docker run -d --name mariadb mariadb_restore
```

---

# Kết luận

Docker Compose giúp triển khai hệ thống nhiều dịch vụ một cách nhanh chóng, đồng nhất và dễ bảo trì.

Trong bài thực hành BT5 sẽ sử dụng:

* Node-RED
* MariaDB
* InfluxDB
* Flask API
* Grafana
* Nginx
* Telegram Bot

để xây dựng hệ thống:

```text
Realtime Data Collection
        ↓
MariaDB + InfluxDB
        ↓
Flask API
        ↓
Frontend HTML/JS
        ↓
Grafana Dashboard
        ↓
Telegram Alert
```

cho phép giám sát dữ liệu thời gian thực, lưu trữ lịch sử, trực quan hóa và cảnh báo tự động khi dữ liệu vượt ngưỡng cho phép.

# II. Thực Hành

# PHẦN 1

# CHUẨN BỊ MÔI TRƯỜNG VÀ DOCKER COMPOSE

---

# 1. Mục tiêu

Xây dựng môi trường Docker Compose gồm:

* Node-RED
* MariaDB
* InfluxDB
* Grafana
* Flask API
* Nginx

Mục tiêu của phần này:

✓ Tạo cấu trúc thư mục

✓ Viết docker-compose.yml

✓ Tạo network riêng

✓ Tạo volume lưu dữ liệu

✓ Khởi động toàn bộ container

✓ Kiểm tra các service hoạt động

---

# 2. Kiến trúc hệ thống

```text
Internet
    |
    v
+-----------+
| Node-RED  |
+-----------+
      |
      +----------------+
      |                |
      v                v
+-----------+    +-----------+
| MariaDB   |    | InfluxDB  |
+-----------+    +-----------+
      |
      v
+-----------+
| Flask API |
+-----------+
      |
      v
+-----------+
|   Nginx   |
+-----------+
      |
      +-------------+
      |             |
      v             v
Realtime Data   Grafana
```

---

# 3. Tạo thư mục dự án

Tạo thư mục:

```bash
mkdir bt5-monitor
cd bt5-monitor
```

Tạo cấu trúc:

```bash
mkdir flask-api
mkdir nginx
mkdir nginx/html
mkdir nodered
mkdir mariadb
mkdir influxdb
mkdir grafana
```

---

# 4. Cấu trúc cuối cùng

```text
bt5-monitor/
│
├── docker-compose.yml
│
├── flask-api/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── nginx/
│   ├── nginx.conf
│   └── html/
│       ├── index.html
│       ├── style.css
│       └── script.js
│
├── nodered/
│
├── mariadb/
│
├── influxdb/
│
└── grafana/
```

---

# 5. Tạo file docker-compose.yml

Tại thư mục gốc:

```bash
nano docker-compose.yml
```

Nội dung:

```yaml
version: "3.9"

services:

  mariadb:
    image: mariadb:11
    container_name: mariadb
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: monitor_db
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - monitor_net

  influxdb:
    image: influxdb:2.7
    container_name: influxdb
    restart: unless-stopped
    ports:
      - "8086:8086"
    volumes:
      - influxdb_data:/var/lib/influxdb2
    networks:
      - monitor_net

  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: unless-stopped
    ports:
      - "1880:1880"
    volumes:
      - ./nodered:/data
    depends_on:
      - mariadb
      - influxdb
    networks:
      - monitor_net

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - influxdb
    networks:
      - monitor_net

  flask-api:
    build:
      context: ./flask-api
    container_name: flask-api
    restart: unless-stopped
    ports:
      - "5000:5000"
    depends_on:
      - mariadb
    networks:
      - monitor_net

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - flask-api
      - grafana
    networks:
      - monitor_net

volumes:

  mariadb_data:

  influxdb_data:

  grafana_data:

networks:

  monitor_net:
    driver: bridge
```

---

# 6. Tạo Flask API tối thiểu

## requirements.txt

```text
flask
pymysql
```

---

## app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return {
        "status":"OK",
        "message":"Flask API Running"
    }

app.run(
    host="0.0.0.0",
    port=5000
)
```

---

## Dockerfile

```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python","app.py"]
```

---

# 7. Tạo Nginx Config

File:

```bash
nano nginx/nginx.conf
```

Nội dung:

```nginx
server {

    listen 80;

    server_name localhost;

    location / {

        root /usr/share/nginx/html;
        index index.html;
    }

    location /api/ {

        proxy_pass http://flask-api:5000/;

    }

}
```

---

# 8. Tạo Frontend Test

## index.html

```html
<!DOCTYPE html>
<html>
<head>
<title>BT5 Monitor</title>
</head>

<body>

<h1>BT5 MONITOR SYSTEM</h1>

<div id="status">
Loading...
</div>

<script src="script.js"></script>

</body>
</html>
```

---

## script.js

```javascript
fetch("/api/")
.then(r=>r.json())
.then(data=>{

document.getElementById("status").innerHTML =
JSON.stringify(data);

});
```

---

## style.css

```css
body{
font-family:Arial;
padding:20px;
}
```

---

# 9. Build và khởi động

Tại thư mục dự án:

```bash
docker compose up -d --build
```

Docker sẽ:

```text
Pull MariaDB
Pull InfluxDB
Pull Grafana
Pull Node-RED
Build Flask API
Pull Nginx
```

---

# 10. Kiểm tra container

```bash
docker ps
```

Kết quả:

```text
mariadb
influxdb
nodered
grafana
flask-api
nginx
```

Tất cả phải ở trạng thái:

```text
Up
```

---

# 11. Kiểm tra Network

```bash
docker network ls
```

Sẽ thấy:

```text
bt5-monitor_monitor_net
```

---

Chi tiết:

```bash
docker network inspect bt5-monitor_monitor_net
```

Phải thấy:

```text
mariadb
influxdb
nodered
grafana
flask-api
nginx
```

---

# 12. Kiểm tra MariaDB

Truy cập:

```bash
docker exec -it mariadb bash
```

Đăng nhập:

```bash
mysql -uroot -p
```

Password:

```text
123456
```

Hiển thị database:

```sql
SHOW DATABASES;
```

Phải thấy:

```text
monitor_db
```

---

# 13. Kiểm tra Flask API

Trình duyệt:

```text
http://localhost:5000
```

Kết quả:

```json
{
  "status":"OK",
  "message":"Flask API Running"
}
```

---

# 14. Kiểm tra Nginx

Trình duyệt:

```text
http://localhost
```

Kết quả:

```text
BT5 MONITOR SYSTEM
```

---

# 15. Kiểm tra Node-RED

Mở:

```text
http://localhost:1880
```

Phải xuất hiện giao diện Node-RED.

---

# 16. Kiểm tra InfluxDB

Mở:

```text
http://localhost:8086
```

Phải xuất hiện màn hình setup InfluxDB.

---

# 17. Kiểm tra Grafana

Mở:

```text
http://localhost:3000
```

Đăng nhập:

```text
user: admin
password: admin
```

Sau đó đổi mật khẩu.

---

# 18. Kiểm tra kết nối giữa các container

Vào Node-RED:

```bash
docker exec -it nodered sh
```

Ping MariaDB:

```bash
ping mariadb
```

Ping Grafana:

```bash
ping grafana
```

Ping Flask:

```bash
ping flask-api
```

Nếu nhận phản hồi:

```text
64 bytes from ...
```

=> network hoạt động chính xác.

---

# 19. Dừng hệ thống

```bash
docker compose down
```

---

# 20. Khởi động lại

```bash
docker compose up -d
```

---

# Kết quả cần đạt

```text
✓ Docker Compose chạy thành công

✓ 6 containers hoạt động

✓ MariaDB hoạt động

✓ InfluxDB hoạt động

✓ Node-RED hoạt động

✓ Grafana hoạt động

✓ Flask API hoạt động

✓ Nginx hoạt động

✓ Các container nhìn thấy nhau qua monitor_net

✓ Sẵn sàng chuyển sang Phần 2:
  MariaDB + InfluxDB + Node-RED thu thập dữ liệu thời tiết thực tế
```

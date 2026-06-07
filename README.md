# Manguonmo_BT5
Đề đây: https://github.com/hoangquyencode/Manguonmo_BT5/blob/main/%C4%90ebai.md
## Lý thuyết

### 1. Docker là gì?

Docker là một nền tảng mã nguồn mở cho phép bạn đóng gói ứng dụng và tất cả các thành phần phụ thuộc của nó (thư viện, cấu hình, môi trường...) vào trong một đơn vị duy nhất gọi là **Container**.

**Hiểu đơn giản:** Bản code của bạn giống như đồ đạc trong nhà. Nếu vận chuyển rời rạc sang nhà mới (server mới), rất dễ bị thất lạc hoặc không vừa vặn.

Docker giúp bạn gom tất cả vào một "thùng container" tiêu chuẩn. Đi đến bất cứ đâu, chỉ cần hạ container xuống là mọi thứ hoạt động y hệt như cũ mà không lo xung đột môi trường.

### 2. Các từ khóa trong docker-compose.yml

Docker Compose là công cụ giúp bạn định nghĩa và chạy hệ thống gồm nhiều container (multi-container) chỉ bằng một file cấu hình duy nhất.

Dưới đây là các từ khóa phổ biến chia theo từng thành phần:

#### Cấu trúc tổng thể (Top-level)

* **version**: Định nghĩa phiên bản cấu hình của Docker Compose (ví dụ: `3.8`). Lưu ý: Từ các phiên bản Compose mới, từ khóa này có thể được lược bỏ nhưng vẫn nên viết để rõ ràng.

* **services**: Nơi định nghĩa các container (dịch vụ) sẽ chạy.

* **networks**: Định nghĩa các mạng ảo để các container giao tiếp với nhau.

* **volumes**: Định nghĩa nơi lưu trữ dữ liệu bền vững (không bị mất khi container bị xóa).

#### Từ khóa mô tả một Service (Container)

* **image**: Chỉ định Docker Image được dùng để tạo container (tải từ Docker Hub hoặc local).

* **build**: Dùng khi bạn muốn Docker tự build image từ Dockerfile thay vì dùng image có sẵn.

* **ports**: Ánh xạ cổng (port) từ máy host vào trong container theo cú pháp `HOST:CONTAINER`.

* **environment**: Thiết lập các biến môi trường cho container (ví dụ: user, password của database).

* **depends_on**: Thiết lập thứ tự khởi động (ví dụ: Web phải đợi DB khởi động xong).

* **restart**: Chính sách tự động khởi động lại container (ví dụ: `always`, `unless-stopped`).

#### Từ khóa mô tả Network & Volume

* **driver**: Chỉ định loại network (thường là `bridge`) hoặc loại volume để quản lý dữ liệu.

  📝 Ví dụ minh họa file docker-compose.yml hoàn chỉnh
Dưới đây là file setup một ứng dụng Web (Node.js) kết nối với một Cơ sở dữ liệu (PostgreSQL):

### Ví dụ file `docker-compose.yml`

```yaml
version: '3.8'

services:
  # 1. Định nghĩa service Database
  db-postgres:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: my_app_db
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app-network

  # 2. Định nghĩa service Ứng dụng Web
  web-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:3000"
    environment:
      - DB_HOST=db-postgres
      - NODE_ENV=production
    depends_on:
      - db-postgres
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```

### Giải thích

#### Service `db-postgres`

* **image: postgres:15-alpine**
  Sử dụng image PostgreSQL phiên bản 15 Alpine từ Docker Hub.

* **restart: always**
  Tự động khởi động lại container nếu bị dừng hoặc gặp lỗi.

* **environment**
  Khai báo các biến môi trường để cấu hình PostgreSQL:

  * `POSTGRES_USER`: tên người dùng.
  * `POSTGRES_PASSWORD`: mật khẩu.
  * `POSTGRES_DB`: tên cơ sở dữ liệu.

* **volumes**
  Gắn volume `db_data` để dữ liệu không bị mất khi container bị xóa.

* **networks**
  Kết nối container vào mạng `app-network`.

#### Service `web-app`

* **build**
  Tự động build image từ file `Dockerfile` trong thư mục hiện tại.

* **ports: "8080:3000"**
  Ánh xạ cổng 3000 trong container ra cổng 8080 của máy host.

* **environment**

  * `DB_HOST=db-postgres`: sử dụng tên service làm địa chỉ kết nối database.
  * `NODE_ENV=production`: chạy ứng dụng ở môi trường production.

* **depends_on**
  Đảm bảo container database được khởi động trước web-app.

* **networks**
  Tham gia mạng `app-network` để giao tiếp với database.

#### Top-level Resources

##### Volume

```yaml
volumes:
  db_data:
```

Tạo volume `db_data` để lưu trữ dữ liệu PostgreSQL lâu dài.

##### Network

```yaml
networks:
  app-network:
    driver: bridge
```

Tạo mạng ảo `app-network` sử dụng driver `bridge` giúp các container giao tiếp với nhau bằng tên service.

### 3. Ưu điểm khi triển khai ứng dụng sử dụng Docker

#### Tính nhất quán (Consistency)

Docker loại bỏ hoàn toàn lỗi **"Chạy được trên máy em nhưng lỗi trên server"**. Môi trường Development, Testing và Production được đóng gói giống nhau nên ứng dụng hoạt động nhất quán trên mọi hệ thống.

#### Tiết kiệm tài nguyên

Khác với máy ảo (Virtual Machine - VM) phải cài đặt một hệ điều hành riêng, Docker Container chia sẻ chung nhân hệ điều hành của máy chủ nên nhẹ hơn rất nhiều, sử dụng ít CPU, RAM và dung lượng lưu trữ hơn.

#### Dễ dàng mở rộng (Scalability)

Khi ứng dụng có nhiều người truy cập, Docker cho phép triển khai nhiều container giống nhau để chia tải một cách nhanh chóng chỉ với vài câu lệnh cấu hình.

#### Tách biệt và an toàn (Isolation)

Mỗi container hoạt động độc lập với các container khác. Nếu một container gặp lỗi hoặc bị tấn công, ảnh hưởng đến toàn bộ hệ thống sẽ được giảm thiểu đáng kể.

#### Quản lý phiên bản dễ dàng

Docker Image được quản lý bằng các tag phiên bản như `v1.0`, `v1.1`, `latest`,... giúp việc cập nhật hoặc quay lại phiên bản cũ (rollback) trở nên nhanh chóng và thuận tiện.

---

### 4. Quy trình triển khai ứng dụng lên Server không có Internet (Offline)

Trong một số môi trường yêu cầu bảo mật cao như ngân hàng, cơ quan chính phủ hoặc nhà máy sản xuất, máy chủ không được kết nối Internet. Khi đó việc triển khai Docker cần thực hiện theo quy trình ngoại tuyến (Offline Deployment).

#### Bước 1: Chuẩn bị bộ cài Docker

Trên máy tính có Internet, tải trước các thành phần cần thiết:

* Docker Engine.
* Docker Compose.
* Các gói cài đặt `.deb` (Ubuntu) hoặc `.rpm` (CentOS/RHEL).
* Hoặc bộ Docker Static Binary từ trang chủ Docker.

#### Bước 2: Đóng gói Docker Image

Sau khi xây dựng và kiểm thử ứng dụng thành công trên máy cá nhân, xuất các Docker Image thành file `.tar`.

```bash id="xrw1ux"
# Xuất image ứng dụng
docker save -o web-app-v1.tar web-app:latest

# Xuất image PostgreSQL
docker save -o postgres-15.tar postgres:15-alpine
```

#### Bước 3: Chuyển dữ liệu sang Server

Sao chép các tệp cần thiết sang máy chủ:

* Bộ cài Docker.
* Các file Image `.tar`.
* Mã nguồn ứng dụng.
* File `docker-compose.yml`.

Việc sao chép có thể thực hiện bằng USB, ổ cứng di động hoặc mạng LAN nội bộ.

#### Bước 4: Cài đặt và khởi chạy ứng dụng

##### 4.1 Cài đặt Docker

Ví dụ trên Ubuntu:

```bash id="8t8gql"
dpkg -i *.deb
```

##### 4.2 Nạp Docker Image vào Server

```bash id="0b1w5y"
docker load -i web-app-v1.tar
docker load -i postgres-15.tar
```

Kiểm tra lại các Image đã được nạp thành công:

```bash id="3mt4if"
docker images
```

##### 4.3 Khởi chạy ứng dụng

Di chuyển đến thư mục chứa file `docker-compose.yml` và thực hiện:

```bash id="6v1d4w"
docker compose up -d
```



Sau khi hoàn tất, Docker sẽ tạo và chạy các container từ các Image đã được nạp sẵn mà không cần kết nối Internet.


## Thực hành

Hệ sinh thái



```text
                          (Ghi giá trị tức thời)
┌───────────┐ ────────────────────────────────► ┌───────────┐ ◄──── ┌───────────┐
│ Node-RED  │                                   │  MariaDB  │       │ Flask API │
└─────┬─────┘                                   └───────────┘       └─────┬─────┘
      │                                                    ▲               │
      │ (Ghi dữ liệu chuỗi thời gian)                      │               │ (Truy vấn dữ liệu)
      ▼                                                    │               ▼

┌───────────┐                                    ┌─────────────────────────────┐
│ InfluxDB  │                                    │        Nginx Frontend       │
└─────┬─────┘                                    │      (html/index.html)      │
      │                                           └──────────────┬─────────────┘
      │ (Cung cấp dữ liệu cho Grafana)                           │
      ▼                                                          │ (Nhúng Dashboard)
┌───────────┐                                                    │
│  Grafana  │ ───────────────────────────────────────────────────┘
└───────────┘
```

# 1. Tạo cấu trúc thư mục dự án
mkdir -p monitor-app/flask-api monitor-app/nginx-frontend/html
cd monitor-app

# 2. Tạo nhanh file docker-compose.yml
cat << 'EOF' > docker-compose.yml
version: '3.8'
services:
  nodered:
    image: nodered/node-red:latest
    container_name: nodered_service
    ports:
      - "1880:1880"
    volumes:
      - nodered_data:/data
    networks:
      - monitor-net
    restart: unless-stopped

  mariadb:
    image: mariadb:latest
    container_name: mariadb_service
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: monitor_db
      MYSQL_USER: app_user
      MYSQL_PASSWORD: app_password
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - monitor-net
    restart: unless-stopped

  influxdb:
    image: influxdb:2.7-alpine
    container_name: influxdb_service
    ports:
      - "8086:8086"
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=adminpassword
      - DOCKER_INFLUXDB_INIT_ORG=myorg
      - DOCKER_INFLUXDB_INIT_BUCKET=history_bucket
    volumes:
      - influxdb_data:/var/lib/influxdb2
    networks:
      - monitor-net
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana_service
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitor-net
    restart: unless-stopped

  flask-api:
    build: ./flask-api
    container_name: flask_api_service
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=mariadb
      - DB_USER=app_user
      - DB_PASSWORD=app_password
      - DB_NAME=monitor_db
    depends_on:
      - mariadb
    networks:
      - monitor-net
    restart: unless-stopped

  nginx-frontend:
    image: nginx:alpine
    container_name: nginx_service
    ports:
      - "80:80"
    volumes:
      - ./nginx-frontend/html:/usr/share/nginx/html
      - ./nginx-frontend/default.conf:/etc/nginx/conf.d/default.conf
    networks:
      - monitor-net
    restart: unless-stopped

volumes:
  nodered_data:
  mariadb_data:
  influxdb_data:
  grafana_data:

networks:
  monitor-net:
    driver: bridge
EOF

# 3. Tạo các file cho Flask API
cat << 'EOF' > flask-api/requirements.txt
Flask==3.0.0
mysql-connector-python==8.2.0
Flask-CORS==4.0.0
EOF

cat << 'EOF' > flask-api/Dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
EOF

cat << 'EOF' > flask-api/app.py
import os
from flask import Flask, jsonify
from flask_cors import CORS
import mysql.connector

app = Flask(__name__)
CORS(app)

def get_db_connection():
    return mysql.connector.connect(
        host=os.environ.get('DB_HOST', 'localhost'),
        user=os.environ.get('DB_USER', 'app_user'),
        password=os.environ.get('DB_PASSWORD', 'app_password'),
        database=os.environ.get('DB_NAME', 'monitor_db')
    )

conn = get_db_connection()
cursor = conn.cursor()
cursor.execute("""
    CREATE TABLE IF NOT EXISTS realtime_data (
        id INT AUTO_INCREMENT PRIMARY KEY,
        device_name VARCHAR(50),
        val_value FLOAT,
        timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
""")
conn.close()

@app.route('/api/live', methods=['GET'])
def get_live_data():
    try:
        conn = get_db_connection()
        cursor = conn.cursor(dictionary=True)
        cursor.execute("SELECT val_value, timestamp FROM realtime_data ORDER BY id DESC LIMIT 1")
        result = cursor.fetchone()
        cursor.close()
        conn.close()
        if result:
            return jsonify({"status": "success", "data": result})
        return jsonify({"status": "empty", "data": {"val_value": 0, "timestamp": "-"}})
    except Exception as e:
        return jsonify({"status": "error", "message": str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# 4. Tạo các file cho Nginx Frontend
cat << 'EOF' > nginx-frontend/default.conf
server {
    listen 80;
    server_name localhost;
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
EOF

cat << 'EOF' > nginx-frontend/html/index.html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Hệ thống Giám sát Realtime</title>
    <style>
        body { font-family: Arial, sans-serif; background: #f4f6f9; text-align: center; }
        .container { max-width: 1000px; margin: 0 auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .card { background: #2c3e50; color: white; padding: 20px; display: inline-block; border-radius: 8px; font-size: 24px; margin-bottom: 20px; }
        .value { font-size: 48px; font-weight: bold; color: #2ecc71; }
        iframe { width: 100%; height: 450px; border: none; border-radius: 8px; margin-top: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <h2>HỆ THỐNG GIÁM SÁT REALTIME</h2>
        <div class="card">
            <div>Giá trị tức thời (MariaDB):</div>
            <div id="live-value" class="value">--</div>
            <div style="font-size: 14px; color: #bdc3c7;">Cập nhật lúc: <span id="live-time">-</span></div>
        </div>
        <h3>Đồ thị lịch sử (Grafana Iframe)</h3>
        <iframe id="grafana-frame" src="http://192.168.100.10:3000/d-solo/your-dashboard-id/history?orgId=1&panelId=1&refresh=5s"></iframe>
    </div>
    <script>
        function fetchLive_Data() {
            fetch('http://192.168.100.10:5000/api/live')
                .then(response => response.json())
                .then(res => {
                    if(res.status === 'success') {
                        document.getElementById('live-value').innerText = res.data.val_value;
                        document.getElementById('live-time').innerText = new Date(res.data.timestamp).toLocaleTimeString();
                    }
                })
                .catch(err => console.error("Lỗi gọi API: ", err));
        }
        fetchLive_Data();
        setInterval(fetchLive_Data, 2000);
    </script>
</body>
</html>
EOF


# Node-Red







<img width="1915" height="955" alt="image" src="https://github.com/user-attachments/assets/bc24acf0-8d40-47b1-be5f-3d8082516127" />






# Grafana





<img width="1903" height="967" alt="image" src="https://github.com/user-attachments/assets/1d194b17-5cb5-4cb2-a2b3-1b1bef35ad51" />









<img width="1918" height="988" alt="image" src="https://github.com/user-attachments/assets/2e10f1d0-0f46-4218-9884-3f09e82b69a0" />











<img width="1755" height="1067" alt="image" src="https://github.com/user-attachments/assets/183d4c58-c514-4c43-a4e5-578c607eca46" />








<img width="1113" height="198" alt="image" src="https://github.com/user-attachments/assets/eb78ce3e-7da9-472c-a7d1-96f7a08fcf7e" />







Cấp quyền Admin cho Bot, chu kỳ lấy dữ liệu tiếp theo (khi giá Bitcoin > 60000) sẽ lập tức kích hoạt luồng.






<img width="550" height="566" alt="image" src="https://github.com/user-attachments/assets/9a8a40ab-8c5c-4e73-b816-d313c045f667" />







### Sao lưu (Backup) và Phục hồi (Restore) ứng dụng Docker Compose

#### 1. Dừng hệ thống
docker compose down

#### 2. Nén toàn bộ thư mục bài Lab thành 1 file duy nhất
tar -czvf baiso2_backup.tar.gz ./thu_muc_bai_lab/

#### 3. Xóa sạch mọi container và dữ liệu trên máy để test khôi phục
docker system prune -a --volumes -f

#### 4. Giải nén để khôi phục
tar -xzvf baiso2_backup.tar.gz

#### 5. Chạy lại
docker compose up -d







<img width="1322" height="914" alt="image" src="https://github.com/user-attachments/assets/833a723f-ccab-4485-b509-33de4587d435" />













<img width="1431" height="960" alt="image" src="https://github.com/user-attachments/assets/6fd4b97a-c404-4fab-99a7-6dc11f3ee161" />














<img width="1436" height="997" alt="image" src="https://github.com/user-attachments/assets/95638fac-2296-4f89-9a2e-6b869b616dab" />

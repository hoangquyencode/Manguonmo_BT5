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



# 🏦 Quản lý Tiệm Cầm Đồ

> Ứng dụng web quản lý nghiệp vụ tiệm cầm đồ, xây dựng trên **Django (Python)**,
> triển khai hoàn toàn bằng **Docker Compose**, public ra Internet qua **Cloudflare Tunnel**.

---

## 1. Tính năng nổi bật

- 📋 Quản lý **Khách hàng · Hợp đồng · Tài sản · Lịch sử thanh toán**
- ⚙️ Trang `/admin` tự sinh bởi Django — thêm / sửa / xoá mọi bảng, khoá ngoại hiển thị **dropdown** thay vì nhập ID thủ công
- 🔴 Trang chủ `/` tự động liệt kê **hợp đồng đến hạn chưa chuộc** (template Jinja2 + context từ view)
- 🔍 Kiểm chứng FK lưu ID số nguyên trực tiếp qua **phpMyAdmin**
- 🌐 Truy cập từ Internet qua subdomain Cloudflare — không cần mở port, không cần IP tĩnh

---

## 2. Công nghệ sử dụng

| Thành phần | Chi tiết |
|---|---|
| Backend | Django 4.2 · Python 3.11 |
| Database | MariaDB 10.11 · mysqlclient driver |
| Container | Docker Compose · Dockerfile tự build |
| Tunnel | Cloudflare Zero Trust |
| Editor | VS Code Remote SSH · `sudo nano` |

---

## 3. Kiến trúc triển khai

```
[Browser] ──HTTPS──▶ [Cloudflare] ──tunnel──▶ [cloudflared]
                                                     │
                                                     ▼
                                              [Django :8000]
                                                     │ ORM
                                                     ▼
                                           [MariaDB :3306] ◀── [phpMyAdmin :8088]
```

| Container | Vai trò | Port |
|---|---|---|
| `camdo_django` | Ứng dụng web chính | 8000 |
| `camdo_mariadb` | Cơ sở dữ liệu | 3306 |
| `camdo_phpmyadmin` | Giao diện kiểm tra CSDL | 8088 |
| `camdo_cloudflared` | Tunnel ra Internet | — |

---

## 4. Cấu trúc dự án

```
django-camdo/
├── docker-compose.yml
├── .env
├── README.md
└── django/
    ├── Dockerfile
    ├── requirements.txt
    └── web/
        ├── manage.py
        ├── config/
        │   ├── settings.py
        │   └── urls.py
        └── core/
            ├── models.py
            ├── admin.py
            ├── views.py
            ├── urls.py
            ├── migrations/
            └── template/
                └── home.html
```

---

## 5. Thiết kế CSDL

<img src="https://github.com/user-attachments/assets/ca257635-531a-4fd3-a177-aa992dbcdfc0"/>

### 5.1 Mô tả các bảng

<details>
<summary>KhachHang</summary>

| Trường | Kiểu | Ghi chú |
|---|---|---|
| id | bigint | PK, auto increment |
| ho_ten | varchar(100) | |
| cmnd | varchar(20) | unique |
| so_dien_thoai | varchar(15) | |
| dia_chi | text | |

</details>

<details>
<summary>HopDong</summary>

| Trường | Kiểu | Ghi chú |
|---|---|---|
| id | bigint | PK |
| ma_hop_dong | varchar(20) | unique |
| khach_hang_id | bigint | **FK → KhachHang** |
| nhan_vien_lap | varchar(100) | |
| ngay_cam | date | |
| ngay_dao_han | date | |
| so_tien_vay | decimal(15,0) | VNĐ |
| lai_suat | decimal(5,2) | %/tháng |
| trang_thai | varchar(20) | `dang_cam` / `da_chuoc` / `qua_han` |
| ghi_chu | text | |

</details>

<details>
<summary>TaiSan</summary>

| Trường | Kiểu | Ghi chú |
|---|---|---|
| id | bigint | PK |
| hop_dong_id | bigint | **FK → HopDong** |
| ten_tai_san | varchar(200) | |
| danh_muc | varchar(20) | `vang` / `dien_tu` / `xe` / `do_dung` / `khac` |
| mo_ta | text | |
| gia_dinh_gia | decimal(15,0) | VNĐ |
| hinh_anh | varchar(100) | đường dẫn file |

</details>

<details>
<summary>LichSuTT</summary>

| Trường | Kiểu | Ghi chú |
|---|---|---|
| id | bigint | PK |
| hop_dong_id | bigint | **FK → HopDong** |
| ngay_tt | date | |
| so_tien | decimal(15,0) | VNĐ |
| loai | varchar(20) | `thanh_toan` / `gia_han` / `phat_lai` |
| ghi_chu | text | |

</details>

### 5.2 Quan hệ khoá ngoại

| Bảng con | Cột FK | Tham chiếu | Quan hệ |
|---|---|---|---|
| `HopDong` | `khach_hang_id` | `KhachHang.id` | 1 → n |
| `TaiSan` | `hop_dong_id` | `HopDong.id` | 1 → n |
| `LichSuTT` | `hop_dong_id` | `HopDong.id` | 1 → n |

> Django lưu **ID số nguyên** vào cột FK, hiển thị text trên form —
> kiểm chứng bằng phpMyAdmin tại `:8088`

---

## 6. Hướng dẫn cài đặt

### 6.1 Chạy migration & tạo admin

**Bước 1 — Tạo migration từ models.py**
```bash
docker compose exec django python manage.py makemigrations core
```
<img src="https://github.com/user-attachments/assets/9438e544-b5af-433a-a1cb-643d6b9d8ae9"/>

**Bước 2 — Áp migration vào database**
```bash
docker compose exec django python manage.py migrate
```
<img src="https://github.com/user-attachments/assets/c69101a3-ddbc-41f1-b9ab-cd4990651060"/>

**Bước 3 — Tạo tài khoản admin**
```bash
docker compose exec django python manage.py createsuperuser
```
```
Username: admin
Email address:        ← Enter bỏ qua
Password: ****
Password (again): ****
```
<img src="https://github.com/user-attachments/assets/d0c7c6d6-1953-4ff2-85a9-ccfd78bcfc37"/>

**Bước 4 — Truy cập**

| URL | Mô tả |
|---|---|
| `http://192.168.126.131:8000` | Trang con nợ đến hạn |
| `http://192.168.126.131:8000/admin/` | Trang quản trị Django |
| `http://192.168.126.131:8088` | phpMyAdmin |

<img src="https://github.com/user-attachments/assets/9fbc4755-b0a5-434e-9afe-f65ab863ffb5"/>
<img src="https://github.com/user-attachments/assets/03110193-0e9d-4261-93f9-840afb4a3f37"/>

---

### 6.2 Cấu hình Cloudflare Tunnel

**Bước 1 — Thêm service vào `docker-compose.yml`**
```yaml
cloudflared:
  image: cloudflare/cloudflared:latest
  container_name: camdo_cloudflared
  command: tunnel --no-autoupdate run --token ${CLOUDFLARE_TOKEN}
  restart: unless-stopped
  depends_on:
    - django
  networks:
    - camdo_net
```

**Bước 2 — Thêm token vào `.env`**
```
CLOUDFLARE_TOKEN=eyJh...token_của_bạn
```

**Bước 3 — Lấy token trên Cloudflare**

1. Vào [one.dash.cloudflare.com](https://one.dash.cloudflare.com) → **Networks → Tunnels**
2. Chọn **Create a Tunnel → Docker**
3. Copy token từ lệnh hiển thị → paste vào `.env`

<img src="https://github.com/user-attachments/assets/c66e5220-21e1-4cdc-b4c8-f5b12cc45536"/>
<img src="https://github.com/user-attachments/assets/c170f7b0-9927-4f1e-9a14-12030517f36d"/>

**Bước 4 — Thêm Public Hostname**

Chọn **Add a public hostname** → trỏ về `http://django:8000`

<img src="https://github.com/user-attachments/assets/3de14515-4ada-41eb-96ec-c5489ef2b362"/>

---

## 7. Kết quả

### 7.1 Trang Admin Django
<img src="https://github.com/user-attachments/assets/6796baf9-f525-4364-b4ca-8dc374b4cc1a"/>

### 7.2 Trang con nợ đến hạn
<img src="https://github.com/user-attachments/assets/f0a7792a-089b-4951-9b2c-276498e5923d"/>

### 7.3 Trang con nợ sau khi thêm đủ dữ liệu
<img src="https://github.com/user-attachments/assets/9a8e5fcd-beee-49b9-853a-b40a7c14163f"/>

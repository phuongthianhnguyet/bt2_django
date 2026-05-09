
# 1. Giới thiệu:
# 2. Thiết kế CSDL:

<img width="2562" height="1740" alt="image" src="https://github.com/user-attachments/assets/ca257635-531a-4fd3-a177-aa992dbcdfc0" />

## Giải thích
- Một hợp đồng có nhiều lịch sử thanh toán
- Một hợp đồng có nhiều tài sản
- Một khách có nhiều hợp đồng

# 3. Mô tả chi tiết từng bảng
#### KhachHang
| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| id | bigint | PK, auto increment |
| ho_ten | varchar(100) | Họ tên khách |
| cmnd | varchar(20) | CMND/CCCD, unique |
| so_dien_thoai | varchar(15) | |
| dia_chi | text | |

#### HopDong
| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| id | bigint | PK |
| ma_hop_dong | varchar(20) | unique |
| khach_hang_id | bigint | FK → KhachHang |
| nhan_vien_lap | varchar(100) | |
| ngay_cam | date | |
| ngay_dao_han | date | |
| so_tien_vay | decimal(15,0) | VNĐ |
| lai_suat | decimal(5,2) | %/tháng |
| trang_thai | varchar(20) | dang_cam / da_chuoc / qua_han |
| ghi_chu | text | |

#### TaiSan
| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| id | bigint | PK |
| hop_dong_id | bigint | FK → HopDong |
| ten_tai_san | varchar(200) | |
| danh_muc | varchar(20) | vang / dien_tu / xe / do_dung / khac |
| mo_ta | text | |
| gia_dinh_gia | decimal(15,0) | VNĐ |
| hinh_anh | varchar(100) | đường dẫn file |

#### LichSuTT
| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| id | bigint | PK |
| hop_dong_id | bigint | FK → HopDong |
| ngay_tt | date | |
| so_tien | decimal(15,0) | VNĐ |
| loai | varchar(20) | thanh_toan / gia_han / phat_lai |
| ghi_chu | text | |

### Quan hệ
- **KhachHang (1 : n) → HopDong**: một khách có nhiều hợp đồng
- **HopDong (1 : n) → TaiSan**: một hợp đồng có nhiều tài sản
- **HopDong (1 : n) → LichSuTT**: một hợp đồng có nhiều lịch sử thanh toán
  
## Quan hệ khoá ngoại

| Bảng con | Cột FK | Tham chiếu đến | Kiểu quan hệ |
|----------|--------|----------------|--------------|
| `HopDong` | `khach_hang_id` | `KhachHang.id` | 1 KhachHang → n HopDong |
| `TaiSan` | `hop_dong_id` | `HopDong.id` | 1 HopDong → n TaiSan |
| `LichSuTT` | `hop_dong_id` | `HopDong.id` | 1 HopDong → n LichSuTT |

> Khi thêm dữ liệu vào bảng con (HopDong, TaiSan, LichSuTT),
> Django tự động lưu ID số nguyên vào cột FK thay vì lưu text —
> có thể kiểm chứng bằng phpMyAdmin.
# 4. Cấu trúc dự án Django Cầm Đồ

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
        ├── staticfiles/
        ├── config/
        │   ├── __init__.py
        │   ├── asgi.py
        │   ├── settings.py
        │   ├── urls.py
        │   └── wsgi.py
        └── core/
            ├── __init__.py
            ├── admin.py
            ├── apps.py
            ├── models.py
            ├── tests.py
            ├── urls.py
            ├── views.py
            ├── migrations/
            └── template/
                └── home.html
```
# 5. Hướng dẫn cài đặt
Chạy lần lượt từng lệnh:  
Bước 1 — Tạo migration:  
```bash
docker compose exec django python manage.py makemigrations core  
```
<img width="598" height="116" alt="image" src="https://github.com/user-attachments/assets/9438e544-b5af-433a-a1cb-643d6b9d8ae9" />

Bước 2 — Áp vào database:  
```bash
docker compose exec django python manage.py migrate
``` 
<img width="544" height="95" alt="image" src="https://github.com/user-attachments/assets/c69101a3-ddbc-41f1-b9ab-cd4990651060" />

Bước 3 — Tạo tài khoản admin:  
```bash
docker compose exec django python manage.py createsuperuser
```

Nó sẽ hỏi lần lượt: 
```
Username: admin  
Email address: (enter để bỏ qua)
Password: ****
Password (again): ****
```
<img width="555" height="131" alt="image" src="https://github.com/user-attachments/assets/d0c7c6d6-1953-4ff2-85a9-ccfd78bcfc37" />

Bước 4 — Kiểm tra kết quả:
Mở trình duyệt vào:
```bash
http://192.168.126.131:8000 → trang con nợ
http://192.168.126.131:8000/admin/ → trang quản trị
```
<img width="950" height="434" alt="image" src="https://github.com/user-attachments/assets/9fbc4755-b0a5-434e-9afe-f65ab863ffb5" />

<img width="943" height="472" alt="image" src="https://github.com/user-attachments/assets/03110193-0e9d-4261-93f9-840afb4a3f37" />
  
# Cloudflared
Mở file:
```
docker-compose.yml
```
Thêm đoạn sau vào phần services::
```yml
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
## Thêm Token Cloudflare
Mở file: `.env`
Thêm:
```
CLOUDFLARE_TOKEN=eyJggoi...
```
## Lấy Tunnel Token Trên Cloudflare
Bước 1: Truy cập Cloudflare 
Bước 2: Tạo Tunnel
Chọn:
Create a Tunnel

<img width="957" height="485" alt="image" src="https://github.com/user-attachments/assets/c66e5220-21e1-4cdc-b4c8-f5b12cc45536" />

Bước 3: Chọn Docker
Cloudflare sẽ hiện lệnh dạng:

docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token xxxxx

<img width="945" height="474" alt="image" src="https://github.com/user-attachments/assets/c170f7b0-9927-4f1e-9a14-12030517f36d" />

## Tạo Public Hostname
Add a public hostname
<img width="944" height="482" alt="image" src="https://github.com/user-attachments/assets/3de14515-4ada-41eb-96ec-c5489ef2b362" />

# Kết Quả
## Trang Admin Django
<img width="956" height="511" alt="image" src="https://github.com/user-attachments/assets/6796baf9-f525-4364-b4ca-8dc374b4cc1a" />

## Trang Con Nợ Đến Hạn
<img width="952" height="474" alt="image" src="https://github.com/user-attachments/assets/f0a7792a-089b-4951-9b2c-276498e5923d" />
## Trang con nợ đến hạn khi đã cho thêm khách hàng.
<img width="1905" height="949" alt="image" src="https://github.com/user-attachments/assets/9a8e5fcd-beee-49b9-853a-b40a7c14163f" />


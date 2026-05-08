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


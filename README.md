# Session 10 - Bài 2: Đóng gói Web Backend bằng Single-stage Dockerfile

**Họ tên:** Nguyễn Thế Kiên | **Lớp:** K24 - DevOps PTIT

## Mục tiêu
Viết Dockerfile single-stage đóng gói ứng dụng REST API Node.js/Express thành image `hw-backend:v1`, chạy container `app-single-stage` với port `8080:3000`.

## Cấu trúc thư mục
```
ex2/
├── package.json
├── server.js
├── Dockerfile
├── .dockerignore
└── README.md
```

## Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

ENV PORT=3000

COPY package*.json ./
RUN npm install --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

| Chỉ thị | Vai trò |
|---|---|
| `FROM node:18-alpine` | Base image Node.js 18 bản Alpine nhẹ |
| `WORKDIR /app` | Thư mục làm việc trong container |
| `ENV PORT=3000` | Biến môi trường cổng cho ứng dụng |
| `COPY package*.json ./` | Chép file khai báo thư viện trước để tận dụng layer cache |
| `RUN npm install --omit=dev` | Cài thư viện (express) lúc build image |
| `COPY . .` | Chép mã nguồn vào image |
| `EXPOSE 3000` | Khai báo cổng container lắng nghe |
| `CMD ["node", "server.js"]` | Lệnh chạy app, dạng exec form để `node` là PID 1 và nhận được SIGTERM |

`.dockerignore` loại `node_modules` khỏi build context để image không mang theo thư viện từ máy host.

---

## Các lệnh đã gõ

### 1. Build image
```bash
docker build -t hw-backend:v1 .
```
```
[+] Building 12.8s (10/10) FINISHED                                                                                                                    docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                   0.1s
 => => transferring dockerfile: 188B                                                                                                                                   0.0s 
 => [internal] load metadata for docker.io/library/node:18-alpine                                                                                                      3.4s 
 => [internal] load .dockerignore                                                                                                                                      0.0s
 => => transferring context: 67B                                                                                                                                       0.0s 
 => [1/5] FROM docker.io/library/node:18-alpine@sha256:8d6421d663b4c28fd3ebc498332f249011d118945588d0a35cb9bc4b8ca09d9e                                                5.1s 
 => => resolve docker.io/library/node:18-alpine@sha256:8d6421d663b4c28fd3ebc498332f249011d118945588d0a35cb9bc4b8ca09d9e                                                0.0s
 => => sha256:d84c815451acbca96b6e6bdb479929222bec57121dfe10cc5b128c5c2dbaf10a 39.66MB / 39.66MB                                                                       3.0s 
 => => sha256:02bb84e9f3412827f177bc6c020812249b32a8425d2c1858e9d71bd4c015f031 443B / 443B                                                                             0.3s 
 => => sha256:8bfa36aa66ce614f6da68a16fb71f875da8d623310f0cb80ae1ecfa092f587f6 1.26MB / 1.26MB                                                                         3.2s 
 => => sha256:6e771e15690e2fabf2332d3a3b744495411d6e0b00b2aea64419b58b0066cf81 3.99MB / 3.99MB                                                                         4.2s 
 => => extracting sha256:6e771e15690e2fabf2332d3a3b744495411d6e0b00b2aea64419b58b0066cf81                                                                              0.1s 
 => => extracting sha256:d84c815451acbca96b6e6bdb479929222bec57121dfe10cc5b128c5c2dbaf10a                                                                              0.7s 
 => => extracting sha256:8bfa36aa66ce614f6da68a16fb71f875da8d623310f0cb80ae1ecfa092f587f6                                                                              0.0s 
 => => extracting sha256:02bb84e9f3412827f177bc6c020812249b32a8425d2c1858e9d71bd4c015f031                                                                              0.0s 
 => [internal] load build context                                                                                                                                      0.0s 
 => => transferring context: 779B                                                                                                                                      0.0s 
 => [2/5] WORKDIR /app                                                                                                                                                 0.2s 
 => [3/5] COPY package*.json ./                                                                                                                                        0.0s 
 => [4/5] RUN npm install --omit=dev                                                                                                                                   3.5s 
 => [5/5] COPY . .                                                                                                                                                     0.0s 
 => exporting to image                                                                                                                                                 0.4s 
 => => exporting layers                                                                                                                                                0.2s 
 => => exporting manifest sha256:0aa4cda9ff62131bf2ef98203c901536e55a8cc2f3b25c9e0c2bf6162d2a1868                                                                      0.0s 
 => => exporting config sha256:b5e4de1d38db2f64d63fd8bfde55ad9a6d137885af0cb309bede16774259d606                                                                        0.0s 
 => => exporting attestation manifest sha256:e896ba8a20798dc7df397e26fb2be2bc93f573ae4a65bfe34c71df66ece20a92                                                          0.0s 
 => => exporting manifest list sha256:979a697035c6572d309d72e2910d59993943f4fdd00bb06089bdce14da4d4c8f                                                                 0.0s 
 => => naming to docker.io/library/hw-backend:v1                                                                                                                       0.0s 
 => => unpacking to docker.io/library/hw-backend:v1   
 ```

### 2. Chạy container
```bash
docker run -d --name app-single-stage -p 8080:3000 hw-backend:v1
```
```
3dc8a6f627b757d80f0786299951fdb02bccc6c688355a793b7a185ad5a6f696
```

### 3. Kiểm tra trạng thái
```bash
docker ps
```
```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS                PORTS                                         NAMES
3dc8a6f627b7   hw-backend:v1         "docker-entrypoint.s…"   15 seconds ago   Up 15 seconds         0.0.0.0:8080->3000/tcp, [::]:8080->3000/tcp   app-single-stage
3e74f68a6229   98a8488af30d          "/docker-entrypoint.…"   12 hours ago     Up 12 hours           0.0.0.0:8081->80/tcp, [::]:8081->80/tcp       html-app
2bf9cb40c995   langfuse/langfuse:2   "dumb-init -- ./web/…"   2 weeks ago      Up 2 days             0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   ptit_final_ai_integrated_007-langfuse-server-1
e0a7fd682b64   postgres:16-alpine    "docker-entrypoint.s…"   2 weeks ago      Up 2 days (healthy)   0.0.0.0:5433->5432/tcp, [::]:5433->5432/tcp   ptit_final_ai_integrated_007-langfuse-db-1
```

### 4. Xem log
```bash
docker logs app-single-stage
```
```
Server listening on port 3000
```

### 5. Gọi API
```bash
curl -i http://localhost:8080/
```
```
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 64
ETag: W/"40-upZb7Fps+hszzRU9BoKtiCsmGCQ"
Date: Mon, 12 Oct 2026 00:46:46 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"status":"success","message":"Hello from Docker Single-stage!"}%
```

---

## Kết quả
- Build hoàn tất không lỗi, container `app-single-stage` ở trạng thái **Up**.
- `curl` trả về **HTTP/1.1 200 OK** với JSON:
```json
{"status":"success","message":"Hello from Docker Single-stage!"}
```

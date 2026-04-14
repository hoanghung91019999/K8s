# QUY TRÌNH DEPLOY ỨNG DỤNG FLUTTER WEB LÊN KUBERNETES
## Từ Source Code đến Production — Hướng dẫn A-Z

| Thông tin | Chi tiết |
|---|---|
| Ứng dụng | MyPoint Flutter Web App v2.01.01 |
| Công ty | PayTech Việt Nam |
| Người thực hiện | hunghv |
| Ngày tạo tài liệu | 14/04/2026 |
| Phiên bản tài liệu | 1.0 |

---

## MỤC LỤC

1. [Tổng quan hệ thống](#1-tổng-quan-hệ-thống)
2. [Hạ tầng](#2-hạ-tầng)
3. [Giai đoạn 1 — Build Docker Image](#3-giai-đoạn-1--build-docker-image)
4. [Giai đoạn 2 — Push lên Harbor Registry](#4-giai-đoạn-2--push-lên-harbor-registry)
5. [Giai đoạn 3 — Cấu hình K8s Nodes](#5-giai-đoạn-3--cấu-hình-k8s-nodes)
6. [Giai đoạn 4 — Deploy lên Kubernetes](#6-giai-đoạn-4--deploy-lên-kubernetes)
7. [Giai đoạn 5 — Kiểm tra & Vận hành](#7-giai-đoạn-5--kiểm-tra--vận-hành)
8. [Troubleshoot](#8-troubleshoot)
9. [Phụ lục — File cấu hình đầy đủ](#9-phụ-lục--file-cấu-hình-đầy-đủ)

---

## 1. TỔNG QUAN HỆ THỐNG

### 1.1. Ứng dụng là gì?

MyPoint là ứng dụng tích điểm / loyalty, được xây dựng bằng Flutter Web. Build output là 100% static files (HTML + JavaScript + WebAssembly + Assets), không yêu cầu server-side runtime. Chỉ cần web server Nginx để serve.

### 1.2. Kiến trúc triển khai

```
┌──────────────────┐       ┌─────────────────────────────────────┐
│  Build Server     │ push  │          Harbor Registry             │
│  ai-agent-noibo   │──────▶│  192.168.200.91                     │
│  (Docker build)   │       │  Project: mypointwebihanoi          │
└──────────────────┘       └──────────────┬──────────────────────┘
                                          │ pull image
                           ┌──────────────▼──────────────────────┐
                           │      Kubernetes Cluster              │
                           │                                      │
                           │  master-01  (.80)  control-plane     │
                           │  worker-01  (.81)  ── pod replica 1  │
                           │  worker-02  (.82)  ── pod replica 2  │
                           │                                      │
                           │  Service NodePort :30080             │
                           └──────────────────────────────────────┘
```

### 1.3. Thành phần trong container

```
Container (nginx:1.27-alpine)
├── /usr/share/nginx/html/     ← Flutter Web build output
│   ├── index.html             ← Entry point (SPA)
│   ├── main.dart.js           ← App logic (3.5MB)
│   ├── flutter_bootstrap.js   ← Flutter loader
│   ├── canvaskit/             ← WASM rendering engine (26MB)
│   └── assets/                ← Images, fonts, data
├── /etc/nginx/nginx.conf      ← Custom Nginx config
└── Port: 8080                 ← Non-root user (UID 1001)
```

---

## 2. HẠ TẦNG

### 2.1. Danh sách server

| Server | IP | Vai trò | OS |
|---|---|---|---|
| ai-agent-noibo | (LAN) | Build server, Docker | Ubuntu |
| master-01 | 192.168.200.80 | K8s control-plane | Ubuntu |
| worker-01 | 192.168.200.81 | K8s worker node | Ubuntu |
| worker-02 | 192.168.200.82 | K8s worker node | Ubuntu |
| Harbor | 192.168.200.91 | Container registry (HTTP) | — |

### 2.2. Phiên bản phần mềm

| Component | Version |
|---|---|
| Kubernetes | v1.30.14 |
| containerd | v2.2.2 (config version 3) |
| Nginx (in container) | 1.27-alpine |
| Harbor | (HTTP, no TLS) |
| CNI | Calico (IPIP/tunl0) |

---

## 3. GIAI ĐOẠN 1 — BUILD DOCKER IMAGE

> Thực hiện trên: **ai-agent-noibo**

### 3.1. Chuẩn bị thư mục build

```bash
mkdir -p /tmp/mypoint-build/app
cd /tmp/mypoint-build

# Giải nén Flutter web build output
unzip /tmp/web_export_20260413_110006.zip -d /tmp/mypoint-build/

# Chuyển nội dung vào app/ (bỏ thư mục bọc ngoài)
mv /tmp/mypoint-build/web_export_20260413_110006/* /tmp/mypoint-build/app/
rmdir /tmp/mypoint-build/web_export_20260413_110006
```

### 3.2. Kiểm tra cấu trúc

```bash
ls /tmp/mypoint-build/app/
```

Kết quả phải có:
```
index.html  main.dart.js  flutter_bootstrap.js  bundle.js
canvaskit/  assets/  icons/  manifest.json  version.json  ...
```

### 3.3. Tạo file nginx.conf

```bash
cat > /tmp/mypoint-build/nginx.conf << 'EOF'
worker_processes auto;
pid /tmp/nginx.pid;
error_log /var/log/nginx/error.log warn;
events { worker_connections 1024; }
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/javascript application/javascript
               application/json application/wasm image/svg+xml;
    types { application/wasm wasm; }
    server {
        listen 8080;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;

        # JS, WASM — cache 1 năm (immutable, chỉ đổi khi build mới)
        location ~* \.(js|css|wasm|json|svg)$ {
            gzip_static on;
            add_header Cache-Control "public, max-age=31536000, immutable";
            try_files $uri =404;
        }

        # HTML — không cache (SPA entry point phải luôn mới)
        location = /index.html {
            add_header Cache-Control "no-store, no-cache, must-revalidate, max-age=0";
        }

        # Flutter bootstrap — không cache
        location = /flutter_bootstrap.js {
            add_header Cache-Control "no-store, no-cache, must-revalidate, max-age=0";
        }

        # Ảnh, fonts — cache 7 ngày
        location ~* \.(png|jpg|jpeg|gif|ico|webp|otf|ttf|woff|woff2)$ {
            add_header Cache-Control "public, max-age=604800";
            try_files $uri =404;
        }

        # CORS Image Proxy (Flutter rewrite external images)
        location /ext-img {
            resolver 8.8.8.8 8.8.4.4 valid=300s ipv6=off;
            resolver_timeout 5s;
            set $ext_scheme $arg_scheme;
            set $ext_host $arg_host;
            set $ext_path $arg_path;
            if ($ext_scheme = '') { set $ext_scheme 'https'; }
            proxy_pass $ext_scheme://$ext_host$ext_path;
            proxy_set_header Host $ext_host;
            proxy_set_header Accept-Encoding "";
            proxy_hide_header Set-Cookie;
            proxy_connect_timeout 10s;
            proxy_read_timeout 15s;
            add_header Cache-Control "public, max-age=3600";
        }

        # SPA fallback — mọi route không match → index.html
        location / { try_files $uri $uri/ /index.html; }

        # Health check endpoints cho K8s probes
        location = /healthz {
            access_log off;
            return 200 'ok';
            add_header Content-Type text/plain;
        }
        location = /readyz {
            access_log off;
            return 200 'ready';
            add_header Content-Type text/plain;
        }

        # Chặn dotfiles
        location ~ /\. { deny all; }
    }
}
EOF
```

### 3.4. Tạo Dockerfile

```bash
cat > /tmp/mypoint-build/Dockerfile << 'EOF'
# Stage 1: Nén trước static files (giảm CPU runtime)
FROM alpine:3.20 AS compressor
RUN apk add --no-cache gzip findutils
WORKDIR /app
COPY app/ .
RUN find . -type f \( \
        -name "*.js" -o -name "*.css" -o -name "*.json" -o \
        -name "*.svg" -o -name "*.wasm" -o -name "*.html" \
    \) -size +1k -exec gzip -9 -k "{}" \;

# Stage 2: Nginx production image
FROM nginx:1.27-alpine
RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=compressor /app /usr/share/nginx/html
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup && \
    mkdir -p /var/cache/nginx /var/log/nginx /tmp && \
    chown -R appuser:appgroup /var/cache/nginx /var/log/nginx /tmp /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html
USER 1001
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -qO- http://localhost:8080/healthz || exit 1
CMD ["nginx", "-g", "daemon off;"]
EOF
```

### 3.5. Build image

```bash
cd /tmp/mypoint-build

sudo docker build -t 192.168.200.91/mypointwebihanoi/mypoint-web:2.01.01 \
                  -t 192.168.200.91/mypointwebihanoi/mypoint-web:latest .
```

### 3.6. Kiểm tra image

```bash
sudo docker images | grep mypoint-web
```

Kết quả mong đợi:
```
192.168.200.91/mypointwebihanoi/mypoint-web   2.01.01   xxx   ~40MB
192.168.200.91/mypointwebihanoi/mypoint-web   latest    xxx   ~40MB
```

### 3.7. Test local

```bash
sudo docker run -d --name mypoint-test -p 9090:8080 \
  192.168.200.91/mypointwebihanoi/mypoint-web:2.01.01

# Health check
curl -s http://localhost:9090/healthz
# → ok

# HTTP status
curl -s -o /dev/null -w "%{http_code}" http://localhost:9090/
# → 200

# Dọn dẹp
sudo docker stop mypoint-test && sudo docker rm mypoint-test
```

---

## 4. GIAI ĐOẠN 2 — PUSH LÊN HARBOR REGISTRY

> Thực hiện trên: **ai-agent-noibo**

### 4.1. Cấu hình insecure registry (Harbor chạy HTTP)

Kiểm tra file daemon.json:
```bash
cat /etc/docker/daemon.json
```

Nếu chưa có `192.168.200.91`:
```bash
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "insecure-registries": ["192.168.200.91"]
}
EOF
sudo systemctl restart docker
```

### 4.2. Login Harbor

```bash
sudo docker login 192.168.200.91
# Username: admin
# Password: ********
```

### 4.3. Push image

```bash
sudo docker push 192.168.200.91/mypointwebihanoi/mypoint-web:2.01.01
sudo docker push 192.168.200.91/mypointwebihanoi/mypoint-web:latest
```

### 4.4. Xác nhận trên Harbor UI

Truy cập http://192.168.200.91/harbor/projects → click **mypointwebihanoi** → phải thấy repo `mypoint-web` với tags `2.01.01` và `latest`.

---

## 5. GIAI ĐOẠN 3 — CẤU HÌNH K8S NODES

> Thực hiện trên: **tất cả 3 node** (master-01, worker-01, worker-02)

### Lý do

Harbor chạy HTTP (không HTTPS). containerd v2.2.2 (config version 3) mặc định chỉ kết nối HTTPS. Cần cấu hình để containerd biết dùng HTTP cho 192.168.200.91.

### 5.1. Lệnh chạy trên MỖI node

SSH vào từng node và chạy:

```bash
# Bước 1: Sửa config_path trong config.toml
# (chỉ đổi dòng config_path rỗng thành đường dẫn thực)
sudo sed -i "s|config_path = ''|config_path = '/etc/containerd/certs.d'|" /etc/containerd/config.toml

# Bước 2: Tạo hosts.toml cho Harbor
sudo mkdir -p /etc/containerd/certs.d/192.168.200.91
sudo tee /etc/containerd/certs.d/192.168.200.91/hosts.toml << 'EOF'
server = "http://192.168.200.91"

[host."http://192.168.200.91"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true
EOF

# Bước 3: Restart containerd
sudo systemctl restart containerd
```

### 5.2. Checklist

| Node | IP | Đã cấu hình? |
|---|---|---|
| master-01 | 192.168.200.80 | ☐ |
| worker-01 | 192.168.200.81 | ☐ |
| worker-02 | 192.168.200.82 | ☐ |

### 5.3. Kiểm tra

Trên bất kỳ worker, test pull (sẽ lỗi auth vì repo private — nhưng KHÔNG được lỗi HTTPS/443):
```bash
sudo crictl pull 192.168.200.91/mypointwebihanoi/mypoint-web:2.01.01
```

Lỗi đúng (chấp nhận):
```
authorization failed: no basic auth credentials
```

Lỗi sai (cần sửa lại):
```
dial tcp 192.168.200.91:443: connect: connection refused
```

### 5.4. Lưu ý đặc biệt cho containerd v2.2.2

Cú pháp cũ `plugins."io.containerd.grpc.v1.cri".registry` **KHÔNG hoạt động** trên containerd v2.x. Config version 3 dùng:

```toml
# ĐÚNG (v2.x / config version 3):
[plugins.'io.containerd.cri.v1.images'.registry]
  config_path = '/etc/containerd/certs.d'

# SAI (v1.x cũ, bị bỏ qua trên v2.x):
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
```

---

## 6. GIAI ĐOẠN 4 — DEPLOY LÊN KUBERNETES

> Thực hiện trên: **master-01** (192.168.200.80)

### 6.1. Tạo thư mục làm việc

```bash
mkdir -p ~/k8s-configs/mypoint-web
cd ~/k8s-configs/mypoint-web
```

### 6.2. Tạo Namespace

```bash
kubectl create namespace mypoint
```

### 6.3. Tạo Secret để pull image từ Harbor private repo

```bash
kubectl -n mypoint create secret docker-registry harbor-regcred \
  --docker-server=192.168.200.91 \
  --docker-username=admin \
  --docker-password=<HARBOR_PASSWORD> \
  --docker-email=hunghv@paytech.vn
```

> **Quan trọng**: Chạy trên MỘT DÒNG duy nhất. Thay `<HARBOR_PASSWORD>` bằng password thật.

Kiểm tra:
```bash
kubectl -n mypoint get secret harbor-regcred
```

### 6.4. Tạo Deployment

```bash
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mypoint-web
  namespace: mypoint
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: mypoint-web
  template:
    metadata:
      labels:
        app: mypoint-web
    spec:
      imagePullSecrets:
        - name: harbor-regcred
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - mypoint-web
                topologyKey: kubernetes.io/hostname
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        runAsGroup: 1001
        fsGroup: 1001
      containers:
        - name: mypoint-web
          image: 192.168.200.91/mypointwebihanoi/mypoint-web:2.01.01
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: http
            initialDelaySeconds: 5
            periodSeconds: 15
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /readyz
              port: http
            initialDelaySeconds: 3
            periodSeconds: 10
            failureThreshold: 2
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: nginx-cache
              mountPath: /var/cache/nginx
      volumes:
        - name: tmp
          emptyDir: {}
        - name: nginx-cache
          emptyDir: {}
EOF

kubectl apply -f deployment.yaml
```

### 6.5. Tạo Service

```bash
cat > service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: mypoint-web
  namespace: mypoint
spec:
  type: NodePort
  selector:
    app: mypoint-web
  ports:
    - name: http
      port: 80
      targetPort: http
      nodePort: 30080
EOF

kubectl apply -f service.yaml
```

### 6.6. Chờ pods sẵn sàng

```bash
kubectl -n mypoint wait --for=condition=Ready pod -l app=mypoint-web --timeout=120s
```

### 6.7. Xác nhận deployment

```bash
kubectl -n mypoint get pods -o wide
```

Kết quả mong đợi:
```
NAME                          READY   STATUS    NODE
mypoint-web-xxxxx-aaaaa       1/1     Running   worker-01
mypoint-web-xxxxx-bbbbb       1/1     Running   worker-02
```

### 6.8. Test truy cập

```bash
# Từ master-01
curl -s http://localhost:30080/healthz
# → ok

# Từ trình duyệt (bất kỳ máy nào trong mạng)
# http://192.168.200.80:30080
# http://192.168.200.81:30080
# http://192.168.200.82:30080
```

---

## 7. GIAI ĐOẠN 5 — KIỂM TRA & VẬN HÀNH

### 7.1. Kiểm tra tổng quan (chạy sau deploy)

```bash
echo "=== Namespace ==="
kubectl get ns mypoint

echo ""
echo "=== Pods ==="
kubectl -n mypoint get pods -o wide

echo ""
echo "=== Service ==="
kubectl -n mypoint get svc

echo ""
echo "=== Endpoints (pods backing service) ==="
kubectl -n mypoint get endpoints mypoint-web

echo ""
echo "=== Events gần nhất ==="
kubectl -n mypoint get events --sort-by='.lastTimestamp' | tail -15
```

### 7.2. Kiểm tra chi tiết từng pod

```bash
# Xem pod đang chạy trên node nào, IP nào
kubectl -n mypoint get pods -o wide

# Describe pod (xem events, probe status, image version)
kubectl -n mypoint describe pod <TEN_POD>

# Xem logs Nginx
kubectl -n mypoint logs <TEN_POD> --tail=30

# Xem logs realtime
kubectl -n mypoint logs -f <TEN_POD>

# Xem logs tất cả pods cùng lúc
kubectl -n mypoint logs -l app=mypoint-web --tail=20
```

### 7.3. Kiểm tra health

```bash
# Từ trong cluster (port-forward)
kubectl -n mypoint port-forward svc/mypoint-web 9090:80 &
curl -s http://localhost:9090/healthz    # → ok
curl -s http://localhost:9090/readyz     # → ready
curl -s -o /dev/null -w "%{http_code}" http://localhost:9090/  # → 200
kill %1

# Từ bên ngoài (qua NodePort)
curl -s http://192.168.200.80:30080/healthz
curl -s http://192.168.200.81:30080/healthz
curl -s http://192.168.200.82:30080/healthz
```

### 7.4. Kiểm tra Gzip hoạt động

```bash
# Không nén
curl -s -o /dev/null -w "Size: %{size_download} bytes\n" http://192.168.200.80:30080/main.dart.js

# Có nén
curl -s -H "Accept-Encoding: gzip" -o /dev/null -w "Size: %{size_download} bytes\n" http://192.168.200.80:30080/main.dart.js

# Size có nén phải nhỏ hơn nhiều (khoảng 700KB so với 3.5MB)
```

### 7.5. Kiểm tra resource usage

```bash
# Cần metrics-server đã cài
kubectl -n mypoint top pods

# Xem resource limits đang set
kubectl -n mypoint get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].resources}{"\n"}{end}'
```

### 7.6. Exec vào container (debug)

```bash
# Vào shell container
kubectl -n mypoint exec -it <TEN_POD> -- sh

# Kiểm tra nginx config
cat /etc/nginx/nginx.conf

# Kiểm tra files
ls -la /usr/share/nginx/html/

# Test từ bên trong
wget -qO- http://localhost:8080/healthz

# Thoát
exit
```

### 7.7. Rolling Update (deploy version mới)

```bash
# Cách 1: Đổi image tag
kubectl -n mypoint set image deployment/mypoint-web \
  mypoint-web=192.168.200.91/mypointwebihanoi/mypoint-web:<VERSION_MOI>

# Cách 2: Sửa file deployment.yaml rồi apply lại
# Đổi dòng image: thành version mới
kubectl apply -f deployment.yaml

# Theo dõi rollout
kubectl -n mypoint rollout status deployment/mypoint-web

# Xem history
kubectl -n mypoint rollout history deployment/mypoint-web
```

### 7.8. Rollback nếu có lỗi

```bash
# Rollback về version trước đó
kubectl -n mypoint rollout undo deployment/mypoint-web

# Rollback về revision cụ thể
kubectl -n mypoint rollout history deployment/mypoint-web
kubectl -n mypoint rollout undo deployment/mypoint-web --to-revision=2

# Kiểm tra sau rollback
kubectl -n mypoint get pods -o wide
kubectl -n mypoint rollout status deployment/mypoint-web
```

### 7.9. Scale thủ công

```bash
# Tăng lên 3 replicas
kubectl -n mypoint scale deployment mypoint-web --replicas=3

# Giảm về 2
kubectl -n mypoint scale deployment mypoint-web --replicas=2

# Kiểm tra
kubectl -n mypoint get pods -o wide
```

### 7.10. Restart pods (không downtime)

```bash
# Rolling restart — từng pod restart, không mất service
kubectl -n mypoint rollout restart deployment/mypoint-web

# Theo dõi
kubectl -n mypoint rollout status deployment/mypoint-web
```

---

## 8. TROUBLESHOOT

### 8.1. Pod ở trạng thái ImagePullBackOff

```bash
kubectl -n mypoint describe pod <TEN_POD> | grep -A5 Events
```

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| `dial tcp 192.168.200.91:443: connection refused` | containerd chưa cấu hình HTTP | Chạy lại Giai đoạn 3 trên node bị lỗi |
| `authorization failed: no basic auth credentials` | Secret sai hoặc chưa tạo | Xóa + tạo lại secret (xem 8.1.1) |
| `repository does not exist` | Tên image sai | Kiểm tra lại tên project/repo trên Harbor |

**8.1.1. Tạo lại Secret:**
```bash
kubectl -n mypoint delete secret harbor-regcred
kubectl -n mypoint create secret docker-registry harbor-regcred --docker-server=192.168.200.91 --docker-username=admin --docker-password=<PASSWORD> --docker-email=hunghv@paytech.vn
kubectl -n mypoint rollout restart deployment/mypoint-web
```

### 8.2. Pod ở trạng thái CrashLoopBackOff

```bash
# Xem logs lần chạy trước (bị crash)
kubectl -n mypoint logs <TEN_POD> --previous
```

Thường do nginx config lỗi. Sửa nginx.conf → build lại image → push → update.

### 8.3. Pod Running nhưng không truy cập được

```bash
# Kiểm tra endpoints
kubectl -n mypoint get endpoints mypoint-web
# Nếu trống → readiness probe fail → kiểm tra logs

# Test trực tiếp vào pod
kubectl -n mypoint port-forward <TEN_POD> 8888:8080
curl http://localhost:8888/healthz

# Kiểm tra service
kubectl -n mypoint describe svc mypoint-web
```

### 8.4. Kiểm tra containerd config đúng chưa

```bash
# Trên mỗi node
grep "config_path" /etc/containerd/config.toml
# Phải thấy: config_path = '/etc/containerd/certs.d'
# KHÔNG ĐƯỢC là: config_path = ''

cat /etc/containerd/certs.d/192.168.200.91/hosts.toml
# Phải thấy: server = "http://192.168.200.91"

# Kiểm tra containerd không log warning
sudo journalctl -u containerd --since "5 min ago" | grep -i "warn\|error"
```

### 8.5. NodePort không truy cập được từ ngoài

```bash
# Kiểm tra firewall
sudo iptables -L -n | grep 30080
sudo ufw status

# Nếu dùng UFW
sudo ufw allow 30080/tcp
```

---

## 9. PHỤ LỤC — FILE CẤU HÌNH ĐẦY ĐỦ

### 9.1. Cấu trúc thư mục trên master-01

```
~/k8s-configs/mypoint-web/
├── deployment.yaml
├── service.yaml
└── (optional)
    ├── namespace.yaml
    ├── ingress.yaml
    └── hpa.yaml
```

### 9.2. Cấu trúc thư mục build trên ai-agent-noibo

```
/tmp/mypoint-build/
├── Dockerfile
├── nginx.conf
└── app/
    ├── index.html
    ├── main.dart.js
    ├── flutter_bootstrap.js
    ├── bundle.js
    ├── canvaskit/
    ├── assets/
    └── ...
```

### 9.3. Containerd hosts.toml (trên mỗi K8s node)

File: `/etc/containerd/certs.d/192.168.200.91/hosts.toml`

```toml
server = "http://192.168.200.91"

[host."http://192.168.200.91"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true
```

### 9.4. Quy trình khi có version mới (tóm tắt)

```bash
# 1. Trên build server: giải nén build mới vào app/
# 2. Build image với tag mới
sudo docker build -t 192.168.200.91/mypointwebihanoi/mypoint-web:<VERSION_MOI> .

# 3. Push
sudo docker push 192.168.200.91/mypointwebihanoi/mypoint-web:<VERSION_MOI>

# 4. Trên master-01: update deployment
kubectl -n mypoint set image deployment/mypoint-web \
  mypoint-web=192.168.200.91/mypointwebihanoi/mypoint-web:<VERSION_MOI>

# 5. Kiểm tra
kubectl -n mypoint rollout status deployment/mypoint-web
kubectl -n mypoint get pods -o wide
curl -s http://192.168.200.80:30080/healthz
```

---

## LỊCH SỬ THAY ĐỔI TÀI LIỆU

| Version | Ngày | Người | Nội dung |
|---|---|---|---|
| 1.0 | 14/04/2026 | hunghv | Tạo mới — quy trình deploy A-Z |

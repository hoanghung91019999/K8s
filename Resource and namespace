# Kubernetes Concepts — Giải Thích Các Khái Niệm Cốt Lõi

> Tài liệu dành cho team vận hành cụm Kubernetes on-premises

---

## 1. RESOURCE LÀ GÌ?

Resource là **bất kỳ thứ gì bạn tạo ra trong Kubernetes**. Mỗi lần bạn chạy `kubectl apply -f file.yaml`, bạn đang tạo 1 hoặc nhiều resource.

### 1.1 Danh sách Resource thường dùng

| Resource | Vai trò | So sánh Docker |
|----------|---------|----------------|
| **Pod** | Đơn vị nhỏ nhất chạy container | `docker run` |
| **Deployment** | Quản lý Pod — đảm bảo đủ số lượng, tự tạo lại khi lỗi | `docker-compose up` |
| **Service** | Cổng vào Pod — giúp các Pod tìm thấy nhau | `ports:` trong compose |
| **Ingress** | Phân luồng HTTP theo domain/path | Nginx reverse proxy |
| **ConfigMap** | Lưu biến môi trường (không nhạy cảm) | `environment:` trong compose |
| **Secret** | Lưu mật khẩu, API key (mã hóa base64) | File `.env` |
| **PersistentVolumeClaim (PVC)** | Yêu cầu cấp ổ cứng | `volumes:` trong compose |
| **PersistentVolume (PV)** | Ổ cứng thực tế được cấp | Ổ cứng vật lý |
| **StorageClass** | Loại ổ cứng (Longhorn, NFS, Local) | Không có trong Docker |
| **Namespace** | Phân nhóm resource theo "phòng" | Không có trong Docker |
| **DaemonSet** | Chạy đúng 1 Pod trên MỖI node | Không có trong Docker |
| **StatefulSet** | Quản lý Pod có trạng thái (database) | Không có trong Docker |
| **CronJob** | Chạy task định kỳ (backup, cleanup) | `crontab` trên Linux |
| **Job** | Chạy task 1 lần rồi dừng | `docker run` rồi thoát |
| **HPA** | Tự động scale Pod theo CPU/RAM | Không có trong Docker |

### 1.2 Mối quan hệ giữa các Resource

```
Deployment (Người quản lý)
│
├── Quản lý → ReplicaSet (Đảm bảo đủ số Pod)
│                │
│                ├── Tạo → Pod 1 (Container chạy bên trong)
│                ├── Tạo → Pod 2
│                └── Tạo → Pod 3
│
├── Kết nối → Service (Cổng vào cho Pod)
│                │
│                └── Kết nối → Ingress (Phân luồng theo domain)
│
├── Đọc cấu hình từ → ConfigMap (Biến môi trường)
├── Đọc mật khẩu từ → Secret (Thông tin nhạy cảm)
│
└── Gắn ổ cứng → PVC (Yêu cầu lưu trữ)
                    │
                    └── Bind → PV (Ổ cứng thực tế)
                                │
                                └── Cung cấp bởi → StorageClass (Longhorn)
```

### 1.3 Ví dụ thực tế — 1 ứng dụng cần những Resource nào

```
Ứng dụng: Website bán hàng

webapp (Deployment)          → Chạy 2 Pod Nginx
├── webapp-config (ConfigMap) → APP_ENV=production, API_URL=...
├── webapp-service (Service)  → Mở port 80 nội bộ
└── webapp-ingress (Ingress)  → shop.paytech.vn → webapp-service

api (Deployment)             → Chạy 3 Pod Node.js
├── api-config (ConfigMap)    → DB_HOST=mysql-service
├── api-secret (Secret)       → DB_PASSWORD=xxx
└── api-service (Service)     → Mở port 8080 nội bộ

mysql (Deployment)           → Chạy 1 Pod MySQL
├── mysql-secret (Secret)     → MYSQL_ROOT_PASSWORD=xxx
├── mysql-service (Service)   → Mở port 3306 nội bộ
└── mysql-data (PVC)          → Yêu cầu 20GB ổ cứng Longhorn
```

### 1.4 Lệnh xem Resource

```bash
# Xem tất cả loại resource có trong cluster
kubectl api-resources

# Xem resource cụ thể
kubectl get pods                    # Xem Pod
kubectl get deployments             # Xem Deployment
kubectl get svc                     # Xem Service (viết tắt)
kubectl get ingress                 # Xem Ingress
kubectl get pvc                     # Xem PersistentVolumeClaim

# Xem TẤT CẢ resource trong 1 namespace
kubectl -n myapp get all

# Xem chi tiết 1 resource
kubectl -n myapp describe pod <pod-name>
kubectl -n myapp describe svc <service-name>

# Xem resource dạng YAML
kubectl -n myapp get deployment webapp -o yaml
```

---

## 2. NAMESPACE LÀ GÌ?

Namespace là **phòng trong ngôi nhà** — giúp phân tách resource theo nhóm, tránh lẫn lộn và dễ quản lý.

### 2.1 Hình dung trực quan

```
Cluster (Ngôi nhà)
│
├── namespace: kube-system            ← Phòng hệ thống K8s
│   ├── coredns pod                     (KHÔNG ĐỤNG VÀO)
│   ├── kube-proxy pod
│   ├── kube-apiserver pod
│   ├── kube-scheduler pod
│   ├── kube-controller-manager pod
│   └── etcd pod
│
├── namespace: metallb-system         ← Phòng MetalLB
│   ├── controller pod
│   ├── speaker pod (trên mỗi node)
│   ├── IPAddressPool
│   └── L2Advertisement
│
├── namespace: ingress-nginx          ← Phòng Ingress
│   ├── ingress-nginx-controller pod
│   └── ingress-nginx-controller service (IP: 192.168.200.85)
│
├── namespace: longhorn-system        ← Phòng Storage
│   ├── longhorn-ui pod x2
│   ├── longhorn-manager pod
│   ├── longhorn-driver pod
│   └── longhorn-frontend service
│
├── namespace: myapp                  ← Phòng ứng dụng A (bạn tạo)
│   ├── webapp pod x2
│   ├── api pod x3
│   ├── mysql pod x1
│   └── các service, ingress, secret...
│
├── namespace: staging                ← Phòng test (bạn có thể tạo)
│   ├── webapp-staging pod
│   └── mysql-staging pod
│
└── namespace: default                ← Phòng mặc định (thường để trống)
```

### 2.2 Tại sao cần Namespace?

**Lý do 1 — Tách biệt ứng dụng**

```bash
# Xóa toàn bộ app mà KHÔNG ảnh hưởng hệ thống
kubectl delete namespace myapp
# → Chỉ xóa webapp, api, mysql trong namespace myapp
# → MetalLB, Ingress, Longhorn hoàn toàn không bị ảnh hưởng
```

**Lý do 2 — Tránh trùng tên**

```bash
# 2 team có thể tạo service cùng tên "webapp" mà không xung đột
namespace: team-a → service/webapp
namespace: team-b → service/webapp    # Không trùng!
```

**Lý do 3 — Phân quyền**

```bash
# Dev chỉ được xem namespace "staging", không được đụng "production"
# (Cấu hình qua RBAC — Role-Based Access Control)
```

**Lý do 4 — Giới hạn tài nguyên**

```yaml
# Giới hạn namespace myapp chỉ được dùng tối đa 4 CPU, 8GB RAM
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myapp-quota
  namespace: myapp
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

### 2.3 Namespace mặc định trong cluster

| Namespace | Mục đích | Được phép xóa? |
|-----------|---------|---------------|
| `default` | Namespace mặc định nếu không chỉ định | Không nên |
| `kube-system` | Thành phần hệ thống K8s | **TUYỆT ĐỐI KHÔNG** |
| `kube-public` | Resource công khai | Không nên |
| `kube-node-lease` | Heartbeat giữa node và master | **TUYỆT ĐỐI KHÔNG** |
| `metallb-system` | MetalLB LoadBalancer | Không (trừ khi gỡ MetalLB) |
| `ingress-nginx` | Nginx Ingress Controller | Không (trừ khi gỡ Ingress) |
| `longhorn-system` | Longhorn Storage | Không (trừ khi gỡ Longhorn) |

### 2.4 Lệnh quản lý Namespace

```bash
# Xem tất cả namespace
kubectl get namespaces

# Tạo namespace mới
kubectl create namespace myapp
kubectl create namespace staging

# Xem resource trong namespace cụ thể
kubectl -n myapp get all
kubectl -n longhorn-system get pods

# Xem resource TOÀN BỘ namespace
kubectl get pods --all-namespaces

# Đặt namespace mặc định (không cần gõ -n mỗi lần)
kubectl config set-context --current --namespace=myapp

# Sau đó chỉ cần gõ:
kubectl get pods          # → tự hiểu là kubectl -n myapp get pods

# Quay về default
kubectl config set-context --current --namespace=default

# Xóa namespace (CẨN THẬN — xóa TẤT CẢ resource bên trong!)
kubectl delete namespace staging
```

### 2.5 Giao tiếp giữa các Namespace

Pod trong namespace khác nhau **vẫn nói chuyện được** với nhau, nhưng cần dùng tên đầy đủ:

```
Cùng namespace:
  mysql-service                              ← Ngắn gọn

Khác namespace:
  mysql-service.myapp.svc.cluster.local      ← Tên đầy đủ (FQDN)
  │             │     │   │
  │             │     │   └── domain gốc cluster
  │             │     └── viết tắt của "service"
  │             └── tên namespace
  └── tên service
```

Ví dụ: Pod ở namespace `staging` muốn kết nối MySQL ở namespace `myapp`:

```yaml
# Trong configmap của staging
DB_HOST: "mysql-service.myapp.svc.cluster.local"
```

---

## 3. RESOURCE QUAN TRỌNG — GIẢI THÍCH CHI TIẾT

### 3.1 Pod

Pod là **đơn vị nhỏ nhất** trong Kubernetes. 1 Pod chứa 1 hoặc nhiều container.

```
Pod (giống 1 căn phòng)
├── Container chính: nginx
├── Container phụ: log-collector (sidecar — tùy chọn)
└── Shared: network, storage
```

Đặc điểm quan trọng:
- Pod có IP riêng (ví dụ: 10.10.171.5)
- Pod là **tạm thời** — có thể bị xóa và tạo lại bất kỳ lúc nào
- Không bao giờ tạo Pod trực tiếp — luôn dùng Deployment để quản lý Pod

### 3.2 Deployment

Deployment là **người quản lý Pod**. Bạn nói "tôi cần 3 Pod webapp", Deployment đảm bảo luôn có đúng 3 Pod chạy.

```
Bạn nói: replicas: 3

Deployment đảm bảo:
  Pod 1: Running ✅
  Pod 2: Running ✅
  Pod 3: Running ✅

Nếu Pod 2 chết:
  Pod 1: Running ✅
  Pod 2: Dead ❌ → Deployment tự tạo Pod 4 thay thế
  Pod 3: Running ✅
  Pod 4: Running ✅ (mới tạo)
```

### 3.3 Service

Service là **số điện thoại cố định** cho nhóm Pod. Pod có thể chết và tạo lại (IP thay đổi), nhưng Service luôn giữ nguyên IP.

```
Service: webapp-service (IP: 10.105.75.118)
         │
         │ Load balance giữa các Pod
         │
    ┌────┼────┐
    ▼    ▼    ▼
  Pod1  Pod2  Pod3     ← IP thay đổi liên tục
```

3 loại Service:

| Type | Truy cập từ đâu | Dùng khi nào |
|------|-----------------|-------------|
| `ClusterIP` | Chỉ trong cluster | Mặc định, dùng với Ingress |
| `NodePort` | IP node + port 30000-32767 | Test nhanh |
| `LoadBalancer` | IP ngoài (MetalLB cấp) | Service TCP/UDP không qua Ingress |

### 3.4 Ingress

Ingress là **Nginx reverse proxy** được Kubernetes tự động cấu hình.

```
1 IP duy nhất: 192.168.200.85
       │
       ▼
┌─────────────────────────────┐
│      Ingress Controller      │
│                              │
│  Host: shop.paytech.vn      │──→ webapp-service:80
│  Host: api.paytech.vn       │──→ api-service:8080
│  Host: longhorn.paytech.vn  │──→ longhorn-frontend:80
│  Host: admin.paytech.vn     │──→ admin-service:3000
└─────────────────────────────┘
```

### 3.5 PVC & PV

PVC là **đơn đặt hàng ổ cứng**, PV là **ổ cứng thực tế** được cấp.

```
Bạn tạo PVC: "Tôi cần 10GB"
       │
       ▼
StorageClass (Longhorn) nhận yêu cầu
       │
       ▼
Tạo PV 10GB trên worker-01 + replica trên worker-02
       │
       ▼
PVC bound → Pod mount và dùng
```

### 3.6 ConfigMap & Secret

```
ConfigMap (Cấu hình thường)          Secret (Thông tin nhạy cảm)
┌─────────────────────┐              ┌─────────────────────┐
│ APP_ENV=production   │              │ DB_PASSWORD=xxx      │
│ DB_HOST=mysql-svc    │              │ API_KEY=yyy          │
│ LOG_LEVEL=info       │              │ JWT_SECRET=zzz       │
└─────────────────────┘              └─────────────────────┘
      Lưu dạng plain text                 Lưu dạng base64
      Ai cũng đọc được                   Cần quyền mới đọc được
```

---

## 4. SƠ ĐỒ TỔNG THỂ CLUSTER HIỆN TẠI

```
                        Internet / Mạng LAN
                              │
                    ┌─────────▼──────────┐
                    │   192.168.200.85    │
                    │   (MetalLB VIP)     │
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │  Ingress Nginx      │
                    │  (Reverse Proxy)    │
                    │                     │
                    │  *.paytech.vn       │
                    └─────────┬──────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                     │
  longhorn.paytech.vn   shop.paytech.vn      api.paytech.vn
         │                    │                     │
  ┌──────▼──────┐     ┌──────▼──────┐      ┌──────▼──────┐
  │ longhorn-ui  │     │   webapp    │      │    api      │
  │ (2 Pods)     │     │  (2 Pods)   │      │  (3 Pods)   │
  │ longhorn-    │     │  myapp      │      │  myapp      │
  │ system ns    │     │  namespace  │      │  namespace  │
  └──────────────┘     └──────┬──────┘      └──────┬──────┘
                              │                     │
                       ┌──────▼─────────────────────▼──┐
                       │         mysql (1 Pod)          │
                       │         myapp namespace        │
                       │              │                 │
                       │         ┌────▼────┐            │
                       │         │   PVC   │            │
                       │         │ Longhorn│            │
                       │         │  20GB   │            │
                       │         └─────────┘            │
                       └────────────────────────────────┘

Hạ tầng vật lý:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  master-01    │  │  worker-01    │  │  worker-02    │
│  Control Plane│  │  Workload     │  │  Workload     │
│  192.168.200. │  │  192.168.200. │  │  192.168.200. │
│  80           │  │  81           │  │  82           │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## 5. TÓM TẮT NHANH

| Khái niệm | Giải thích 1 câu |
|-----------|-----------------|
| **Resource** | Bất kỳ thứ gì bạn tạo trong K8s (Pod, Service, Ingress...) |
| **Namespace** | Phòng/thư mục chứa resource — giúp phân tách và quản lý |
| **Pod** | Container đang chạy (đơn vị nhỏ nhất) |
| **Deployment** | Người quản lý Pod — đảm bảo đủ số lượng |
| **Service** | Cổng vào cố định cho nhóm Pod |
| **Ingress** | Nginx proxy tự động — phân luồng theo domain |
| **ConfigMap** | Biến môi trường thường |
| **Secret** | Mật khẩu, API key |
| **PVC** | Đơn đặt hàng ổ cứng |
| **StorageClass** | Loại ổ cứng (Longhorn) |

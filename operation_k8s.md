# Kubectl Cheat Sheet — Lệnh Vận Hành K8s Cần Biết

> Tham khảo nhanh cho cụm Kubernetes on-premises (master-01, worker-01, worker-02)

---

## 1. KIỂM TRA SỨC KHỎE CLUSTER

```bash
# Xem tất cả node và trạng thái
kubectl get nodes

# Xem chi tiết node (CPU, RAM, số Pod đang chạy)
kubectl describe node worker-01

# Xem mức sử dụng CPU/RAM từng node
kubectl top nodes

# Xem thông tin cluster
kubectl cluster-info

# Xem tất cả resource trong cluster
kubectl get all --all-namespaces
```

---

## 2. QUẢN LÝ POD

```bash
# Xem pod trong namespace cụ thể
kubectl -n myapp get pods

# Xem pod trên tất cả namespace
kubectl get pods --all-namespaces

# Xem pod chi tiết (đang chạy trên node nào, IP nào)
kubectl -n myapp get pods -o wide

# Xem chi tiết 1 pod (events, lỗi, trạng thái)
kubectl -n myapp describe pod <pod-name>

# Xem mức sử dụng CPU/RAM từng pod
kubectl -n myapp top pods

# Xóa 1 pod (K8s sẽ tự tạo pod mới thay thế)
kubectl -n myapp delete pod <pod-name>

# Xóa pod bị stuck (force delete)
kubectl -n myapp delete pod <pod-name> --grace-period=0 --force
```

---

## 3. XEM LOG

```bash
# Xem log pod
kubectl -n myapp logs <pod-name>

# Xem log realtime (giống tail -f)
kubectl -n myapp logs -f <pod-name>

# Xem 100 dòng log cuối
kubectl -n myapp logs --tail=100 <pod-name>

# Xem log của pod đã crash (lần chạy trước)
kubectl -n myapp logs <pod-name> --previous

# Xem log của tất cả pod trong 1 deployment
kubectl -n myapp logs -f deployment/webapp

# Xem log của container cụ thể (nếu pod có nhiều container)
kubectl -n myapp logs <pod-name> -c <container-name>
```

---

## 4. VÀO BÊN TRONG CONTAINER

```bash
# Mở shell bash
kubectl -n myapp exec -it <pod-name> -- /bin/bash

# Nếu container không có bash, dùng sh
kubectl -n myapp exec -it <pod-name> -- /bin/sh

# Chạy 1 lệnh đơn lẻ (không cần vào shell)
kubectl -n myapp exec <pod-name> -- ls /app
kubectl -n myapp exec <pod-name> -- cat /etc/nginx/nginx.conf
kubectl -n myapp exec <pod-name> -- env    # xem biến môi trường
```

---

## 5. DEPLOYMENT (Quản lý ứng dụng)

```bash
# Xem danh sách deployment
kubectl -n myapp get deployments

# Xem chi tiết deployment
kubectl -n myapp describe deployment webapp

# Scale (tăng/giảm số pod)
kubectl -n myapp scale deployment webapp --replicas=3    # tăng lên 3 pod
kubectl -n myapp scale deployment webapp --replicas=1    # giảm về 1 pod

# Cập nhật image (deploy version mới)
kubectl -n myapp set image deployment/webapp webapp=nginx:1.25

# Xem lịch sử deploy
kubectl -n myapp rollout history deployment/webapp

# Xem trạng thái đang deploy
kubectl -n myapp rollout status deployment/webapp

# Rollback về version trước (nếu deploy lỗi)
kubectl -n myapp rollout undo deployment/webapp

# Rollback về version cụ thể
kubectl -n myapp rollout undo deployment/webapp --to-revision=2

# Restart tất cả pod trong deployment (không downtime)
kubectl -n myapp rollout restart deployment/webapp
```

---

## 6. SERVICE & NETWORKING

```bash
# Xem tất cả service
kubectl -n myapp get svc

# Xem chi tiết service (endpoint, selector)
kubectl -n myapp describe svc webapp-service

# Xem endpoint (pod nào đang nhận traffic)
kubectl -n myapp get endpoints webapp-service

# Xem tất cả ingress
kubectl get ingress --all-namespaces

# Xem chi tiết ingress
kubectl -n myapp describe ingress webapp-ingress

# Test kết nối từ trong cluster
kubectl run test --image=busybox --rm -it -- wget -qO- http://webapp-service.myapp:80
```

---

## 7. STORAGE

```bash
# Xem PersistentVolumeClaim (đơn đặt hàng ổ cứng)
kubectl -n myapp get pvc

# Xem PersistentVolume (ổ cứng thực tế)
kubectl get pv

# Xem StorageClass
kubectl get storageclass

# Xem chi tiết PVC
kubectl -n myapp describe pvc mysql-data
```

---

## 8. CONFIGMAP & SECRET

```bash
# Xem danh sách configmap
kubectl -n myapp get configmap

# Xem nội dung configmap
kubectl -n myapp describe configmap webapp-config

# Xem nội dung configmap dạng yaml
kubectl -n myapp get configmap webapp-config -o yaml

# Xem danh sách secret
kubectl -n myapp get secret

# Xem nội dung secret (base64 encoded)
kubectl -n myapp get secret mysql-secret -o yaml

# Decode secret để đọc giá trị thật
kubectl -n myapp get secret mysql-secret -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 -d

# Tạo configmap từ command
kubectl -n myapp create configmap app-config --from-literal=APP_ENV=production

# Tạo secret từ command
kubectl -n myapp create secret generic db-secret --from-literal=PASSWORD=abc123
```

---

## 9. NAMESPACE

```bash
# Xem tất cả namespace
kubectl get namespaces

# Tạo namespace mới
kubectl create namespace myapp

# Xóa namespace (XÓA TẤT CẢ resource bên trong!)
kubectl delete namespace myapp

# Đặt namespace mặc định (không cần gõ -n myapp mỗi lần)
kubectl config set-context --current --namespace=myapp

# Quay về default
kubectl config set-context --current --namespace=default
```

---

## 10. EVENTS & DEBUG

```bash
# Xem events gần đây (rất hữu ích khi debug)
kubectl -n myapp get events --sort-by='.lastTimestamp'

# Xem events của toàn cluster
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Debug pod không start được
kubectl -n myapp describe pod <pod-name>    # xem phần Events ở cuối

# Debug DNS
kubectl run test --image=busybox --rm -it -- nslookup webapp-service.myapp.svc.cluster.local

# Debug network
kubectl run test --image=busybox --rm -it -- ping <pod-ip>

# Xem config kubectl đang dùng
kubectl config view

# Kiểm tra quyền
kubectl auth can-i create pods -n myapp
```

---

## 11. APPLY / DELETE RESOURCE

```bash
# Apply file yaml
kubectl apply -f deployment.yaml

# Apply tất cả file trong thư mục
kubectl apply -f ~/k8s-configs/apps/

# Apply đệ quy (tất cả thư mục con)
kubectl apply -R -f ~/k8s-configs/

# Xóa resource từ file
kubectl delete -f deployment.yaml

# Xóa resource trực tiếp
kubectl -n myapp delete deployment webapp
kubectl -n myapp delete svc webapp-service
kubectl -n myapp delete ingress webapp-ingress

# Xóa tất cả pod trong namespace (deployment sẽ tạo lại)
kubectl -n myapp delete pods --all
```

---

## 12. EXPORT / BACKUP

```bash
# Export resource hiện tại ra yaml
kubectl -n myapp get deployment webapp -o yaml > webapp-backup.yaml

# Export tất cả resource trong namespace
kubectl -n myapp get all -o yaml > myapp-full-backup.yaml

# Export ingress
kubectl get ingress --all-namespaces -o yaml > ingress-backup.yaml

# Export secret
kubectl -n myapp get secret -o yaml > secrets-backup.yaml
```

---

## 13. LỆNH HỮU ÍCH HÀNG NGÀY

```bash
# Xem tổng quan nhanh toàn cluster (chạy mỗi sáng)
echo "=== NODES ===" && kubectl get nodes && \
echo "=== PODS LỖI ===" && kubectl get pods --all-namespaces | grep -v Running | grep -v Completed && \
echo "=== INGRESS ===" && kubectl get ingress --all-namespaces

# Tìm pod bị lỗi trên toàn cluster
kubectl get pods --all-namespaces | grep -E "Error|CrashLoop|Pending|ImagePull"

# Xem pod nào đang dùng nhiều CPU/RAM nhất
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# Xem node nào đang đầy tải
kubectl top nodes

# Đếm số pod trên mỗi node
kubectl get pods --all-namespaces -o wide --no-headers | awk '{print $8}' | sort | uniq -c | sort -rn
```

---

## 14. BẢNG SO SÁNH DOCKER ↔ KUBECTL

| Mục đích | Docker | Kubectl |
|----------|--------|---------|
| Xem container/pod | `docker ps` | `kubectl get pods` |
| Xem log | `docker logs -f <name>` | `kubectl logs -f <pod>` |
| Vào container | `docker exec -it <name> bash` | `kubectl exec -it <pod> -- bash` |
| Dừng container | `docker stop <name>` | `kubectl delete pod <pod>` |
| Xem images | `docker images` | (không có — image quản lý ở registry) |
| Xem network | `docker network ls` | `kubectl get svc` |
| Xem volume | `docker volume ls` | `kubectl get pvc` |
| Xem tài nguyên | `docker stats` | `kubectl top pods` |
| Deploy | `docker-compose up -d` | `kubectl apply -f .` |
| Dừng tất cả | `docker-compose down` | `kubectl delete -f .` |
| Scale | `docker-compose up --scale web=3` | `kubectl scale deploy web --replicas=3` |
| Restart | `docker restart <name>` | `kubectl rollout restart deploy <name>` |

---

## 15. MẸO TIẾT KIỆM THỜI GIAN

### Alias — gõ nhanh hơn

```bash
# Thêm vào ~/.bashrc rồi chạy: source ~/.bashrc
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ka='kubectl apply -f'
alias kdel='kubectl delete'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgi='kubectl get ingress'
alias kgn='kubectl get nodes'

# Sau khi thêm alias, thay vì gõ:
#   kubectl -n myapp get pods
# Chỉ cần gõ:
#   k -n myapp get pods
```

### Auto-complete — nhấn Tab để gợi ý

```bash
# Bật auto-complete cho kubectl
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# Giờ gõ: kubectl get d[Tab] → tự hoàn thành "deployments"
```

---

## TÓM TẮT: 5 LỆNH CHẠY MỖI NGÀY

```bash
kubectl get nodes                    # Cluster có khỏe không?
kubectl get pods --all-namespaces    # Pod nào đang lỗi?
kubectl top nodes                    # Node nào đầy tải?
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -20   # Có gì bất thường?
kubectl get ingress --all-namespaces # Ingress hoạt động đúng?
```

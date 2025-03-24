# K8s
## kubernetes là gì ?
- Kubernetes (K8s- orchestration) là một nền tảng mã nguồn mở giúp tự động triển khai, mở rộng và quản lý các ứng dụng container. Nó ban đầu được phát triển bởi Google và hiện được Cloud Native Computing Foundation (CNCF) duy trì.
### tại sao nên sử dụng k8s 
- Kubernetes giúp bạn tự động hóa việc triển khai, mở rộng và quản lý ứng dụng container, đặc biệt khi hệ thống có nhiều container chạy trên nhiều máy chủ khác nhau. Nếu không có Kubernetes, bạn sẽ phải quản lý thủ công các container, gây khó khăn khi mở rộng hệ thống hoặc xử lý lỗi.

- Tự động hóa việc triển khai & quản lý container
    + Không cần khởi động container bằng tay, K8s tự động chạy đúng số lượng container cần thiết.
    + Nếu container bị lỗi, K8s sẽ tự động restart hoặc thay thế.
- Tự động cân bằng tải (Load Balancing)
    +  Nếu một container nhận quá nhiều request, K8s có thể tự động tạo thêm container mới để chia tải.
    +  Kubernetes Service đảm bảo request luôn được gửi đến các Pod còn hoạt động.
- Tự động mở rộng (Auto-scaling)
    + Hỗ trợ Horizontal Pod Autoscaler (HPA) để tự động scale up/down số lượng Pod theo CPU, RAM hoặc metric tùy chỉnh.
    + Hỗ trợ Cluster Autoscaler để tự động thêm hoặc giảm số lượng node trong cluster.
- Khả năng tự phục hồi (Self-healing)
    + Nếu một container bị lỗi, K8s sẽ tự động khởi động lại nó.
    + Nếu một node bị crash, K8s sẽ di chuyển Pod sang node khác.
- Dễ dàng triển khai ứng dụng mới mà không gây downtime
    + Hỗ trợ Rolling Update để cập nhật phiên bản mới mà không làm gián đoạn dịch vụ.
    + Hỗ trợ Canary Deployment & Blue-Green Deployment để kiểm tra bản cập nhật trước khi đưa vào production.
- Tích hợp tốt với các hệ thống lưu trữ & mạng
    + Hỗ trợ các hệ thống lưu trữ như NFS, AWS EBS, Google Persistent Disk, Ceph...
    + Quản lý mạng nội bộ giữa các container bằng Kubernetes Network Policy.
- Quản lý cấu hình & bảo mật hiệu quả
    + ConfigMap & Secret giúp lưu trữ cấu hình và dữ liệu nhạy cảm (mật khẩu, API key) mà không cần hardcode vào code.
    + RBAC (Role-Based Access Control) giúp phân quyền người dùng và ứng dụng một cách an toàn.
- Chạy được trên mọi nền tảng (Cloud, On-Prem, Hybrid Cloud)
    + Kubernetes có thể chạy trên AWS, GCP, Azure, On-Premises, hoặc Hybrid Cloud.
    + Dễ dàng di chuyển ứng dụng giữa các môi trường mà không cần thay đổi quá nhiều.
### kiến trúc của K8s
- gồm 2 thành phần chính :
    + control plane/Master node : quản lý triển khai 
    + worker node (data plane)
![image](https://github.com/user-attachments/assets/91640a84-c5e6-43b6-b921-afdd7fac477f)
#### Control Plane
- API Server :
    + Cung cấp API để giao tiếp với Kubernetes.
    + Xử lý các yêu cầu từ kubectl, dashboard, hoặc các thành phần khác.
    + Là "cửa ngõ" của Kubernetes.
- etcd (Database lưu trữ)
    + Lưu trữ toàn bộ trạng thái của cluster. dưới dạng key-value
    + Kubelet trên mỗi Worker Node gửi thông tin lên API Server (trạng thái Node, tài nguyên sử dụng (CPU, RAM, Disk), và các Pod đang chạy lên API Server.) --> API Server xử lý và ghi dữ liệu vào etcd --> Controller Manager kiểm tra và cập nhật trạng thái Node trong etcd --> Scheduler & Kube Proxy sử dụng dữ liệu từ etcd 
    + Khi có thay đổi (ví dụ: thêm Pod, xóa Node), etcd cập nhật dữ liệu.
- Controller Manager (kube-controller-manager)
    + Gồm nhiều controller để kiểm tra và quản lý trạng thái cluster.
- Scheduler (kube-scheduler)
    + Quyết định Pod chạy trên Node nào dựa trên tài nguyên, taints, affinities...
    + Ví dụ: Nếu Node A còn RAM, CPU phù hợp → Scheduler sẽ đặt Pod vào Node A.
#### Worker Nodes (Chạy ứng dụng)
- Kubelet
    + Agent chạy trên mỗi Node, giao tiếp với API Server.
    + Kiểm tra trạng thái của Pod và đảm bảo container chạy đúng theo yêu cầu.
- Kube Proxy
    + Quản lý networking, giúp các Pod giao tiếp với nhau và với bên ngoài.
    + Điều phối traffic theo Service (ClusterIP, NodePort, LoadBalancer).
- Container Runtime
    + Chạy container thực tế (Docker, containerd, CRI-O…).
    + Kubernetes không bắt buộc dùng Docker, có thể dùng bất kỳ runtime nào hỗ trợ Container Runtime Interface (CRI).
![image](https://github.com/user-attachments/assets/ae5a1d98-fb39-42a7-b3c0-bd93f3fb92f8)
- quy trình làm việc
  + API nhận yêu cầu từ admin --> API server đi hỏi scheduler deploy lên worker node nào --> scheduler không thể đi hỏi từng worker note ( vì trong cluster có thể có hàng trăm worker note). scheduler lấy dữ liệu từ ETCD xem hiện tại worker node nào phù hợp chỉ định deploy lên worker node nào --> lúc này scheduler thông tin lại cho API server --> API server sẽ giao tiếp với kubelet trên worker node đó và deploy pod lên worker node
## Hướng dẫn cài K8s
##### Sử dụng Kubespray 
- Kubespray là một công cụ mã nguồn mở dùng để triển khai Kubernetes trên nhiều môi trường khác nhau (bare-metal, cloud, VM). Nó sử dụng Ansible để tự động hóa quá trình cài đặt và cấu hình cluster Kubernetes.
- Hướng dẫn cài xem [tại đây](https://github.com/hoanghung91019999/K8s/new/mai)

##### sử dụng kubeadm
- Kubeadm là một công cụ chính thức của Kubernetes giúp cài đặt và cấu hình một cluster Kubernetes một cách đơn giản và nhanh chóng. Nó tự động thực hiện các bước cần thiết để thiết lập một cluster tối giản nhưng hoạt động được, bao gồm:
    + Tạo control-plane (API Server, Controller, Scheduler, v.v.)
    + Kết nối các worker node vào cluster
    + Cấu hình các chứng chỉ TLS & token bảo mật
    + Thiết lập networking cơ bản (DNS, CNI)
- hướng dẫn cài đặt xem tại đây
    
# Pod là gì
- Pod là đơn vị nhỏ nhất trong Kubernetes, chứa một hoặc nhiều container chạy cùng nhau trên một node.
- Tất cả các container trong pod sẽ:
      + Chia sẻ network namespace (cùng địa chỉ IP)
      + Chia sẻ storage volumes (nếu có).
      + Có thể giao tiếp với nhau thông qua localhost.
- Mỗi pod có một địa chỉ IP duy nhất trong cluster.
- Các pod giao tiếp với nhau qua Service hoặc DNS nội bộ.
- Các container trong pod giao tiếp với nhau qua localhost.
- Các container trong pod có thể chia sẻ Persistent Volume để lưu trữ dữ liệu.
- Kiểm tra danh sách các pods đang chạy
```
kubectl get pods
```
- Xem thông tin chi tiết về một pod
```
kubectl describe pod <tên-pod>
```
- Tạo một pod từ file YAML ,Ví dụ, tạo một file pod.yaml với nội dung:
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
chạy lệnh: 
kubectl apply -f pod.yaml
```
- Xóa một pod
```
kubectl delete pod <tên-pod>
```
- Kiểm tra logs của container trong pod
```
kubectl logs <tên-pod>
```
- Nếu pod có nhiều container, chỉ định container cụ thể:
```
kubectl logs <tên-pod> -c <tên-container>
```
- Truy cập vào container trong pod
```
kubectl exec -it <tên-pod> -- /bin/sh
```
-  Forward port từ pod ra ngoài
```
kubectl port-forward pod/<tên-pod> 8080:80
```
- Kiểm tra trạng thái pod liên tục
```
kubectl get pods --watch
```
- Lấy thông tin pod dưới dạng YAML
```
kubectl get pod <tên-pod> -o yaml
```
- Debug pod nếu bị lỗi (CrashLoopBackOff, Pending, etc.)
```
kubectl describe pod <tên-pod>
kubectl logs <tên-pod>
```
# nodeport
- NodePort là một loại Service trong Kubernetes, giúp mở một cổng trên tất cả các Node trong cluster, cho phép truy cập từ bên ngoài vào Pod.
- Nguyên lý hoạt động:
    + Kubernetes tự động mở một cổng trên tất cả các node (ví dụ: 30000-32767).
    + Khi có request đến NodeIP:NodePort, Kubernetes sẽ forward request đó đến Pod bên trong cluster.
- cấu trúc kết nối
```
Client --> NodeIP:NodePort --> Service --> Pod
```
- Cấu hình NodePort trong Kubernetes
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80         # Cổng trong cluster
      targetPort: 8080 # Cổng của container (Pod)
      nodePort: 30080  # Cổng mở trên Node (tùy chọn, nếu không chỉ định, Kubernetes sẽ chọn tự động)
```
- chạy lệnh
```
kubectl apply -f service-nodeport.yaml
```
-  Xem danh sách Service trong cluster
```
kubectl get svc
```
- Xem chi tiết Service cụ thể
```
kubectl describe svc my-service
```
- xóa service
```
kubectl delete svc my-service
```
# logs
- xem logs
```
kubectl logs < name pods >
```
- sử dụng exec
```
kubectl exec -it <pod_name> -- <command>
```
# declarative 
- sử dụng tài liệu trên kubernetes.io
# replicaset

                                                                                                                                                                                                                                                                                                 


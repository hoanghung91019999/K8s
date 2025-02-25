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

    

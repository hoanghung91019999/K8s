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
- ReplicaSet (RS) là một tài nguyên trong Kubernetes giúp đảm bảo rằng một số lượng cố định của Pod luôn chạy trong cluster. Nó đảm nhiệm vai trò duy trì trạng thái mong muốn bằng cách tự động tạo thêm hoặc xóa bớt Pod khi cần thiết.
- ReplicaSet bao gồm 3 phần chính:
    + Số lượng Replica (replicas): Xác định số lượng Pod mong muốn.
    + Selector: Định danh các Pod mà ReplicaSet sẽ quản lý.
    + Template: Mẫu Pod sẽ được ReplicaSet tạo ra khi cần.
- Vòng đời hoạt động
     + bước 1 : ReplicaSet kiểm tra số lượng Pod : Khi ReplicaSet được tạo, nó sẽ kiểm tra trong cluster xem có bao nhiêu Pod khớp với selector của nó.
     + Bước 2: Điều chỉnh số lượng Pod : Nếu số Pod ít hơn giá trị replicas → ReplicaSet tạo thêm Pod.Nếu số Pod nhiều hơn giá trị replicas → ReplicaSet xóa bớt Pod.
     + Bước 3: Theo dõi và tự động cân bằng. Nếu một Pod bị xóa hoặc gặp lỗi → ReplicaSet sẽ tự động tạo Pod mới để thay thế.Nếu tài nguyên hệ thống bị thiếu → ReplicaSet cố gắng duy trì số Pod gần với giá trị replicas nhất có thể.

- ví dụ :
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3  # Số lượng Pod mong muốn
  selector:
    matchLabels:
      app: nginx  # Chỉ quản lý các Pod có label "app=nginx"
  template:
    metadata:
      labels:
        app: nginx  # Label để Pod được quản lý bởi ReplicaSet này
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```
- Label là các cặp key-value được gán cho đối tượng (Pod, Node, Service, ReplicaSet, v.v.).
- Selector được dùng để tìm kiếm và chọn lọc các đối tượng có label phù hợp.
- Expose ReplicaSet giúp tạo Service để truy cập các Pod.

# deployment
- Deployment là một tài nguyên trong Kubernetes giúp quản lý ReplicaSet và Pod. Nó cung cấp các tính năng như:
    +  Tự động triển khai (deploy) các Pod.
    +  Cập nhật phiên bản ứng dụng mà không làm gián đoạn dịch vụ.
    +  Khôi phục (rollback) về phiên bản trước nếu có lỗi.
    +  Tự động thay đổi số lượng Pod (scaling) theo nhu cầu.
- Thành phần chính
    + ReplicaSet: Đảm bảo số lượng Pod luôn chạy đúng với mong muốn.
    + Pod Template: Mẫu Pod chứa cấu hình về container, image, ports, v.v.
    + Strategy (Chiến lược triển khai): Cách cập nhật Pod khi có thay đổi.
- Vòng đời hoạt động
    + tạo Deployment → Kubernetes tạo ReplicaSet quản lý các Pod.
    + Kiểm tra số lượng Pod → ReplicaSet tạo đúng số Pod theo replicas.
    + Cập nhật Deployment → Triển khai phiên bản mới theo chiến lược cập nhật.
    + Rollback nếu lỗi → Nếu phát hiện lỗi, có thể quay lại phiên bản trước.
    +  Tự động sửa lỗi → Nếu một Pod bị lỗi, ReplicaSet sẽ tạo Pod mới thay thế.
- ví dụ :
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3  # Số lượng Pod mong muốn
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```
- triển khai
```
kubectl apply -f deployment.yaml
```
- Kiểm tra Deployment
```
kubectl get deployments
kubectl get pods -o wide
```
- Kiểm tra ReplicaSet
```
kubectl get rs
```
- Cập nhật Image của Deployment
```
kubectl set image deployment/nginx-deployment nginx=nginx:1.21
```
- **Kubernetes sẽ thực hiện Rolling Update, thay thế từng Pod một.**
- Kiểm tra quá trình cập nhật
```
kubectl rollout status deployment/nginx-deployment
kubectl get pods
```
### Chiến lược cập nhật Deployment
- Rolling Update (Mặc định)
    + Thay thế từng Pod một, không ảnh hưởng đến ứng dụng.
    + Kubernetes tạo Pod mới trước khi xóa Pod cũ.
- Cấu hình trong YAML:
```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1  # Số lượng Pod bị mất tối đa
    maxSurge: 1        # Số lượng Pod mới được tạo thêm tối đa
```
- Recreate (Xóa hết Pod cũ, tạo Pod mới)
    + Toàn bộ Pod cũ sẽ bị xóa trước, sau đó Pod mới được tạo.
    + Dẫn đến downtime nếu không có cơ chế dự phòng.
- Cấu hình trong YAML:
```
strategy:
  type: Recreate
```
- Dùng khi nào?
    + Khi ứng dụng không hỗ trợ chạy song song nhiều phiên bản cùng lúc.
    + Khi có thay đổi lớn về cấu trúc dữ liệu.
- **Rollback (Quay lại phiên bản cũ)**
- Kiểm tra lịch sử Deployment
```
kubectl rollout history deployment/nginx-deployment
```
- Quay lại phiên bản trước
```
kubectl rollout undo deployment/nginx-deployment
```
### Expose Deployment bằng Service
- Dùng kubectl expose
```
kubectl expose deployment nginx-deployment --type=NodePort --port=80
```
- xóa Deployment
```
kubectl delete deployment nginx-deployment
```
##### lưu ý 
- không cần tạo ReplicaSet trước!
- Khi bạn tạo Deployment, Kubernetes tự động tạo ReplicaSet để quản lý các Pod.
-  Bạn không cần khai báo ReplicaSet riêng, vì Deployment đã bao gồm cơ chế này.
  ### *Khi nào nên dùng ReplicaSet mà không cần Deployment?*
-  Trong thực tế, luôn dùng Deployment trừ khi bạn cần: ✔ Chạy Pod cố định mà không cần cập nhật tự động.
-   Có một hệ thống quản lý Deployment riêng. Nhưng hầu hết các trường hợp, bạn chỉ cần Deployment.

# service 
![image](https://github.com/user-attachments/assets/b12a0844-30cf-4b62-b63c-459ee3758f92)
- kube proxy giúp các pod ( container trong các pod )  giao tiếp với nhau trong 1 node và pod giữa các node
- mỗi service trên k8s sẽ có 1 IP , IP này sẽ đi kèm với dịch vụ suốt quá trình vận hành
- service giúp cân bằng tải ( ví dụ service 1 có 2 pod hệ thống sẽ tự động cân bằng tải )
- cấu trúc tạo service
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```
- nếu type khoogn định nghĩa loại service sẽ mặc định là cluster IP
##### cluster IP 
- các pod có thể dược redeploy và địa chỉ IP của pod thay đổi
- cluster IP sẽ cấp 1 địa chỉ IP vĩnh viễn
- Bất kỳ Pod nào trong cluster có thể gọi tên Service (  myapp-service ) để gửi request thay vì gọi trực tiếp IP của Pod.
- Kubernetes sử dụng iptables hoặc IPVS để load balance request đến các Pod đang chạy.
- khi không có cluster IP các pod vẫn có thể giao tiếp với nhau bằng IP của pod hoặc sử dụng core DNS là name của pod ( 2 cách này phức tạp và ko tối ưu )
##### nodeport 
- nodeport sẽ Cấp phát một cổng ngẫu nhiên từ 30000-32767 trên mỗi Node trong cluster
- Mọi request đến <NodeIP>:<NodePort> sẽ được chuyển tiếp đến Pod đích.
- ví dụ có 3 node và bạn muốn truy cập vào 1 pod nằm ở 1 trong 3 node này chỉ cần điền đúng port đã được mở ra bên ngoài k8s sẽ tự biết và định tuyến đến đúng pod đó
- Nhận request trên NodePort --> Forward request đến ClusterIP của Service. --> Service tiếp tục load balance request đến một Pod backend.
- cấu trúc :
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80           # Service lắng nghe trên cổng 80 (ClusterIP)
      targetPort: 8080   # Chuyển tiếp request đến cổng 8080 của Pod
      nodePort: 30080    # Mở cổng 30080 trên tất cả các Node (nếu không chỉ định, Kubernetes sẽ tự động chọn một cổng từ 30000-32767)
  type: NodePort
```
##### loadbalancer
- chỉ hỗ trợ ở môi trường cloud ( dưới onpremis có thể sử dụng các tool HAproxy ,MetalLB , Ingress Controller
- ![image](https://github.com/user-attachments/assets/f6f6cb2e-110b-47d8-9391-5bf91ee9553e)
- Khi bạn tạo một Service kiểu LoadBalancer, Kubernetes sẽ yêu cầu nhà cung cấp cloud tạo một IP công cộng và load balancer.
- Load balancer này sẽ forward lưu lượng đến các Pod trong cluster thông qua IP và port của Service.
- Kubernetes sẽ sử dụng Round Robin hoặc các thuật toán load balancing khác để phân phối lưu lượng đến các Pod backend.
##### externalname 
- k8s sẽ tạo 1 dns name và chuyển tiếp tới 1 domain name bên ngoài cluster
```
apiVersion: v1
kind: Service
metadata:
  name: my-external-api
spec:
  type: ExternalName
  externalName: api.example.com
```
- khi Pod gọi my-external-api, Kubernetes sẽ tự động chuyển tiếp yêu cầu đến api.example.com.

# Namespaces
- Trong Kubernetes, Namespace là một cách để tách biệt các tài nguyên trong cùng một cluster
- Mỗi Namespace có thể chứa các Pod, Service, Deployment, và các tài nguyên khác, giúp bạn dễ dàng phân chia, quản lý và kiểm soát các nhóm tài nguyên khác nhau trong cluster.
- Đây là một công cụ rất hữu ích khi bạn muốn phân chia tài nguyên, tổ chức cluster, và giảm thiểu xung đột giữa các ứng dụng hoặc môi trường khác nhau.
#### Kiến trúc và Cách thức Hoạt động của Namespaces trong Kubernetes
- Mỗi Namespace là một không gian logic:
    + Các tài nguyên trong Kubernetes như Pods, Services, Deployments, Secrets, và ConfigMaps đều có thể được chỉ định vào một namespace cụ thể.
    + Mỗi namespace là một không gian riêng biệt, với tài nguyên của nó không ảnh hưởng trực tiếp đến các namespace khác.
- Mặc định không có Namespace:
    + Khi bạn không chỉ định một namespace, Kubernetes sẽ tự động gán tài nguyên đó vào namespace mặc định (default).
- Tài nguyên không thể trùng tên giữa các namespaces:
    + Trong cùng một namespace, bạn không thể có hai tài nguyên có cùng tên. Tuy nhiên, các tài nguyên có thể có cùng tên nhưng nằm trong các namespaces khác nhau.
- Tài nguyên trong namespace có thể giao tiếp với nhau:
    + Pods trong một namespace có thể giao tiếp với nhau thông qua tên dịch vụ hoặc các tài nguyên khác mà không cần cấu hình phức tạp.
- Kết nối giữa các namespaces:
    + Nếu bạn muốn các tài nguyên trong một namespace giao tiếp với tài nguyên trong namespace khác, bạn cần chỉ định namespace trong các truy vấn (ví dụ: thông qua DNS hoặc các service discovery).
#### tạo namespace
- tạo namespace
```
kubectl create namespace <namespace-name>
```
- tạo bằng file yml
```
apiVersion: v1
kind: Namespace
metadata:
  name: development
```
- deploy pod vào namespace
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: development
spec:
  containers:
  - name: mycontainer
    image: myimage
```
- deploy pod vào name space và service
```
apiVersion: v1
kind: Deployment
metadata:
  name: appv1
  namespace: development
  lables:
    app: app1
spec:
  replicaset: 3
  selecttor:
    matchLabels:
        app: app1
  template:
    metadata:
        labels:
            app: app1
    spec:
      containers:
        - name: simple-app
          image: demo/app
          port:
            - containerPort: 8080    
---
apiVersion: v1
kind: Service
metadata:
  name: nodeport
  namespace: development      
  lables:
    app: app1
spec:
  type: Nodeport
  selector:
    app: app1
 ports:
   - port: 8080
     targetPort : 8080
```
- liệt kê namespace
```
kubectl get namespaces
```
- liệt kê các Pods trong một namespace cụ thể
```
kubectl get pods -n <namespace-name>
```
- lấy thông tin về Pod
```
kubectl get pod <pod-name> -n <namespace-name>
```
- Để liệt kê tất cả tài nguyên (Pods, Deployments, Services, v.v.) trong một namespace
```
kubectl get all -n <namespace-name>
```
- Để liệt kê Pods trong tất cả các namespaces
```
kubectl get pods --all-namespaces
```
# 🚀 Hướng Dẫn Xây Dựng Cụm Kubernetes On-Premises trên Ubuntu 22.04

> **Từ Docker Compose đến Kubernetes — Step by Step cho người mới**
> Tác giả: Infrastructure Architect | AIOps Engineer

---

## 📋 MỤC LỤC

1. [Tổng quan: Docker Compose vs Kubernetes](#1-tổng-quan)
2. [Quy hoạch hạ tầng & Chuẩn bị máy chủ](#2-quy-hoạch)
3. [Chuẩn bị hệ thống (Tất cả các node)](#3-chuẩn-bị-hệ-thống)
4. [Cài đặt Container Runtime (containerd)](#4-cài-containerd)
5. [Cài đặt kubeadm, kubelet, kubectl](#5-cài-k8s-tools)
6. [Khởi tạo Master Node (Control Plane)](#6-khởi-tạo-master)
7. [Join Worker Nodes vào cụm](#7-join-worker)
8. [Cài đặt CNI — Mạng nội bộ cho Pod](#8-cài-cni)
9. [Cài đặt MetalLB — LoadBalancer cho Bare-metal](#9-metallb)
10. [Cài đặt Ingress Controller (Nginx)](#10-ingress)
11. [Cài đặt Storage (NFS / Local Path)](#11-storage)
12. [Chuyển đổi Docker Compose sang Kubernetes](#12-chuyển-đổi)
13. [Công cụ quản lý & Monitoring](#13-công-cụ)
14. [Checklist & Troubleshooting](#14-checklist)

---

## 1. TỔNG QUAN

### 1.1 Tại sao cần chuyển từ Docker Compose sang Kubernetes?

Hãy tưởng tượng thế này:

- **Docker Compose** giống như bạn quản lý một **quán cà phê nhỏ** — bạn tự tay pha chế, phục vụ, tính tiền. Mọi thứ chạy trên 1 máy duy nhất.
- **Kubernetes** giống như bạn quản lý một **chuỗi cà phê** — có hệ thống tự động phân công nhân viên, khi 1 chi nhánh sập thì tự mở chi nhánh khác thay thế, tự scale khi đông khách.

| Tính năng | Docker Compose | Kubernetes |
|-----------|---------------|------------|
| Chạy trên | 1 máy duy nhất | Nhiều máy (cluster) |
| Tự khởi động lại khi lỗi | Cơ bản (restart policy) | Nâng cao (self-healing) |
| Tự scale | Không | Có (HPA, VPA) |
| Load Balancing | Không có sẵn | Có sẵn (Service) |
| Rolling Update | Không | Có (zero downtime) |
| Quản lý Secret | File .env | Secret object (mã hóa) |
| Phù hợp | Dev, dự án nhỏ | Production, dự án lớn |

### 1.2 Các khái niệm cốt lõi cần hiểu

```
┌─────────────────────────────────────────────────────┐
│                   CLUSTER (Cụm K8s)                 │
│                                                     │
│  ┌──────────────┐  ┌──────────┐  ┌──────────┐     │
│  │ MASTER NODE  │  │ WORKER 1 │  │ WORKER 2 │     │
│  │ (Bộ não)     │  │ (Tay chân)│ │(Tay chân)│     │
│  │              │  │          │  │          │     │
│  │ • API Server │  │ ┌──────┐ │  │ ┌──────┐ │     │
│  │ • Scheduler  │  │ │ Pod  │ │  │ │ Pod  │ │     │
│  │ • etcd       │  │ │┌────┐│ │  │ │┌────┐│ │     │
│  │ • Controller │  │ ││Con-││ │  │ ││Con-││ │     │
│  │   Manager    │  │ ││tai-││ │  │ ││tai-││ │     │
│  │              │  │ ││ner ││ │  │ ││ner ││ │     │
│  │              │  │ │└────┘│ │  │ │└────┘│ │     │
│  │              │  │ └──────┘ │  │ └──────┘ │     │
│  └──────────────┘  └──────────┘  └──────────┘     │
└─────────────────────────────────────────────────────┘
```

**So sánh dễ hiểu với Docker Compose:**

| Docker Compose | Kubernetes | Giải thích |
|---------------|------------|------------|
| `docker-compose.yml` | `Deployment` + `Service` | File cấu hình chính |
| `service` (trong compose) | `Pod` / `Deployment` | 1 ứng dụng chạy |
| `ports: "8080:80"` | `Service` (type: NodePort/LoadBalancer) | Mở cổng ra ngoài |
| `volumes:` | `PersistentVolume` + `PersistentVolumeClaim` | Lưu trữ dữ liệu |
| `environment:` | `ConfigMap` / `Secret` | Biến môi trường |
| `networks:` | `CNI` (Calico/Flannel) | Mạng nội bộ |
| `depends_on:` | Không cần (K8s tự quản lý) | Thứ tự khởi động |
| `restart: always` | Mặc định trong Deployment | Tự khởi động lại |

---

## 2. QUY HOẠCH HẠ TẦNG

### 2.1 Cấu hình tối thiểu đề xuất

```
Cụm Kubernetes tối thiểu cho Production:

┌─────────────────────────────────────────────────┐
│            Network: 192.168.1.0/24              │
│                                                 │
│  ┌─────────────┐                                │
│  │  master-01   │  IP: 192.168.1.10            │
│  │  CPU: 2 core │  RAM: 4GB  │  Disk: 50GB    │
│  │  Role: Control Plane                        │
│  └─────────────┘                                │
│                                                 │
│  ┌─────────────┐                                │
│  │  worker-01   │  IP: 192.168.1.11            │
│  │  CPU: 4 core │  RAM: 8GB  │  Disk: 100GB   │
│  │  Role: Worker                               │
│  └─────────────┘                                │
│                                                 │
│  ┌─────────────┐                                │
│  │  worker-02   │  IP: 192.168.1.12            │
│  │  CPU: 4 core │  RAM: 8GB  │  Disk: 100GB   │
│  │  Role: Worker                               │
│  └─────────────┘                                │
│                                                 │
│  (Tùy chọn) ┌──────────┐                       │
│  │  nfs-server  │  IP: 192.168.1.20            │
│  │  Disk: 500GB │  Dùng cho Shared Storage     │
│  └──────────┘                                   │
└─────────────────────────────────────────────────┘
```

> **Lưu ý:** Trong bài hướng dẫn này, tôi dùng IP ví dụ trên. Bạn hãy thay bằng IP thực tế của bạn.

### 2.2 Checklist trước khi bắt đầu

- [ ] Tất cả máy chủ đã cài Ubuntu 22.04 LTS (Server)
- [ ] Tất cả máy nằm cùng mạng LAN, ping được lẫn nhau
- [ ] Có quyền root / sudo trên tất cả máy
- [ ] Mỗi máy có hostname khác nhau
- [ ] Đã tắt swap trên tất cả máy
- [ ] Thời gian đồng bộ (NTP) trên tất cả máy

---

## 3. CHUẨN BỊ HỆ THỐNG (Thực hiện trên TẤT CẢ các node)

> ⚠️ **QUAN TRỌNG:** Phần 3, 4, 5 phải thực hiện trên TẤT CẢ các máy (master + worker)

### Bước 3.1 — Đặt hostname cho từng máy

```bash
# === Trên máy master ===
sudo hostnamectl set-hostname master-01

# === Trên máy worker 1 ===
sudo hostnamectl set-hostname worker-01

# === Trên máy worker 2 ===
sudo hostnamectl set-hostname worker-02
```

**Giải thích:** Hostname giống như "tên" của mỗi máy. Kubernetes cần mỗi máy có tên riêng biệt để phân biệt.

### Bước 3.2 — Cấu hình file hosts

```bash
# Thực hiện trên TẤT CẢ các máy
sudo tee -a /etc/hosts <<EOF

# Kubernetes Cluster
192.168.200.80  master-01
192.168.200.81  worker-01
192.168.200.82  worker-02
EOF
```

**Giải thích:** File `/etc/hosts` giống như "danh bạ điện thoại" — giúp các máy tìm nhau bằng tên thay vì phải nhớ IP.

### Bước 3.3 — Tắt Swap (BẮT BUỘC)

```bash
# Tắt swap ngay lập tức
sudo swapoff -a

# Tắt swap vĩnh viễn (khi khởi động lại máy vẫn tắt)
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Kiểm tra — kết quả phải là 0
free -h | grep Swap
# Swap:            0B          0B          0B   <-- Đúng!
```

**Tại sao phải tắt Swap?**
Kubernetes cần kiểm soát chính xác bộ nhớ RAM mà mỗi container sử dụng. Nếu swap bật, khi RAM đầy, hệ thống sẽ dùng ổ cứng làm RAM (rất chậm) — điều này khiến Kubernetes không thể biết container thực sự cần bao nhiêu RAM và gây ra lỗi không lường trước.

### Bước 3.4 — Cấu hình Kernel Modules

```bash
# Tạo file cấu hình load module khi boot
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# Load module ngay bây giờ (không cần restart)
sudo modprobe overlay
sudo modprobe br_netfilter
```

**Giải thích đơn giản:**
- `overlay`: Cho phép container xếp chồng nhiều "lớp" filesystem — giống như nhiều tấm kính trong suốt xếp chồng lên nhau tạo thành 1 bức tranh hoàn chỉnh.
- `br_netfilter`: Cho phép Linux "nhìn thấy" và kiểm soát traffic mạng giữa các container (bridge network).

### Bước 3.5 — Cấu hình Sysctl cho Networking

```bash
# Thiết lập tham số mạng
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Áp dụng ngay
sudo sysctl --system
```

**Giải thích:**
- `bridge-nf-call-iptables = 1`: Cho phép iptables (tường lửa Linux) kiểm soát traffic đi qua bridge network → Kubernetes cần điều này để điều hướng traffic giữa các Pod.
- `ip_forward = 1`: Cho phép máy chuyển tiếp gói tin mạng giữa các network interface → Cần thiết để Pod trên máy này nói chuyện được với Pod trên máy khác.

### Bước 3.6 — Cập nhật hệ thống & Cài đặt các gói cần thiết

```bash
sudo apt-get update && sudo apt-get upgrade -y

sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release \
  software-properties-common
```

---

## 4. CÀI ĐẶT CONTAINER RUNTIME (containerd)

> **Tại sao containerd mà không phải Docker?**
> Từ Kubernetes 1.24+, Docker không còn được hỗ trợ trực tiếp. Kubernetes dùng containerd — "phần ruột" bên trong Docker — để chạy container. Hãy nghĩ Docker = containerd + nhiều tính năng thêm. Kubernetes chỉ cần phần containerd thôi.

### Bước 4.1 — Cài đặt containerd

```bash
# Thêm Docker repository (containerd nằm trong repo của Docker)
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Cài containerd
sudo apt-get update
sudo apt-get install -y containerd.io
```

### Bước 4.2 — Cấu hình containerd

```bash
# Tạo file cấu hình mặc định
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# BẮT BUỘC: Bật SystemdCgroup
# Kubernetes yêu cầu containerd dùng systemd làm cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' \
  /etc/containerd/config.toml

# Khởi động lại containerd
sudo systemctl restart containerd
sudo systemctl enable containerd

# Kiểm tra trạng thái
sudo systemctl status containerd
# ● containerd.service - containerd container runtime
#      Active: active (running)   <-- Phải thấy dòng này!
```

**Giải thích SystemdCgroup:**
Cgroup (Control Group) là cách Linux giới hạn tài nguyên (CPU, RAM) cho mỗi process. Kubernetes và containerd cần dùng chung 1 "người quản lý" cgroup — đó là systemd. Nếu không đồng bộ, kubelet sẽ bị crash liên tục.

---

## 5. CÀI ĐẶT kubeadm, kubelet, kubectl

> **3 công cụ này là gì?**
> - **kubeadm**: Công cụ "xây nhà" — dùng để khởi tạo và thiết lập cluster
> - **kubelet**: "Quản đốc công trường" trên mỗi máy — nhận lệnh từ master và chạy container
> - **kubectl**: "Remote control" — công cụ dòng lệnh để bạn ra lệnh cho cluster

### Bước 5.1 — Thêm Kubernetes Repository

```bash
# Tạo thư mục cho keyring
sudo mkdir -p /etc/apt/keyrings

# Tải signing key của Kubernetes
# (Dùng phiên bản 1.30 — bạn có thể thay đổi version)
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Thêm repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Bước 5.2 — Cài đặt và ghim phiên bản

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# "Ghim" version — không cho tự động upgrade
# (Upgrade K8s cần làm có kế hoạch, không nên để tự động)
sudo apt-mark hold kubelet kubeadm kubectl

# Kiểm tra version
kubeadm version
kubectl version --client
```

---

## 6. KHỞI TẠO MASTER NODE (Control Plane)

> ⚠️ **CHỈ thực hiện trên máy MASTER**

### Bước 6.1 — Khởi tạo Cluster

```bash
# Khởi tạo cluster
# --pod-network-cidr: Dải IP nội bộ cho các Pod (dùng cho Calico)
# --apiserver-advertise-address: IP của máy master
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=192.168.1.10 \
  --control-plane-endpoint=192.168.1.10
```

**Giải thích các tham số:**
- `--pod-network-cidr=10.244.0.0/16`: Đây là dải IP "ảo" dành riêng cho các Pod bên trong cluster. Giống như bạn tạo 1 mạng LAN riêng cho container, không liên quan đến mạng LAN thật của bạn.
- `--apiserver-advertise-address`: IP thật của máy master — để worker biết kết nối đến đâu.
- `--control-plane-endpoint`: Endpoint của Control Plane (dùng IP master, hoặc VIP nếu có HA).

### Bước 6.2 — Lưu lại thông tin quan trọng

Sau khi chạy thành công, bạn sẽ thấy output tương tự:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Then you can join any number of worker nodes by running the following
on each as root:

  kubeadm join 192.168.1.10:6443 --token abc123.xxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxx
```

> 📝 **GHI LẠI** lệnh `kubeadm join ...` — bạn sẽ cần nó ở Bước 7!

### Bước 6.3 — Cấu hình kubectl cho user hiện tại

```bash
# Tạo thư mục .kube và copy config
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Kiểm tra — phải thấy master node (trạng thái NotReady là bình thường
# vì chưa cài CNI network plugin)
kubectl get nodes
# NAME        STATUS     ROLES           AGE   VERSION
# master-01   NotReady   control-plane   1m    v1.30.x
```

**Tại sao "NotReady"?**
Master node đang ở trạng thái NotReady vì chưa có plugin mạng (CNI). Giống như bạn đã xây xong tòa nhà nhưng chưa lắp hệ thống điện thoại nội bộ — các phòng chưa liên lạc được với nhau. Chúng ta sẽ cài CNI ở Bước 8.

---

## 7. JOIN WORKER NODES VÀO CỤM

> ⚠️ **Thực hiện trên TỪNG máy WORKER**

### Bước 7.1 — Chạy lệnh join

```bash
# Dán lệnh đã lưu từ Bước 6.2
sudo kubeadm join 192.168.1.10:6443 \
  --token abc123.xxxxxxxxxxxx \
  --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxx
```

> **Nếu bạn quên hoặc mất token**, chạy lệnh này trên MASTER để tạo lại:
> ```bash
> kubeadm token create --print-join-command
> ```

### Bước 7.2 — Kiểm tra trên Master

```bash
# Quay lại máy MASTER, kiểm tra
kubectl get nodes
# NAME        STATUS     ROLES           AGE   VERSION
# master-01   NotReady   control-plane   5m    v1.30.x
# worker-01   NotReady   <none>          1m    v1.30.x
# worker-02   NotReady   <none>          30s   v1.30.x
```

Tất cả đều NotReady — hoàn toàn bình thường! Tiếp tục bước tiếp theo.

### Bước 7.3 — Gán label cho Worker (tùy chọn nhưng nên làm)

```bash
# Gán role "worker" cho dễ nhận biết
kubectl label node worker-01 node-role.kubernetes.io/worker=worker
kubectl label node worker-02 node-role.kubernetes.io/worker=worker
```

---

## 8. CÀI ĐẶT CNI — MẠNG NỘI BỘ CHO POD

> **CNI (Container Network Interface) là gì?**
> CNI giống như "hệ thống đường ống nước" trong tòa nhà. Nó cho phép các Pod (container) ở các máy khác nhau nói chuyện được với nhau. Không có CNI, Pod trên worker-01 không thể giao tiếp với Pod trên worker-02.

### Lựa chọn CNI phổ biến:

| CNI | Ưu điểm | Phù hợp |
|-----|---------|---------|
| **Flannel** | Đơn giản, dễ cài | Cluster nhỏ, mới bắt đầu |
| **Calico** | Mạnh mẽ, hỗ trợ Network Policy | Production |
| **Cilium** | Hiện đại, eBPF-based | Cluster lớn, hiệu năng cao |

### Cài Flannel (Đơn giản nhất — khuyến nghị cho người mới)

```bash
# Trên MASTER
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

### HOẶC Cài Calico (Khuyến nghị cho Production)

```bash
# Trên MASTER
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

### Kiểm tra sau khi cài CNI

```bash
# Đợi 1-2 phút rồi kiểm tra
kubectl get nodes
# NAME        STATUS   ROLES           AGE   VERSION
# master-01   Ready    control-plane   10m   v1.30.x   <-- Ready!
# worker-01   Ready    worker          5m    v1.30.x   <-- Ready!
# worker-02   Ready    worker          4m    v1.30.x   <-- Ready!

# Kiểm tra tất cả các Pod hệ thống đang chạy
kubectl get pods -n kube-system
# Tất cả phải ở trạng thái Running hoặc Completed
```

🎉 **Cluster đã sẵn sàng!** Tất cả node đều ở trạng thái Ready.

---

## 9. CÀI ĐẶT MetalLB — LOADBALANCER CHO BARE-METAL

> **Tại sao cần MetalLB?**
> Trên Cloud (AWS, GCP), khi bạn tạo Service type `LoadBalancer`, cloud tự cấp 1 IP public. Nhưng On-premises không có ai cấp IP cho bạn → MetalLB sẽ làm việc đó. Nó lấy 1 dải IP từ mạng LAN của bạn và gán cho các Service.

### Bước 9.1 — Cài MetalLB

```bash
# Cài MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml

# Đợi cho MetalLB pods sẵn sàng
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=90s
```

### Bước 9.2 — Cấu hình IP Pool

```bash
# Tạo file cấu hình cho MetalLB
# Dải IP này phải CHƯA ĐƯỢC SỬ DỤNG trong mạng LAN của bạn
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.200-192.168.1.250  # <-- Thay bằng dải IP trống trong LAN của bạn
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```

**Giải thích:**
- `IPAddressPool`: Bạn "cho" MetalLB một dải IP (ví dụ: 192.168.1.200 - 192.168.1.250). Mỗi khi có Service type LoadBalancer, MetalLB sẽ lấy 1 IP từ dải này gán vào.
- `L2Advertisement`: MetalLB sẽ "quảng bá" các IP này trong mạng LAN bằng giao thức Layer 2 (ARP) — giống như nó nói "Này mạng LAN, IP 192.168.1.200 ở đây nhé!"

---

## 10. CÀI ĐẶT INGRESS CONTROLLER (Nginx)

> **Ingress là gì?**
> Hãy tưởng tượng Ingress là "bảo vệ tòa nhà" — khi có request HTTP đến, Ingress kiểm tra domain/path rồi chuyển đến đúng dịch vụ bên trong.
>
> Ví dụ:
> - `app.example.com` → Service A
> - `api.example.com` → Service B
> - `app.example.com/admin` → Service C

### Bước 10.1 — Cài Nginx Ingress Controller

```bash
# Cài bằng manifest chính thức
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/baremetal/deploy.yaml
```

### Bước 10.2 — Chuyển sang LoadBalancer (dùng với MetalLB)

```bash
# Mặc định, bản baremetal dùng NodePort.
# Chúng ta đổi sang LoadBalancer vì đã có MetalLB
kubectl -n ingress-nginx patch svc ingress-nginx-controller \
  -p '{"spec":{"type":"LoadBalancer"}}'

# Kiểm tra — MetalLB sẽ gán 1 IP từ pool
kubectl -n ingress-nginx get svc
# NAME                       TYPE           EXTERNAL-IP     PORT(S)
# ingress-nginx-controller   LoadBalancer   192.168.1.200   80:xxxxx,443:xxxxx
```

**Bây giờ bạn có thể trỏ domain về IP 192.168.1.200 để truy cập các ứng dụng!**

---

## 11. CÀI ĐẶT STORAGE

> **Storage trong K8s là gì?**
> Trong Docker Compose, bạn mount volume kiểu `-v ./data:/app/data`. Trong K8s, vì Pod có thể chạy trên BẤT KỲ máy nào, nên cần "shared storage" — nơi lưu trữ mà tất cả các máy đều truy cập được.

### Phương án A: Local Path Provisioner (Đơn giản nhất)

Dữ liệu lưu trên ổ cứng **của chính node đang chạy Pod**. Phù hợp cho dev/test.

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.26/deploy/local-path-storage.yaml

# Đặt làm StorageClass mặc định
kubectl patch storageclass local-path \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Phương án B: NFS Storage (Khuyến nghị cho Production)

Dữ liệu lưu trên 1 máy NFS Server riêng — tất cả node đều truy cập được.

#### B.1 — Cài NFS Server (trên máy NFS — 192.168.1.20)

```bash
sudo apt-get install -y nfs-kernel-server

# Tạo thư mục chia sẻ
sudo mkdir -p /srv/nfs/k8s
sudo chown nobody:nogroup /srv/nfs/k8s
sudo chmod 777 /srv/nfs/k8s

# Cấu hình export
echo "/srv/nfs/k8s 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)" | \
  sudo tee -a /etc/exports

sudo exportfs -rav
sudo systemctl restart nfs-kernel-server
```

#### B.2 — Cài NFS Client (trên TẤT CẢ các node K8s)

```bash
sudo apt-get install -y nfs-common
```

#### B.3 — Cài NFS Provisioner trong K8s (trên Master)

```bash
# Cài Helm (package manager cho K8s)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Thêm repo và cài NFS provisioner
helm repo add nfs-subdir-external-provisioner \
  https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-subdir-external-provisioner \
  nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.1.20 \
  --set nfs.path=/srv/nfs/k8s \
  --set storageClass.defaultClass=true
```

---

## 12. CHUYỂN ĐỔI DOCKER COMPOSE SANG KUBERNETES

Đây là phần quan trọng nhất! Tôi sẽ lấy ví dụ cụ thể.

### 12.1 Ví dụ Docker Compose gốc

```yaml
# docker-compose.yml
version: '3.8'
services:
  webapp:
    image: nginx:latest
    ports:
      - "80:80"
    environment:
      - APP_ENV=production
      - DB_HOST=database
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - database
    restart: always

  database:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=secretpass123
      - MYSQL_DATABASE=myapp
    volumes:
      - db-data:/var/lib/mysql
    restart: always

volumes:
  db-data:
```

### 12.2 Chuyển đổi sang Kubernetes — Giải thích từng file

**Cấu trúc thư mục:**
```
myapp-k8s/
├── namespace.yaml          # Tạo "không gian" riêng cho app
├── configmap.yaml          # Biến môi trường thường
├── secret.yaml             # Biến môi trường nhạy cảm (password)
├── mysql-deployment.yaml   # Database
├── mysql-service.yaml      # Mở cổng cho database
├── mysql-pvc.yaml          # Yêu cầu lưu trữ cho database
├── webapp-deployment.yaml  # Ứng dụng web
├── webapp-service.yaml     # Mở cổng cho web
└── webapp-ingress.yaml     # Cấu hình domain/URL
```

#### File 1: namespace.yaml — Tạo không gian riêng

```yaml
# namespace.yaml
# Namespace giống như "thư mục" trong K8s — giúp tách biệt các ứng dụng
# Ví dụ: app A ở namespace "myapp", app B ở namespace "another-app"
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
```

**So sánh Compose:** Docker Compose không có khái niệm này. Tất cả container chạy chung 1 "không gian".

#### File 2: configmap.yaml — Biến môi trường

```yaml
# configmap.yaml
# ConfigMap = nơi lưu các cấu hình KHÔNG nhạy cảm
# Giống phần "environment:" trong Docker Compose
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
  namespace: myapp          # Thuộc namespace "myapp"
data:
  APP_ENV: "production"
  DB_HOST: "mysql-service"  # Tên Service của MySQL (thay cho container name)
```

**So sánh Compose:** Giống `environment:` nhưng tách riêng ra file, có thể dùng lại cho nhiều Deployment.

#### File 3: secret.yaml — Mật khẩu, key nhạy cảm

```yaml
# secret.yaml
# Secret = nơi lưu thông tin nhạy cảm (password, API key, certificate)
# Dữ liệu được mã hóa base64
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: myapp
type: Opaque
stringData:                    # stringData = tự động encode base64
  MYSQL_ROOT_PASSWORD: "secretpass123"
  MYSQL_DATABASE: "myapp"
```

**So sánh Compose:** Giống `environment:` nhưng dành cho thông tin nhạy cảm. Trong Compose bạn hay dùng file `.env` — nhưng nó lưu dạng plain text. K8s Secret an toàn hơn (mặc dù mặc định chỉ là base64, bạn có thể kết hợp với Vault để mã hóa thực sự).

> **Tạo Secret bằng command (cách khác):**
> ```bash
> kubectl create secret generic mysql-secret \
>   --from-literal=MYSQL_ROOT_PASSWORD=secretpass123 \
>   --from-literal=MYSQL_DATABASE=myapp \
>   -n myapp
> ```

#### File 4: mysql-pvc.yaml — Yêu cầu lưu trữ

```yaml
# mysql-pvc.yaml
# PVC (PersistentVolumeClaim) = "Đơn đặt hàng" ổ cứng
# Bạn nói: "Tôi cần 10GB ổ cứng" → K8s tự tìm và cấp cho bạn
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data
  namespace: myapp
spec:
  accessModes:
    - ReadWriteOnce    # Chỉ 1 node được ghi cùng lúc
  resources:
    requests:
      storage: 10Gi    # Yêu cầu 10GB
  # storageClassName: nfs-client   # Bỏ comment nếu dùng NFS
```

**So sánh Compose:** Giống `volumes: db-data:` nhưng có thêm khả năng chỉ định dung lượng, kiểu lưu trữ.

#### File 5: mysql-deployment.yaml — Chạy MySQL

```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: myapp
  labels:
    app: mysql           # Label = "nhãn dán" để K8s nhận biết
spec:
  replicas: 1            # Chạy 1 bản (database thường chỉ 1)
  selector:
    matchLabels:
      app: mysql         # Deployment quản lý Pod có label "app: mysql"
  template:              # "Khuôn mẫu" để tạo Pod
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0             # Giống "image:" trong Compose
        ports:
        - containerPort: 3306        # Giống "ports:" bên trong container
        envFrom:
        - secretRef:
            name: mysql-secret       # Lấy biến từ Secret
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql   # Giống mount path trong Compose
        resources:                    # KHÁC Compose: giới hạn tài nguyên
          requests:
            cpu: "250m"              # Tối thiểu 0.25 CPU
            memory: "512Mi"          # Tối thiểu 512MB RAM
          limits:
            cpu: "1"                 # Tối đa 1 CPU
            memory: "1Gi"            # Tối đa 1GB RAM
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-data       # Gắn PVC đã tạo ở trên
```

**Giải thích cấu trúc Deployment:**

```
Deployment (Người quản lý)
  └── ReplicaSet (Đảm bảo đủ số bản copy)
       └── Pod (Đơn vị nhỏ nhất chạy container)
            └── Container (Ứng dụng thực sự — giống container Docker)
```

#### File 6: mysql-service.yaml — Mở cổng nội bộ

```yaml
# mysql-service.yaml
# Service = "Số điện thoại nội bộ" để các Pod khác gọi đến MySQL
apiVersion: v1
kind: Service
metadata:
  name: mysql-service     # Tên này được dùng làm "hostname" nội bộ
  namespace: myapp
spec:
  selector:
    app: mysql            # Chuyển traffic đến Pod có label "app: mysql"
  ports:
  - port: 3306            # Port mà Service lắng nghe
    targetPort: 3306      # Port của container bên trong Pod
  type: ClusterIP         # Chỉ truy cập được từ BÊN TRONG cluster
```

**So sánh Compose:** Trong Compose, container tự tìm nhau qua tên service (ví dụ: `database`). Trong K8s, bạn cần tạo Service — webapp sẽ kết nối MySQL qua `mysql-service:3306` hoặc `mysql-service.myapp.svc.cluster.local:3306`.

#### File 7: webapp-deployment.yaml — Chạy Web App

```yaml
# webapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: myapp
spec:
  replicas: 2              # Chạy 2 bản copy (High Availability!)
  selector:                # Không có trong Compose
    matchLabels:
      app: webapp
  strategy:                # Chiến lược cập nhật — không có trong Compose!
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1    # Tối đa 1 Pod ngừng khi đang update
      maxSurge: 1          # Tối đa 1 Pod mới được tạo thêm khi update
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: webapp-config    # Lấy biến từ ConfigMap
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
        # Health Check — K8s tự kiểm tra app còn sống không
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10     # Đợi 10s sau khi start mới check
          periodSeconds: 5            # Check mỗi 5s
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 3
```

**Điểm khác biệt lớn so với Compose:**
- `replicas: 2` → Tự động chạy 2 bản, nếu 1 bản chết, K8s tạo bản mới
- `strategy: RollingUpdate` → Khi deploy version mới, K8s update từng Pod một → zero downtime
- `livenessProbe` → K8s tự kiểm tra sức khỏe app, nếu app treo → tự restart
- `resources` → Giới hạn CPU/RAM rõ ràng cho từng container

#### File 8: webapp-service.yaml — Mở cổng cho web

```yaml
# webapp-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: myapp
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP           # Dùng Ingress ở trước, không cần LoadBalancer
```

#### File 9: webapp-ingress.yaml — Cấu hình domain

```yaml
# webapp-ingress.yaml
# Ingress = "Bảng chỉ dẫn" khi request HTTP đến
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: myapp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com    # Domain của bạn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service   # Chuyển đến webapp-service
            port:
              number: 80
```

### 12.3 Triển khai (Deploy)

```bash
# Bước 1: Tạo namespace trước
kubectl apply -f namespace.yaml

# Bước 2: Tạo config và secret
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml

# Bước 3: Tạo storage
kubectl apply -f mysql-pvc.yaml

# Bước 4: Deploy database trước
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml

# Đợi MySQL sẵn sàng
kubectl -n myapp wait --for=condition=ready pod -l app=mysql --timeout=120s

# Bước 5: Deploy webapp
kubectl apply -f webapp-deployment.yaml
kubectl apply -f webapp-service.yaml
kubectl apply -f webapp-ingress.yaml

# HOẶC deploy tất cả cùng lúc (nếu bạn tin tưởng thứ tự không quan trọng)
# kubectl apply -f myapp-k8s/
```

### 12.4 Kiểm tra trạng thái

```bash
# Xem tất cả resource trong namespace myapp
kubectl -n myapp get all

# Xem chi tiết Pod
kubectl -n myapp get pods -o wide

# Xem logs (giống docker logs)
kubectl -n myapp logs -f deployment/webapp

# Xem logs của 1 Pod cụ thể
kubectl -n myapp logs <pod-name>

# Vào bên trong container (giống docker exec)
kubectl -n myapp exec -it <pod-name> -- /bin/bash

# Xem events (rất hữu ích khi debug)
kubectl -n myapp get events --sort-by='.lastTimestamp'
```

### 12.5 Bảng chuyển đổi lệnh Docker ↔ Kubernetes

| Mục đích | Docker / Docker Compose | Kubernetes (kubectl) |
|----------|------------------------|---------------------|
| Xem container/Pod | `docker ps` | `kubectl get pods` |
| Xem logs | `docker logs <name>` | `kubectl logs <pod>` |
| Vào container | `docker exec -it <name> bash` | `kubectl exec -it <pod> -- bash` |
| Deploy | `docker-compose up -d` | `kubectl apply -f .` |
| Dừng | `docker-compose down` | `kubectl delete -f .` |
| Scale | `docker-compose up --scale web=3` | `kubectl scale deploy webapp --replicas=3` |
| Xem network | `docker network ls` | `kubectl get svc` |
| Xem volume | `docker volume ls` | `kubectl get pvc` |
| Build image | `docker build -t myapp .` | (Build ngoài cluster, push lên registry) |
| Xem tài nguyên | `docker stats` | `kubectl top pods` |

---

## 13. CÔNG CỤ QUẢN LÝ & MONITORING

### 13.1 Kubernetes Dashboard (Web UI)

```bash
# Cài Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Tạo admin user
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Lấy token đăng nhập
kubectl -n kubernetes-dashboard create token admin-user

# Mở proxy để truy cập
kubectl proxy
# Truy cập: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### 13.2 K9s — Terminal UI (Khuyến nghị!)

```bash
# Cài K9s — giao diện terminal cực kỳ tiện lợi
# Download từ: https://github.com/derailed/k9s/releases
# Hoặc:
snap install k9s

# Chạy
k9s
```

K9s giống như "Task Manager" cho Kubernetes — bạn có thể xem, filter, xóa, xem log Pod chỉ bằng phím tắt.

### 13.3 Lens — Desktop GUI

Tải Lens Desktop tại: https://k8slens.dev
Import file `~/.kube/config` vào Lens để quản lý cluster bằng giao diện đồ họa.

### 13.4 Monitoring với Prometheus + Grafana (Tùy chọn)

```bash
# Cài bằng Helm
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123
```

---

## 14. CHECKLIST & TROUBLESHOOTING

### 14.1 Checklist hoàn thành

- [ ] Tất cả node trạng thái **Ready**: `kubectl get nodes`
- [ ] Tất cả system Pod đang **Running**: `kubectl get pods -n kube-system`
- [ ] CNI đã cài và hoạt động (Flannel/Calico)
- [ ] MetalLB đã cấp IP cho Ingress Controller
- [ ] Storage provisioner hoạt động (local-path hoặc NFS)
- [ ] Có thể tạo Pod và truy cập qua Ingress

### 14.2 Lỗi thường gặp & Cách xử lý

#### 🔴 Node ở trạng thái NotReady

```bash
# Kiểm tra kubelet
sudo systemctl status kubelet
sudo journalctl -u kubelet -f

# Nguyên nhân phổ biến:
# 1. CNI chưa cài → Cài Flannel/Calico
# 2. Swap chưa tắt → swapoff -a
# 3. containerd bị lỗi → systemctl restart containerd
```

#### 🔴 Pod ở trạng thái CrashLoopBackOff

```bash
# Xem logs
kubectl -n <namespace> logs <pod-name> --previous

# Xem chi tiết events
kubectl -n <namespace> describe pod <pod-name>

# Nguyên nhân phổ biến:
# 1. Image sai tên hoặc không tồn tại
# 2. Thiếu biến môi trường
# 3. Port conflict
# 4. Không đủ tài nguyên (CPU/RAM)
```

#### 🔴 Pod ở trạng thái Pending

```bash
kubectl -n <namespace> describe pod <pod-name>

# Nguyên nhân phổ biến:
# 1. Không đủ tài nguyên trên node → Giảm requests hoặc thêm node
# 2. PVC không bind được → Kiểm tra StorageClass
# 3. Node có taint mà Pod không có toleration
```

#### 🔴 Service không thể truy cập

```bash
# Kiểm tra endpoint
kubectl -n <namespace> get endpoints <service-name>

# Nếu ENDPOINTS trống → selector không match với Pod label
# Kiểm tra label:
kubectl -n <namespace> get pods --show-labels
```

### 14.3 Lệnh debug hữu ích

```bash
# Tổng quan cluster
kubectl cluster-info
kubectl get componentstatuses

# Xem resource usage
kubectl top nodes
kubectl top pods -n <namespace>

# Debug networking
kubectl run debug --image=busybox -it --rm -- sh
# Trong container debug:
# nslookup mysql-service.myapp.svc.cluster.local
# wget -qO- http://webapp-service.myapp.svc.cluster.local

# Export cấu hình hiện tại (backup)
kubectl get all -n myapp -o yaml > backup-myapp.yaml
```

---

## PHỤ LỤC: SƠ ĐỒ TỔNG THỂ

```
                    Internet / Người dùng
                           │
                           ▼
                  ┌────────────────┐
                  │   MetalLB IP   │  192.168.1.200
                  │  (Virtual IP)  │
                  └───────┬────────┘
                          │
                  ┌───────▼────────┐
                  │    Ingress     │  Nginx Ingress Controller
                  │   Controller   │  Định tuyến theo domain/path
                  └───────┬────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐
    │ Service A  │  │ Service B  │  │ Service C  │
    │ (ClusterIP)│  │ (ClusterIP)│  │ (ClusterIP)│
    └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
          │               │               │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐
    │  Pod(s)   │  │  Pod(s)   │  │  Pod(s)   │
    │ webapp x2 │  │   api x3  │  │ mysql x1  │
    └───────────┘  └───────────┘  └─────┬─────┘
                                        │
                                  ┌─────▼─────┐
                                  │    PVC     │
                                  │ (NFS/Local)│
                                  └───────────┘
```

---

> 💡 **Mẹo cuối cùng:** Bắt đầu với 1 ứng dụng đơn giản nhất (ví dụ: nginx) để làm quen. Khi đã thoải mái rồi, hãy chuyển dần các service từ Docker Compose sang. Không cần chuyển tất cả cùng lúc!

> 📚 **Tài liệu tham khảo:**
> - Kubernetes Documentation: https://kubernetes.io/docs/
> - Kubernetes the Hard Way: https://github.com/kelseyhightower/kubernetes-the-hard-way
> - K8s Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/







  
                                                                                                                                                                                                                                                                                                 


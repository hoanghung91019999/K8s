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
kubectl get pods --all-namespaces
```








  
                                                                                                                                                                                                                                                                                                 


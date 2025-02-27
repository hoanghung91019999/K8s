#### chuẩn bị môi trường ( toàn bộ sử dụng centos9)
    + 1 máy chủ master
    + 2 máy chủ worker
    + 1 máy chủ ansible
### Cấu hình hệ thống trên tất cả các node master và worker
- Cập nhật hệ thống và Tắt SELinux
```
dnf update -y
setenforce 0
sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```
- Tắt Swap
- Kubelet kiểm tra trạng thái của hệ thống và nếu phát hiện Swap không bị tắt, nó sẽ báo lỗi và không chạy.
- Kubernetes quản lý tài nguyên CPU và RAM bằng cách lên lịch (schedule) các pod lên các node dựa trên tài nguyên có sẵn.
- Nếu hệ thống có Swap, Kubernetes có thể hiểu nhầm rằng node vẫn có đủ RAM, dẫn đến việc sắp xếp pod lên các node bị quá tải (out of memory - OOM).
```
swapoff -a
sed -i '/swap/d' /etc/fstab
```
- Bật các module cần thiết 
```
modprobe overlay
modprobe br_netfilter
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
- Cấu hình sysctl
```
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system
```
- Cài đặt Python
```
dnf install -y python3 python3-pip
```

### Cài đặt Kubespray trên máy Ansible
- Cài đặt Ansible và python , git
```
dnf install -y python3 python3-pip
dnf install -y epel-release
dnf install -y ansible git
```
- chú ý version của ansible
- Clone repo Kubespray
```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
git checkout release-2.22  # Chọn phiên bản phù hợp với K8s
```
- Cài đặt các Python dependencies
- Kubespray là một bộ công cụ Ansible-based để triển khai Kubernetes. Do đó, nó phụ thuộc vào Python và các thư viện Python để thực hiện các tác vụ tự động hóa trên các node. Nếu không cài đặt đầy đủ dependencies, quá trình triển khai có thể bị lỗi.
- Các module Python hỗ trợ Kubespray hoạt động đúng
    + Jinja2 (dùng để render các template YAML cho Kubernetes)
    + PyYAML (để xử lý file YAML trong Ansible)
    + netaddr (để xử lý địa chỉ IP và subnet)
    + jsonpatch (dùng để sửa đổi cấu hình Kubernetes)
```
pip3 install -r requirements.txt
```
- Tạo inventory cho cluster
```
cp -rfp inventory/sample inventory/mycluster
```

- Cấu hình Inventory
```
declare -a IPS=(điền IP của master và workrer)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```
  + kiểm tra trong file inventory/mycluster/hosts.yaml xem các IP đã được thay đổi chưa
- **lưu ý : các node phải đúng với tên của host master và woker**
### cấu hình ssh không pass cấu hình trên máy ansible
```
ssh-keygen -t rsa -b 4096
ssh-copy-id root@IP_master
ssh-copy-id root@IP_woker1
ssh-copy-id root@IP_woker2
```

### Chạy Ansible Playbook để cài đặt K8s
- chạy lệnh sau để cài đặt
```
ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml
```
## lưu ý 
- quá trình chạy sẽ mất khoảng 10-15p
- sau khi chạy xong chạy lệnh sau để kiểm tra
```
kubectl get nodes
```
- nếu hiện ra thông tin master và worker là cài đặt thành công
- nếu không có gì mà log ansible báo done có thể là do k8s đã cài đặt thành công nhưng chưa thiết lập đúng đường dẫn cho kubectl
- chạy lệnh
```
which kubectl
```
- nếu có out put là đường dẫn chứng tỏ k8s đã cài đặt thành công nhưng chưa thiết lập đúng đường dẫn
- kiểm tra kubectl có tồn tại trên hệ thống không 
```
find / -name kubectl 2>/dev/null
```
- Nếu có output, nghĩa là Kubectl đã được cài nhưng không có trong $PATH. Hãy thêm đường dẫn Kubectl vào $PATH.
```
export PATH=$PATH:/path/to/kubectl-directory
```
- kiểm tra lại
```
kubectl cluster-info
kubectl get nodes
```



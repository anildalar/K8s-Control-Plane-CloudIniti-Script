# K8s-Control-Plane-CloudIniti-Script
K8s-Control-Plane-CloudIniti-Script

#cloud-config
hostname: master-node-1

package_update: true
package_upgrade: true
packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - containerd

write_files:
  - path: /etc/modules-load.d/k8s.conf
    content: |
      br_netfilter
  - path: /etc/sysctl.d/k8s.conf
    content: |
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
  - path: /etc/sysctl.d/99-kubernetes-ip-forward.conf
    content: |
      net.ipv4.ip_forward=1

runcmd:
  - sudo hostname master-node-1
  - sudo apt update && sudo apt upgrade -y
  - sudo swapoff -a
  - sudo sed -i '/ swap / s/^/#/' /etc/fstab
  - echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  - curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  - sudo apt-get update -y
  - sudo apt-get install -y kubelet kubeadm kubectl
  - sudo apt-mark hold kubelet kubeadm kubectl
  - sudo systemctl enable containerd
  - sudo systemctl start containerd
  - sudo systemctl restart containerd
  - sudo systemctl enable kubelet
  - sudo systemctl start kubelet
  - containerd config default | sudo tee /etc/containerd/config.toml
  - sudo systemctl restart containerd
  - sudo systemctl enable containerd
  - sudo systemctl status containerd
  - kubelet --version
  - kubeadm version
  - kubectl version --client
  - sudo modprobe br_netfilter
  - sudo sysctl --system
  
  # Get the public IP and add it to /etc/hosts
  - PUBLIC_IP=$(curl -s ifconfig.me)
  - echo "$PUBLIC_IP master-node-1" | sudo tee -a /etc/hosts > /dev/null
  
  # Initialize Kubernetes with kubeadm
  - sudo kubeadm init --control-plane-endpoint=master-node-1 --pod-network-cidr=192.168.0.0/16
  - mkdir -p $HOME/.kube
  - sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  - sudo chown $(id -u):$(id -g) $HOME/.kube/config
  - kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
  - kubectl get nodes

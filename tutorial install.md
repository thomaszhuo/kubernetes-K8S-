##
INSTALLL KUBERNETES with UBUNTU (on VM)
##

==========================================================================================================================================================================================================================================
konfigurasi hosts dan IP
```bash
root@k8s-master:/# cat /etc/hosts
127.0.0.1 localhost
#127.0.1.1 k8s-master

#K8S
10.10.10.123 k8s-master
10.10.10.88 k8s-worker1
10.10.10.89 k8s-worker2
```

menonaktifkan Swap
```bash
sudo swapoff -
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```


IPV4 untuk Bridge disemua Node
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

mengaktifkan konfigurasi tanpa restart
```bash
sudo sysctl --system
```


install yang dibutuhkan
```bash
apt-get install -y apt-transport-https ca-certificates curl
```

install docker
```bash
{
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 }
apt install docker-ce=5:20.10.20~3-0~ubuntu-jammy docker-ce-cli=5:20.10.20~3-0~ubuntu-jammy containerd.io docker-buildx-plugin docker-compose-plugin
```

install kubernetes
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.26/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

update repo
```bash
sudo apt-update
```

install perlengkapan kube
dengan spesifik versi
```bash
apt install -y kubelet=1.26.5-00 kubeadm=1.26.5-00 kubectl=1.26.5-00
```
atau tanpa spesifik versi
```bash
sudo apt-get install -y kubelet kubeadm kubectl
```

konfig paket
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Konfigurasi Contained agar bisa digunakan oleh Kubernetes
```bash
root@k8s-master:/# sudo mkdir /etc/containerd
root@k8s-master:/# sudo sh -c "containerd config default > /etc/containerd/config.toml"
root@k8s-master:/# sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
root@k8s-master:/# sudo systemctl restart containerd.service
root@k8s-master:/# sudo systemctl restart kubelet.service
root@k8s-master:/# sudo systemctl enable kubelet.service
```
_note : lakukan konfigurasi diatas disemua node (master dan worker)_

==========================================================================================================================================================================================================================================
_note : lakukan konfigurasi didibawah hanya pada node master._

Konfigurasi Kubernetes Cluster

```bash
sudo kubeadm config images pull
```
```bash
sudo kubeadm init --pod-network-cidr=10.10.0.0/16
```
atau tambahkan 
```bash
 --ignore-preflight-errors=all --v=5
```

Hingga keluar output seperti ini
#
root@k8s-master:/# 
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
#

==========================================================================================================================================================================================================================================
Setting CALICO

```bash
root@k8s-master:/home/kube-K8S#kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

root@k8s-master:/home/kube-K8S# curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   824  100   824    0     0    727      0  0:00:01  0:00:01 --:--:--   727

root@k8s-master:/home/kube-K8S# sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.10.0.0\/16/g' custom-resources.yaml

root@k8s-master:/home/kube-K8S# kubectl create -f custom-resources.yaml

installation.operator.tigera.io/default created

apiserver.operator.tigera.io/default created
```
==========================================================================================================================================================================================================================================

untuk join Cluster pada Node Master/Control Pane
```bash
kubeadm join 10.10.10.123:6443 --token (token kamu) \
        --discovery-token-ca-cert-hash sha256:(sesuaikan) 
```

==========================================================================================================================================================================================================================================

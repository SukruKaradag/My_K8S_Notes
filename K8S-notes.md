# Kubernetes

## installation

### Kubectl
```bash
# Download the latest release with the command:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Validate the binary (optional)
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

# Validate the kubectl binary against the checksum file:
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#Test to ensure the version you installed is up-to-date:
kubectl version --client

# use this for detailed view of version:
kubectl version --client --output=yaml   

```

### Minukube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### kubeadm

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
### Kubernetes Cluster

```bash

sudo kubeadm config images pull
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<ip> --control-plane-endpoint=<ip>

#Calico install
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```
## Commands

```bash
kubectl get nodes # list nodes

```

## Notes
- ```The connection to the server x.x.x.:6443 was refused - did you specify the right host or port?``` hatası içiN:
```bash
sudo systemctl stop kubelet
sudo systemctl start kubelet
strace -eopenat kubectl version
```
- Düşük RAM ve CPU hatalarını ignore etmek için:
```sudo kubeadm init --ignore-preflight-errors="Mem" --ignore-preflight-errors="NumCPU"  --apiserver-advertise-address=<ec2-private-ip> --pod-network-cidr=10.244.0.0/16```

**kubectl config dosyası:**
- kubectl aracı bağlanacağı Kubernetes cluster bilgilerine config dosyaları aracılığıyla erişir.
- Config dosyasının içerisinde Kubernetes cluster bağlantı bilgilerini ve oraya bağlanırken kullanmak istediğimiz kullanıcıları belirtiriz.
- Daha sonra bu bağlantı bilgileri ve kullanıcıları ve ek olarak namespace bilgilerini de oluşturarak context'ler yaratırız.
- Kubectl varsayılan olarak$HOME/.kube/altındaki config isimli dosyaya bakar.
- Kubectl varsayılan olarak$HOME/.kube/altındaki config dosyasına bakar ama bunu KUBECONFIG enviroment variable değerini değiştirerek güncelleyebiliriz.
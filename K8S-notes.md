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
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.29.158.170 --control-plane-endpoint=172.29.158.170

#Calico install
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```
## Commands

```bash

kubectl explain "obje_adı" # objeleri açıklar.

kubectl "command" --help # Command'ı açıklar.

kubectl describe pods "pod_name" # belirtilen pod hakkında bilgi verir.

kubectl get nodes # list nodes

kubectl config get-contexts #

kubectl config current-context

kubectl config use-context "context_name" # Context değiştirmek için

kubectl cluster-info

kubectl delete pods "pod_name" # Delete pods

kubectl get pods -A -o "output_type" # Allowed formats are: custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-as-json,jsonpath-file,name,template,templatefile,wide,yaml

kubectl get pods -w # "-w" ile girilen kod canlı olarak izlenir.

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

- image çekme işleminde bir hata ile karşılaşılırsa k8s o image'ı belli aralıklarla çekmeye devam eder.

**kubectl config dosyası:**
- kubectl aracı bağlanacağı Kubernetes cluster bilgilerine config dosyaları aracılığıyla erişir.
- Config dosyasının içerisinde Kubernetes cluster bağlantı bilgilerini ve oraya bağlanırken kullanmak istediğimiz kullanıcıları belirtiriz.
- Daha sonra bu bağlantı bilgileri ve kullanıcıları ve ek olarak namespace bilgilerini de oluşturarak context'ler yaratırız.
- Kubectl varsayılan olarak$HOME/.kube/altındaki config isimli dosyaya bakar.
- Kubectl varsayılan olarak$HOME/.kube/altındaki config dosyasına bakar ama bunu KUBECONFIG enviroment variable değerini değiştirerek güncelleyebiliriz.

- Birden fazla cluster oluşturulabilir. Ayrım `contexts`'ler ile yapılır.


## Pods

- Pod'lar,Kubernetes'te oluşturabileceğiniz ve yönetebileceğiniz en küçük birimleridir.
                  
- Pod'lar bir ya da daha fazla container barındırabilir.Ama çoğu durumda pod tek container barındırır. 

- Her pod'un eşsiz bir id'si "uid" bulunur.

- Her pod eşsiz bir ip adresine sahiptir.

- Ayni pod içerisindeki containerlar aynı node üstünde çalışıtırılır ve bu containerlar birbirleriyle **localhost** üstünden haberleşebilirler.

### Pod lifecycle
- *RestartPolicy*:
  - Always
  - On-failure
  - Never

1- Imageerrorpod/ImagePullBackOff:
- Image pull işleminde hata ile karşılaşılması durumu.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: firstpod
  labels:
    app: frontend
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```
2- succeededpod
- Başarılı pod işlemi.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: succeededpod
spec:
  restartPolicy: Never
  containers:
  - name: succeedcontainer
    image: ubuntu:latest
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 20']
```

3- failedpod


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: failedpod
spec:
  restartPolicy: Never
  containers:
  - name: failcontainer
    image: ubuntu:latest
    command: ['sh', '-c', 'abc']
```

4- crashloopbackpod
- Conrainer'ın her çöküşünde yeniden restart edilmesidir. Zamanla restart aralıkları artar.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crashloopbackpod
spec:
  restartPolicy: Always
  containers:
  - name: crashloopbackcontainer
    image: ubuntu:latest
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 20']
```

### Pod Commands
```bash
kubectl run "pod_name" --image=nginx --restart=Never

kubectl delete pods "pod_name"

kubectl logs "pod_name" # for logs

kubectl logs -f "pod_name" # interactive logs

kubectl exec "pod_name" -- "command" # Belirtilen komut belirtilen pod'un içinde çalıştırılır.

kubectl exec -it "pod_name" -- /bin/sh # Pod'a bağlanmak için

kubectl apply -f objectbasetemplate.yaml #Declarative

kubectl edit pods "pod_name" # Declarative file'ı çıktı olarak verir.

k exec -it "pod_name" -c "container_name" -- /bin/bash` #Pod içindeki istenilen container'a bağlanmak için

```

#### Multicontainer Pods
- İstenilen container'a bağlanmak için: `kubectl exec -it "pod_name" -c "container_name" -- /bin/bash`
- Loglarına bakma için: `kubectl logs -f "pod_name" -c "container_name"`
- port forwarding için: `kubectl port-forward pod/"pod_name" 8080:80`
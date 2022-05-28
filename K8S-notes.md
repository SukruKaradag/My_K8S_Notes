## installation
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

## Notes
- ```The connection to the server x.x.x.:6443 was refused - did you specify the right host or port?``` hatası içiN:
```bash
sudo systemctl stop kubelet
sudo systemctl start kubelet
strace -eopenat kubectl version
```
- Düşük RAM ve CPU hatalarını ignore etmek için:
```sudo kubeadm init --ignore-preflight-errors="Mem" --ignore-preflight-errors="NumCPU"  --apiserver-advertise-address=<ec2-private-ip> --pod-network-cidr=10.244.0.0/16```
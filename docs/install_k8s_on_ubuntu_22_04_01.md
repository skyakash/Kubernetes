**Create a 3 node Kubernetes cluster on a Ubuntu 22.04 VMs** 

1. Create 3 VMs with Ubuntu 22.04.01 LTS Desktop edition, Install OpenSSH server during VM creation, username: akash, machine names: k8smaster, k8sworker1, k8sworker2
2. Note IP address of VMs
```
sudo apt-get install openssh-server -y
ip a
```

**Use Mobaxterm, log into machines and use MultiExec option to execute same commands on all 3 machines together. To paste content on all terminals use ctrl+shift+ins**


3. Update the VM
```
sudo apt update
sudo apt -y upgrade
```
4. edit /etc/hosts and add entry of nodes
```
192.168.68.123 k8smaster
192.168.68.124 k8sworker1
192.168.68.125 k8sworker2
```

5. Disable firewall 
```
sudo ufw disable
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

6. Install containerd
```
sudo apt install apt-transport-https curl
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io  
```

7. Create containerd configuration
```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Edit /etc/containerd/config.toml and set SystemdCgroup = true
```
sudo systemctl restart containerd
```

8. Forwarding IPv4 and letting iptables see bridged traffic
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


lsmod | grep br_netfilter
lsmod | grep overlay
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

9. Installing kubeadm, kubelet and kubectl
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install kubeadm kubelet kubectl kubernetes-cni

```
10. Now run following commands only on master
```
sudo kubeadm init --pod-network-cidr=10.11.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
11. Copy join command 

12. Install calico, Download calico.yaml from https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/calico.yaml
 ```
 kubectl apply -f calico.yaml
 ```

13. Run following command only on workers to join then in cluster
```
kubeadm join 192.168.68.123:6443 --token bgycye.pbwrq9ehcgyjy154 \
        --discovery-token-ca-cert-hash sha256:6c20296d8a81770c1d283fe4a0a8a1e820ed164ebc316693ce45f71cc476c4e8
        
kubectl get po -A --watch
kubectl get nodes -o wide
        
```
 

Links:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

https://serverfault.com/questions/1118051/failed-to-run-kubelet-validate-service-connection-cri-v1-runtime-api-is-not-im
https://docs.docker.com/engine/install/ubuntu/
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker

Set container runtime on node
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock config --set runtime-endpoint=unix:///run/containerd/containerd.sock

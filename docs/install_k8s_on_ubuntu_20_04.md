**Create a 3 node Kubernetes cluster on a Ubuntu 20.04 VMs**

1. Create 3 VMs with Ubuntu 20.04, username: ubuntu, machine names: master, worker1, worker2
2. update the VM
3. Install openssh so that we can connect to nodes via ssh
```
sudo apt-get update
sudo apt-get install openssh-server -y
```
4. Note IP address of VMs
```
ip a
```

**Use Mobaxterm, log into machines and use MultiExec option to execute same commands on all 3 machines together**

5. edit /etc/hosts and add entry of nodes
```
192.168.68.129 master
192.168.68.130 worker1
192.168.68.131 worker2
```

6. Disable firewall 
```
sudo ufw disable
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```
7. Install containerd
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
8. Setup required sysctl params, these persist across reboots.
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

9. Apply sysctl params without reboot
```
sudo sysctl --system

sudo apt-get install containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

10. Letting iptables see bridged traffic
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

11. Installing kubeadm, kubelet and kubectl
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
12. Now run following commands only on master
```
sudo kubeadm init
```
13. copy join command 

14. install weave net 
 ```
 kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
 ```

15. Run following command only on workers to join then in cluster
```
kubeadm join 192.168.68.129:6443 --token k94ju3.l4drm9e700p5uscy --discovery-token-ca-cert-hash sha256:b887ebaf510b564979b3cd07bc2b8e2e7f55f2c389c6487c78712f86eb1c6211
```
 

Links:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd


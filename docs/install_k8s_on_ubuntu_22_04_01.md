**Create a 3 node Kubernetes cluster on a Ubuntu 20.04 VMs** -- Work in progress - DO NOT USE ----

1. Create 3 VMs with Ubuntu 22.04.01 LTS Desktop edition, Install OpenSSH server during VM creation, username: akash, machine names: k8smaster, k8sworker1, k8sworker2
2. Note IP address of VMs
```
sudo apt-get install openssh-server -y
ip a
```

**Use Mobaxterm, log into machines and use MultiExec option to execute same commands on all 3 machines together**

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


cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```


10. Installing kubeadm, kubelet and kubectl
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install kubeadm kubelet kubectl kubernetes-cni

```
11. Now run following commands only on master
```
sudo kubeadm init
```
12. copy join command 

13. install weave net 
 ```
 kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
 ```

14. Run following command only on workers to join then in cluster
```
kubeadm join 192.168.68.129:6443 --token k94ju3.l4drm9e700p5uscy --discovery-token-ca-cert-hash sha256:b887ebaf510b564979b3cd07bc2b8e2e7f55f2c389c6487c78712f86eb1c6211
```
 

Links:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

https://serverfault.com/questions/1118051/failed-to-run-kubelet-validate-service-connection-cri-v1-runtime-api-is-not-im
https://docs.docker.com/engine/install/ubuntu/
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker


# Install Vanilla Kubernetes


OS : Debian 11
Current version : 1.24


```
sudo apt update
sudo apt upgrade

sudo apt -y install apt-transport-https gnupg2 curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
curl -s https://download.docker.com/linux/debian/gpg | sudo apt-key add -


echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker-ce.list

sudo apt update

sudo apt -y install containerd.io
sudo apt -y install kubeadm kubelet kubectl

```


prepull images in a local repo
```

docker pull k8s.gcr.io/kube-apiserver:v1.24.3
docker pull k8s.gcr.io/kube-controller-manager:v1.24.3
docker pull k8s.gcr.io/kube-scheduler:v1.24.3
docker pull k8s.gcr.io/kube-proxy:v1.24.3
docker pull k8s.gcr.io/pause:3.7
docker pull k8s.gcr.io/etcd:3.5.3-0
docker pull k8s.gcr.io/coredns/coredns:v1.8.6


docker tag k8s.gcr.io/kube-apiserver:v1.24.3 registry.work.lan/kube-apiserver:v1.24.3
docker tag k8s.gcr.io/kube-controller-manager:v1.24.3 registry.work.lan/kube-controller-manager:v1.24.3
docker tag k8s.gcr.io/kube-scheduler:v1.24.3 registry.work.lan/kube-scheduler:v1.24.3
docker tag k8s.gcr.io/kube-proxy:v1.24.3 registry.work.lan/kube-proxy:v1.24.3
docker tag k8s.gcr.io/pause:3.7 registry.work.lan/pause:3.7
docker tag k8s.gcr.io/etcd:3.5.3-0 registry.work.lan/etcd:3.5.3-0
docker tag k8s.gcr.io/coredns/coredns:v1.8.6 registry.work.lan/coredns:v1.8.6


docker push registry.work.lan/kube-apiserver:v1.24.3
docker push registry.work.lan/kube-controller-manager:v1.24.3
docker push registry.work.lan/kube-scheduler:v1.24.3
docker push registry.work.lan/kube-proxy:v1.24.3
docker push registry.work.lan/pause:3.7
docker push registry.work.lan/etcd:3.5.3-0
docker push registry.work.lan/coredns:v1.8.6


sudo ctr images pull registry.work.lan/kube-apiserver:v1.24.3
sudo ctr images pull registry.work.lan/kube-controller-manager:v1.24.3
sudo ctr images pull registry.work.lan/kube-scheduler:v1.24.3
sudo ctr images pull registry.work.lan/kube-proxy:v1.24.3
sudo ctr images pull registry.work.lan/pause:3.7
sudo ctr images pull registry.work.lan/etcd:3.5.3-0
sudo ctr images pull registry.work.lan/coredns:v1.8.6
```


COnfigure container d

```

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system


sudo mkdir -p /etc/containerd


containerd config default | sudo tee /etc/containerd/config.toml

modify config.toml like this


    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.work.lan".tls]
          insecure_skip_verify = true

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugin."io.containerd.grpc.v1.cri".registry.mirrors."registry.work.lan"]
          endpoint = ["https://registry.work.lan"]


copy pem file of registry to /etc/ssl/certs/ on master


sudo systemctl restart containerd

sudo reboot now

sudo kubeadm config images pull --image-repository registry.work.lan 

sudo kubeadm init --image-repository registry.work.lan

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

kubectl label node master node-role.kubernetes.io/role=master

kubectl get po --all-namespaces
kubectl get nodes

```

# Sandboxing

```

vmcreate sandbox 4096 4 debian-10-genericcloud-amd64 50 40G debian10

```
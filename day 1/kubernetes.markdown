## Kubernetes setting

* Install kubeadm, kubelet, kubectl (master, worker node 둘 다)

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install -y kubelet=1.15.5-00 kubeadm=1.15.5-00 kubectl=1.15.5-00

# hold: 해당 패키지가 자동으로 설치, 업그레이드, 제거되지 않도록 설정
sudo apt-mark hold kubelet kubeadm kubectl

# bridge 네트워크를 통한 패킷이 iptables를 거치냐 안거치냐를 설정하는 부분.
# docker로 구축한 컨테이너들이 호스트 OS와 iptables 설정을 공유하도록 하는 설정이라 생각하면 될듯
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

* Create kubernetes cluster (master node only)
```bash
# 메모리 부족시 스왑하는 기능을 끔. 쿠버네티스가 제대로 작동하지 않는 이슈가 있음
sudo swapoff -a
# 이때 kubeadm join ... 하는 명령어가 출력됨. 이걸 워커노드들에서 실행해야 하므로 얻어둘 것.
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# taint: master node가 상태를 보고 스케쥴링하는 것
kubectl taint nodes --all node-role.kubernetes.io/master-
# $ node/johnnyapu untainted

# calico: Cloud network를 관리하는 툴
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

* Join in kubernetes cluster (worker nodes only)
```bash
sudo swapoff -a
sudo kubeadm join IP_MASTER_NODE:6443 --token TOKEN --discovery-token-ca-cert-hash sha256:HASH_VALUE
```

* Check kubernetes nodes
```bash
kubectl get nodes
```

* When the status is NOT READY...
```bash
# kubelet 상태 확인
sudo systemctl status kubelet
# kubelet 재시작
sudo systemctl restart kubelet
sudo systemctl enable kubelet

sudo systemctl status kubelet
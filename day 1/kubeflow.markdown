## Kubernetes storage class setting
* 쿠버네티스 내 동적 프로비저닝을 지원하는 스토리지 클래스가 필요
* Storage class는 Persistant Volume을 동적으로 프로비저닝하기 위한 다음 필드가 필요
  * provisioner
  * parameters
  * reclaimPolicy

* 아래는 provisioner의 설정이다.

## Provisioner for storageclass
* provisioner는 Volume plugin, internal provisioner가 필요하며, 우리는 nfs plugin과 local path provisioner를 활용할 것이다.

### Volume plugin
* 클라우드 서비스의 다양한 데이터베이스 서비스를 이용할 수 있음.
* 정석은 이 스토리지 서비스를 별도 서버로 구축하는 거지만 우리는 그냥 마스터노드에 얹자.
* 우리는 nfs를 이용한다.
  * nfs(Network File System): 한 서버의 디렉토리를 다른 서버에서 로컬 디렉토리처럼 사용할 수 있는 프로그램. 공유 폴더라고 생각하면 됨
```bash
sudo mkdir -p /mnt/storage/nfs_storage
sudo chmod -R 777 /mnt/storage/nfs_storage

sudo apt-get -y install nfs-common nfs-kernel-server rpcbind portmap

# Get internal ip of master, worker nodes
kubectl get node -o wide
```

* 아래 /etc/exports에 nfs를 위한 폴더를 등록하자
```txt
/mnt/storage/nfs_storage MASTER_NODE_IP(rw,insecure,sync,no_root_squash,no_subtree_check)
/mnt/storage/nfs_storage WORKER_NODE_IP(rw,insecure,sync,no_root_squash,no_subtree_check)
```
  * no_root_squash: 서버의 root 계정 권한을 클라이언트 root 계정이 공유
  * no_subtree_check: 공유 디렉토리 내 서브 디렉토리들은 공유하지 않음

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

* Install nfs client
```bash
sudo apt install nfs-common
```
* Test at client
```bash
mkdir test_mount
sudo mount SERVER_IP:/mnt/storage/nfs_storage test_mount
touch test_mount/test.txt
```

* Test at Server
```bash
ls /mnt/storage/nfs_storage
```

* Test clear (at Client)
```bash
umount test_mount
```

* Install Local path provisioner for dynamic provisioning
* Apply local path storage into kubectl
* To understand this, checkout
  1. Persistent Volume (PV): https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/
  2. Local Persistent Volume: https://kubernetes.io/blog/2018/04/13/local-persistent-volumes-beta/
  3. Local Path Provisioner: https://github.com/rancher/local-path-provisioner
* 간단하게 설명하면 stateful application을 위해 PV가 필요한데, 그냥 PV는 Cluster내 특정 node를 통해 접근할 수 밖에 없음. 그래서 느리니까 local PV가 필요했고, kubernetes 내에 LPV를 제공하기는 하는데 dynamic provisioning 지원이 안됨. 그래서 DP를 제공하는 Local Path Provisioner를 추가 적용하겠다는 스토리.

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

* Install helm
* Helm: Chart manager for kubernetes like apt
  * Chart: Package-like object for kubernetes. Helm package.
  * release: chart instance. it created when installing a chart
* It maybe cause a permittion issue
```bash
curl https://raw.githubusercontent.com/helm/helm/release-2.16/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

* Install nfs-client-provisioner
* 우리는 nfs 서버가 곧 master node 서버니까 같은데 설치해주면 된당
```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm install RELEASE_NAME --set nfs.server=SERVER_IP --set nfs.path=/mnt/storage/nfs_storage stable/nfs-client-provisioner
```

* Storage class setting
* 

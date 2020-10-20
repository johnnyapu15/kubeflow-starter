# docker setting

## docker data 영역 만들기

```bash
mkdir -p /mnt/storage/docker_data
```

## Install

```bash
sudo apt update

sudo apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] http://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

# Install docker community edition
sudo apt install -y docker-ce=5:18.09.9~3-0~ubuntu-bionic docker-ce-cli=5:18.09.9~3-0~ubuntu-bionic containerd.io

# Set docker daemon
sudo nano /etc/docker/daemon.json
```
```json
{
    "exec-opts":["native.cgroupdriver=systemd"],
    "log-driver":"json-file",
    "log-opts": {
        "max-size":"100m"
    },
    "data-root":"/mnt/storage/docker_data",
    "storage-driver":"overlay2"
}
```
## about 'storage-driver' for docker
* docker의 image와 container를 효율적으로 관리하기 위한 filesystem 일종. layer로 추상화하여 관리함. 
* 원본 image를 변경하지 않으면서, 거기에 변경되야 하는 container 구현체를 효율적으로 관리하는 filesystem. 이에 Union File System이라는 것을 도입한다. 이때 사용되는 프로그램의 종류를 'storage-driver'로 선언하는 것이다.
* almost about docker storage driver, joinc: https://www.joinc.co.kr/w/man/12/docker/storage
* docker docs: https://docs.docker.com/storage/storagedriver/


```
# Registry daemon service 
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl restart docker
# Check
sudo systemctl status docker
```
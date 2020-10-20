## Mount

```bash
# 마운트할 폴더 생성
sudo mkdir -p /mnt/storage

# 마운트 방법중 하나로, /etc/fstab에 마운트할 디스크 정보를 입력한다. 이때 UUID가 필요한데, blkid 명령을 통해 얻을 수 있다.

blkid /dev/xvdb1

sudo nano /etc/fstab

# UUID=got_UUID /mnt/storage ext4 defaults 0 0


# mount
sudo mount -a
sudo chmod 777 /mnt/storage

# check
df -h
```

## Create partition on disk and format

```bash
# Check disk list
sudo fdisk -l
# Create partition on /dev/xvdb
sudo fdisk /dev/xvdb


n
# default setting
w

# 파티션이 /dev/xvdb1으로 저장

# format

mkfs.ext4 /dev/xvdb1
```
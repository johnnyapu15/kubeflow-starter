## Add user
sudo는 특정 사용자가 슈퍼유저처럼 권한을 행사할 수 있도록 하는 프로그램으로, sudoers는 sudo를 이용할 수 있는 유저의 목록임.
```bash
user_name=''

adduser $user_name

sudo nano /etc/sudoers
```
### sudoer 예시
  user  host    operations
  root ALL=(ALL)  ALL
  $user_name  ALL=(ALL)   ALL
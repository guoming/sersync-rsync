# sersync + rsync 双向文件同步部署说明

## 1. Rsync 部署

### 1.1 安装 rsync
``` Shell
apt install rsync
```
### 1.2 创建 rsync 用户
``` Shell
useradd w -M  -s  /bin/nologin
```

### 1.3 配置 rsync
> vi /etc/rsyncd.conf 
```  INI
uid = w
gid = w
port = 873
use chroot = on
max connections = 200
timeout = 120
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
log file = /var/log/rsyncd.log

[rfs]
path = /mnt/rfs
read only = false
list = false
hosts allow = 192.168.0.0/24
hosts deny = 0.0.0.0/32
auth users = rsync_backup
secrets file = /etc/rsyncd.password
```

> rsync服务端需要的密码文件（密码：123456）
```  Shell
echo rsync_backup:123456>/etc/rsyncd.password
```

> rsync 客户端需要的密码文件（密码：123456）
```  Shell
echo 123456>/etc/rsyncd.client-password
```

> 设置密码文件权限
``` Shell
chmod 600 /etc/rsyncd.password 
chmod 600 /etc/rsyncd.client-password
chown w /etc/rsyncd.client-password ## 请确保和执行rsync的用户保持一致
```
> 创建需要同步的目录
``` Shell
mkdir /mnt/rfs
chmod 777 /mnt/rfs
```

> 测试同步
 ``` Shell
rsync -artuz -R /mnt/rfs/1 rsync_backup@192.168.109.180::rfs --password-file=/etc/rsyncd.client-password --port=873
rsync -artuz -R /mnt/rfs/ rsync_backup@192.168.109.129::rfs --password-file=/etc/rsyncd.client-password --port=873
rsync -rrtuz -R /mnt/rfs/ rsync_backup@192.168.109.128::rfs --password-file=/etc/rsyncd.client-password --port=873
```

## 1.3 启动 rsync
``` Shell
systemctl enable rsync.service
systemctl restart rsync.service
```
## 2. sersync 部署
### 2.1 下载sersync
git clone https://github.com/guoming/sersync-rsync.git
### 2.2 启动sersync
``` Shell
./bin/sersync -o ./conf/confxml.xml
```


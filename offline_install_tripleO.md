Offline Install tripleO
=======================

- 以下操作，都使用 root user
- 下載資料

```bash
wget http://172.22.104.1:4433/centos7mirror.tar.gz -O ~/centos_offline_repo.tar.gz
```

- 解壓縮

```bash
tar zxvf ~/centos_offline_repo.tar.gz -C ~
```

- 安裝下面套件

```bash

rpm -ivh ~/osprepo/base/Packages/libxml2-python-2.9.1-6.el7_2.3.x86_64.rpm
rpm -ivh ~/osprepo/base/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
rpm -ivh ~/osprepo/base/Packages/python-chardet-2.2.1-1.el7_1.noarch.rpm
rpm -ivh ~/osprepo/base/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
rpm -ivh ~/osprepo/base/Packages/deltarpm-3.6-3.el7.x86_64.rpm
rpm -ivh ~/osprepo/base/Packages/yum-utils-1.1.31-40.el7.noarch.rpm
rpm -ivh ~/osprepo/base/Packages/python-deltarpm-3.6-3.el7.x86_64.rpm
rpm -ivh ~/osprepo/base/Packages/createrepo-0.9.9-26.el7.noarch.rpm

```

- 執行下面指令

```bash

for DIR in `find ~/osprepo -maxdepth 1 -mindepth 1 -type d`; do createrepo $DIR; done;
bash ~/osprepo/local-repo.sh
sudo yum update
bash ~/osprepo/local-repo.sh

```

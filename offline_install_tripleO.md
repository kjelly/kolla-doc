Offline Install tripleO
=======================

- 以下操作，都使用 root user
- 下載資料

```bash
wget http://172.22.104.1:4433/centos7mirror.tar.gz -O ~/centos_offline_repo.tar.gz
```

- 解壓縮

```bash
tar zxvf ~/centos_offline_repo.tar.gz -C /opt
```

- 安裝下面套件

```bash

rpm -ivh /opt/osprepo/base/Packages/libxml2-python-2.9.1-6.el7_2.3.x86_64.rpm
rpm -ivh /opt/osprepo/base/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
rpm -ivh /opt/osprepo/base/Packages/python-chardet-2.2.1-1.el7_1.noarch.rpm
rpm -ivh /opt/osprepo/base/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
rpm -ivh /opt/osprepo/base/Packages/deltarpm-3.6-3.el7.x86_64.rpm
rpm -ivh /opt/osprepo/base/Packages/python-deltarpm-3.6-3.el7.x86_64.rpm
rpm -ivh /opt/osprepo/base/Packages/yum-utils-1.1.31-42.el7.noarch.rpm
rpm -ivh /opt/osprepo/base/Packages/createrepo-0.9.9-28.el7.noarch.rpm

```

- 建立 ~/local-repo.sh 並有以下內容

```bash
#!/bin/sh

REPODIR="/opt/osprepo"
REPOFILE="/opt/osp.repo"
rm $REPOFILE 2> /dev/null
rm -rf /etc/yum.repos.d/*
touch $REPOFILE

for DIR in `find $REPODIR -maxdepth 1 -mindepth 1 -type d`; do
    echo -e "[`basename $DIR`]" >> $REPOFILE
    echo -e "name=`basename $DIR`" >> $REPOFILE
    echo -e "baseurl=file://$REPODIR/`basename $DIR`/" >> $REPOFILE
    echo -e "enabled=1" >> $REPOFILE
    echo -e "gpgcheck=0" >> $REPOFILE
    echo -e "gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release" >> $REPOFILE
    echo -e "\n" >> $REPOFILE
done;

cp $REPOFILE /etc/yum.repos.d
```

- 執行下面指令

```bash

for DIR in `find /opt/osprepo -maxdepth 1 -mindepth 1 -type d`; do createrepo $DIR; done;
bash ~/local-repo.sh
sudo yum update
bash ~/local-repo.sh

```

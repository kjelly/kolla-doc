安裝 tripleO
============

此文件示範如何用 TripleO 來佈署作業系統


步驟
---

- 先在 deploy node 上安裝 CentOS

- 如果要 offline 安裝，請先參考這文件 [Offline installation](https://github.com/ya790206/kolla-doc/blob/master/offline_install_tripleO.md)

- 在 root 使用者下執行下面指令

```bash

sudo useradd stack
sudo passwd stack
echo "stack ALL=(root) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/stack
sudo hostnamectl set-hostname undercloud.tripleo
sudo hostnamectl set-hostname --transient undercloud.tripleo
sudo curl -L -o /etc/yum.repos.d/delorean-newton.repo https://trunk.rdoproject.org/centos7-newton/current/delorean.repo
sudo curl -L -o /etc/yum.repos.d/delorean-deps-newton.repo http://trunk.rdoproject.org/centos7-newton/delorean-deps.repo
sudo curl -L -o /etc/yum.repos.d/delorean.repo http://buildlogs.centos.org/centos/7/cloud/x86_64/rdo-trunk-master-tripleo/delorean.repo
sudo yum install -y yum-plugin-priorities
sudo yum install -y python-tripleoclient

```

- 切換使用者到 stack，並在 stack 使用者下執行下面指令。此後都用 stack 使用者在下指令

```

cp /usr/share/instack-undercloud/undercloud.conf.sample ~/undercloud.conf

```

- 修改 /home/stack/undercloud.conf 的下列屬性。記得移除註解。
  - local_ip : 連接在 pxe 網段的 deploy node ip
  - network_gateway : pxe 網段的 gateway
  - local_interface : pxe 網段的網路卡名稱
  - network_cird : 值可以設定成和 local_ip 一模一樣
  - dhcp_start : instance 要用的 ip 開頭
  - dhcp_end : instance 要用的 ip 結束
  - inspection_iprange : inspection 要用的 ip range。不要和其他 ip 衝突即可，要在 pxe
                         網段內
  - enable_ui : 改成 true

- 執行 openstack undercloud install

- source ~/stackrc

- 下載並解壓縮資料

```bash
cd ~stack/
wget https://buildlogs.centos.org/centos/7/cloud/x86_64/tripleo_images/newton/delorean/ironic-python-agent.tar
wget https://buildlogs.centos.org/centos/7/cloud/x86_64/tripleo_images/newton/delorean/overcloud-full.tar
tar xvf overcloud-full.tar
tar xvf ironic-python-agent.tar
```

- 上傳 image

```bash
openstack overcloud image upload
```

- 建立或編輯 ~/nodes.json 並有以下內容。記得修改 mac (pxe用的mac address),
  pm_user, pm_password, pm_addr(ipmi的相關資訊)，name(主機名稱)。可以一次匯入任意數量的
  node

這是一個 node 的範例
```json
{
   "nodes": [
       {
           "pm_type":"pxe_ipmitool",
           "mac":[
               "90:e2:ba:74:24:bd"
            ],
           "pm_user":"admin",
           "pm_password":"admin",
           "pm_addr":"10.20.0.245",
           "name": "node1"
       }
   ]
}
```

這是兩個 node 的範例
```json
{
   "nodes": [
       {
           "pm_type":"pxe_ipmitool",
           "mac":[
               "90:e2:ba:74:24:bd"
            ],
           "pm_user":"admin",
           "pm_password":"admin",
           "pm_addr":"10.20.0.245",
           "name": "node1"
       },
       {
           "pm_type":"pxe_ipmitool",
           "mac":[
               "90:e2:ba:74:24:bc"
            ],
           "pm_user":"admin",
           "pm_password":"admin",
           "pm_addr":"10.20.0.243",
           "name": "node2"
       }
   ]
}
```

- 註冊 nodes

```bash
openstack overcloud node import ~/nodes.json
```

- 執行下面指令，讓那其他 node 變成可用狀態

```bash

openstack overcloud node introspect --all-manageable --provide

```

- 執行下列指令

```bash
openstack baremetal configure boot
```

- 建立 nova key-pair，名稱為 my。用來 ssh 用的

```bash
nova keypair-add my --pub-key ~/.ssh/id_rsa.pub
```


- 替實體主機安裝系統
```bash
    nova boot --image overcloud-full --flavor baremetal --key-name my instance_name
```

替三台實體主機安裝作業系統，主機名稱為 controller
```bash
    nova boot --min-count 3 --max-count 3 --image overcloud-full --flavor baremetal --key-name my controller
```

- 預設那些實體機器的 default gateway 是 deploy node。如果要讓那些機器可以透過 deploy node
  連到外面，可透過下面指令完成。192.0.2.0/24 改成你 ~/undercloud.conf 裡 network_cidr 的值

```bash
iptables -t nat -A POSTROUTING -s 192.0.2.0/24 -o 可以對外的網卡 -j MASQUERADE
```


-  如果連接到 instance 時，ssh 的速度過慢則將 /etc/ssh/sshd_config 的 UseDNS 改成 no

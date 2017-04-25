Deploy OpenStack
================


步驟：
=====

- 先安裝 docker

- 下載 kolla-ansible-docker

```
git clone https://github.com/ya790206/kolla-ansible-docker ~/kolla-ansible-docker
```

- 下載並安裝 kolla-ansible-docker image

```
wget http://172.22.104.1:4433/kolla-ansible-docker/ocata -O ~/kolla-ansible-docker-ocata
docker load < ~/kolla-ansible-docker-ocata
```

- 到 kolla-ansible-docker 目錄下執行下面指令

    ```
    cd ~/kolla-ansible-docker
    ./run.sh
    ./config.sh
    ```

- 修改 /etc/kolla/inventory。將 controller 資訊填在 [control]下方，network node 資訊填在
  [network] 下方。以此類推。network node 無特別需求，則和 controller 一樣。第一欄位填寫 ip 或是主機名稱。如

    ```
    c1 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=/etc/kolla/my ansible_become_user=root ansible_become=true ansible_ssh_extra_args='-o StrictHostKeyChecking=no'
    ```

  c1 是指主機名稱，可以用 ip 代替。
  ansible_user 是指 deploy node 要 ssh 到 user。可以用 root 代替。使用 TripleO 安裝作業系統時，可以用 centos 或 ubuntu 代替。
  ansible_ssh_private_key_file 是指你 ssh 時要用的 key file path 。

- 修改 /etc/kolla/globals.yml 的下列欄位
    - kolla_base_distro : centos
    - kolla_install_type ： source
    - openstack_release : 4.0.1
    - docker_registry: 填寫裝在 deploy node 的 registry 格式如 "192.0.2.1:4000"，4000 為 port 號
    - network_interface: management 網段用的 interface
    - kolla_internal_vip_address : controller 的 vip ，此 ip 不能有人使用
    - kolla_external_vip_interface: controller 的外部 vip 所有的 interface 。外部是指 keystone 裡面的 public url
    - kolla_external_vip_address: controller 的外部 vip ，此 ip 不能有人使用。外部是指 keystone 裡面的 public url
    - docker_registry: docker registry，之前你將 kolla docker push 到的地方
    - storage_interface : storage 網段用的 interface
    - neutron_external_interface: neutron 網段用的 interface

- 如果要設定外部的 ceph ，請多做這些事情 [external ceph](https://github.com/ya790206/kolla-doc/blob/master/external_ceph.md)

- 修改 /etc/kolla/passwords.yml 如果你想換密碼的話

- 執行下面指令，進入 container

    ```
    cd ~/kolla-ansible-docker
    ./exec.sh
    ```

- 在 container 裡面執行下列指令，

    ```
    ka bootstrap-servers
    ka prechecks
    ka deploy
    ```

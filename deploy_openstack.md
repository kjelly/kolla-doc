Deploy OpenStack
================


步驟：
=====

- 先安裝 docker

- 下載 kolla-ansible-docker

```
git clone https://github.com/ya790206/kolla-ansible-docker
```

- 執行下面指令

    ```
    ./build.sh
    ./run.sh
    ./config.sh
    ```

- 修改 /etc/kolla/inventory。將 controller 資訊填在 [control]下方，network 資訊填在
  [network] 下方。以此類推。第一欄位填寫 ip 或是主機名稱。如

    ```
    c1 ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=/etc/kolla/my ansible_become_user=root ansible_become=true
    ```

  c1 是指主機名稱，可以用 ip 代替。ansible_user 是指 deploy node 要 ssh 到 user。可以用 root 代替。
  ansible_ssh_private_key_file 是指你 ssh 時要用的 key file path 。

- 修改 /etc/kolla/globals.yml 的下列欄位
    - kolla_base_distro : centos
    - kolla_install_type ： binary
    - openstack_release : 4.0.1
    - kolla_internal_vip_address : controller 的 vip ，此 ip 不能有人使用
    - docker_registry: docker registry，之前你將 kolla docker push 到的地方
    - network_interface: management 網段用的 interface
    - storage_interface : storage 網段用的 interface
    - neutron_external_interface: neutron 網段用的 interface

- 修改 /etc/kolla/passwords.yml 如果你想換密碼的話

- 執行下面指令

    ```
    ./exec.sh
    ```

- 在 container 裡面執行下列指令

    ```
    ka bootstrap-servers
    ka prechecks
    ka deploy
    ```

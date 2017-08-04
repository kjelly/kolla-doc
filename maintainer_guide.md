Maintainer guide
================

尋找 log
--------

kolla-ansible 的 log 不是存放在主機的 /var/log 下面
而是放在 container 下的 /var/log/kolla 路徑
想要查詢 container name 的方法

```bash
docker ps -a
```

想要進入 container 的方法，nova_libvirt 為 container name
請換成你要的 container name
```
docker exec -it nova_libvirt /bin/bash
```

如果你有啟用 ELK ，則可以在 ELK 看到所有的 log


你也可以到每台電腦的此路徑找到 log


```
cd /var/lib/docker/volumes/kolla_logs/_data
```


查看設定檔
---------

有兩個方法可以看設定檔
第一個是進去 container 裡面看
第二個方法是看 deploy node 下的 /etc/kolla 路徑
裡面有各個元件的設定檔

啟動或停止服務
------------

kolla-ansible 並無提供友善的管理界面來開啟或停止服務
需要到各個主機去開啟或停止服務，指令如下

```bash
docker stop glance_registry
docker start glance_registry
docker restart glance_registry
```

檢查 cinder 和 glance 正確接到 ceph
---------------------------------

```bash
sudo docker exec -it -u root cinder_volume rbd --id cinder -p volumes ls
sudo docker exec -it -u root cinder_backup rbd --id cinder-backup -p backups ls
```

主機重開機後要做的事情
------------------

由於很多 interface 需要和 br-ex 相接，可是 br-ex 剛開機完並不存在(br-ex 是 container 建立的)
因此需要在 br-ex 建立後，將其他 interface 加入 br-ex


修改 OpenStack 設定檔
--------------------

設定檔都在 Container 裡面，實務上你不應該直接修改它。在 deploy node 的 /etc/kolla/config 放你要修改
的設定檔。
如你想要修改 nova.conf 的 cpu_mode 的值，則在 /etc/kolla/config 建立 nova.conf 並填入 cpu_mode=kvm


執行 reconfig 時，產生出來的設定檔有變動，但 container 內的設定沒變
-------------------------------------------------------------

遇到此問題時，解決方法是重新起到那個 container ，讓它讀取新的 container。
目前已知有此問題的 container 有 cinder_backup


計劃性停機與還原
----------------

原則：先關機的最後開
假設有三台 controller node, 名字為 c1, c2, c3
誰是 c1, c2, c3 不重要

關機步驟：
    - 關閉 compute node
    - 關閉 c1
    - 關閉 c2
    - 關閉 c3

復原步驟：
    - 開啟 c3
    - 開啟 c2
    - 開啟 c1
    - 開啟 compute node


突然停機應變措施
----------------

compute node 突然停機： 開啟它
單一 controller node 突然停機： 開啟它
複數的 controller node 突然停機： 找出最晚掛的，最先開

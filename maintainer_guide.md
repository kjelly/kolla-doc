Maintainer guide
================

尋找 log
--------

kolla-ansible 的 log 不是存放在主機的 /var/log 下面
而是放在 container 下的 /var/log/kolla 路徑
想要查詢 container name 的方法

```
docker ps -a
```

想要進入 container 的方法，nova_libvirt 為 container name
請換成你要的 container name
```
docker exec -it nova_libvirt /bin/bash
```

如果你有啟用 ELK ，則可以在 ELK 看到所有的 log


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

```
docker stop glance_registry
docker start glance_registry
docker restart glance_registry
```


sudo docker exec -it -u root cinder_volume rbd --id cinder -p volumes ls
sudo docker exec -it -u root cinder_volume ceph --id glance -p images ls


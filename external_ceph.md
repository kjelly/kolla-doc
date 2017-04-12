external ceph
=============

glance 與 cinder
----------------


要讓 OpenStack 的 cinder 與 glance 接 ceph 很簡單
只要將 globals.yml 中 enable_ceph 設成 no
glance_backend_ceph 和 cinder_backend_ceph 設成 yes 即可
然後在 /etc/kolla 目錄下建立下列的目錄結構
```

.
├── config
│   ├── cinder
│   │   ├── cinder-backup
│   │   │   ├── ceph.client.cinder.keyring
│   │   │   └── ceph.conf
│   │   ├── cinder-volume
│   │   │   ├── ceph.client.cinder.keyring
│   │   │   └── ceph.conf
│   │   └── cinder-volume.conf
│   └── glance
│       ├── ceph.client.glance.keyring
│       ├── ceph.conf
│       └── glance-api.conf
├── globals.yml
└── passwords.yml

```

ceph.conf 直接拿 ceph cluster 的 ceph.conf
glance 的 keyring 的內容看起來如下

```
[client.glance]
    key = AQBUZ9JYYUNnOxAAd8MB03XmOc/HkdfLU1W8MQ==

```

cinder 的 keyring 的內容看起來如下

```
[client.cinder]
    key = AQBUZ9JYYUNnOxAAd8MB03XmOc/HkdfLU1W8MQ==

```


glance-api 的內容如下
```
[DEFAULT]
show_image_direct_url = True

[glance_store]
stores = rbd
default_store = rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf

[image_format]
container_formats = bare
disk_formats = raw
```

cinder-volume.conf 內容如下

```

[DEFAULT]
enabled_backends=rbd-1

[rbd-1]
rbd_ceph_conf=/etc/ceph/ceph.conf
rbd_user=cinder
backend_host=rbd:volumes
rbd_pool=volumes
volume_backend_name=rbd-1
volume_driver=cinder.volume.drivers.rbd.RBDDriver


```

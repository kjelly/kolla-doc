external ceph
=============

glance 與 cinder
----------------


要讓 OpenStack 的 cinder 與 glance 接 ceph 很簡單，
只要將 globals.yml 中 enable_ceph 設成 no。
glance_backend_ceph 和 cinder_backend_ceph 和 nova_backend_ceph 設成 yes 即可。
然後在 /etc/kolla 目錄下建立下列的目錄結構：
```
.
├── config
│   ├── cinder
│   │   ├── cinder-backup
│   │   │   ├── ceph.client.cinder.keyring
│   │   │   ├── ceph.client.cinder-backups.keyring
│   │   │   └── ceph.conf
│   │   ├── cinder-volume
│   │   │   ├── ceph.client.cinder.keyring
│   │   │   ├── ceph.client.cinder-backups.keyring
│   │   │   └── ceph.conf
│   │   └── cinder-volume.conf
│   └── glance
│       ├── ceph.client.glance.keyring
│       ├── ceph.conf
│       └── glance-api.conf
├────── nova
│       ├── ceph.client.cinder.keyring
│       ├── ceph.client.nova.keyring
│       ├── ceph.conf
│       └── nova-compute.conf
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

cinder-volume.conf and cinder-backup.conf 內容如下

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
rbd_secret_uuid = {{ cinder_rbd_secret_uuid }}

backup_driver = cinder.backup.drivers.ceph
backup_ceph_conf = /etc/ceph/ceph.conf
backup_ceph_user = cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true

```

/etc/kolla/config/nova/nova-compute.conf 的內容如下(如果 nova 要接 ceph 的話)

```
[libvirt]
images_rbd_pool=vms
images_type=rbd
images_rbd_ceph_conf=/etc/ceph/ceph.conf
rbd_user=nova
```

/etc/kolla/config/nova/nova-compute.conf 的內容如下(如果 nova 不要接 ceph 的話)
```
[libvirt]
images_type=default

```

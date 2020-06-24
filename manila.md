# manila


```
manila type-create default true
manila share-network-create --name s1 --neutron-net-id n1 --neutron-subnet-id 2bc2a3ba-127b-4462-8c8b-0c80de58e2ec
manila delete s1;manila create NFS 10 --name s1 --share-type default --availability-zone nova --share-network s1

```

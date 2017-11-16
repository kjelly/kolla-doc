# Maintainer guide

## 尋找 log

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


## 查看設定檔

有兩個方法可以看設定檔
第一個是進去 container 裡面看
第二個方法是看 deploy node 下的 /etc/kolla 路徑
裡面有各個元件的設定檔

## 啟動或停止服務

kolla-ansible 並無提供友善的管理界面來開啟或停止服務
需要到各個主機去開啟或停止服務，指令如下

```bash
docker stop glance_registry
docker start glance_registry
docker restart glance_registry
```

## 檢查 cinder 和 glance 正確接到 ceph

```bash
sudo docker exec -it -u root cinder_volume rbd --id cinder -p volumes ls
sudo docker exec -it -u root cinder_backup rbd --id cinder-backup -p backups ls
```

## 主機重開機後要做的事情

由於很多 interface 需要和 br-ex 相接，可是 br-ex 剛開機完並不存在(br-ex 是 container 建立的)
因此需要在 br-ex 建立後，將其他 interface 加入 br-ex


## 修改 OpenStack 設定檔

設定檔都在 Container 裡面，實務上你不應該直接修改它。在 deploy node 的 /etc/kolla/config 放你要修改
的設定檔。
如你想要修改 nova.conf 的 cpu_mode 的值，則在 /etc/kolla/config 建立 nova.conf 並填入 cpu_mode=kvm


## 執行 reconfig 時，產生出來的設定檔有變動，但 container 內的設定沒變

遇到此問題時，解決方法是重新起到那個 container ，讓它讀取新的 container。
目前已知有此問題的 container 有 cinder_backup


## 計劃性停機與還原

原則：先關機的最後開
假設有三台 controller node, 名字為 c1, c2, c3
誰是 c1, c2, c3 不重要

- 關機步驟：
    - 關閉所有 instance
    - 關閉 compute node
    - 停用 kolla container
			```
			wget -O ~/stop_kolla.sh https://gist.githubusercontent.com/kjelly/ad57505d8eaa21ca3b79aa4b18b17de6/raw/e0b29a51f8bb3277f9f3398c615fa5f6224d208e/stop_kolla.sh
			bash ~/stop_kolla.sh
			```
    - 關閉 c1
    - 關閉 c2
    - 關閉 c3

- 復原步驟：
    -
    - 開啟 c3
    - 開啟 c2
    - 開啟 c1
    - 開啟 compute node
    - 開啟 instance


## 突然停機應變措施

compute node 突然停機： 開啟它
單一 controller node 突然停機： 開啟它
複數的 controller node 突然停機： 找出最晚掛的，最先開


## 備份 kolla

- 備份 deploy node 下的 /etc/kolla 的檔案
- 使用 sqldump 備份資料庫:
  - `mysqldump -u root -p --all-databases > alldb.sql`
- 備份 ceph 的 pool

## 根據備份重建 kolla 環境

- 將之前備份的 kolla 設定檔還原到 /etc/kolla 路徑
- 重新佈署 OpenStack
- 還原 mysql cluseter/mariadb cluster:
  - mysql -u root -p < alldb.sql
- 還原 ceph 資料


## 單一 controller node 損壞

- 重灌作業系統（如果作業系統有故障的話）
- 清除 kolla container
  ```bash
  wget https://raw.githubusercontent.com/openstack/kolla-ansible/master/tools/cleanup-containers -O ~/cleanup-containers
  sudo bash ~/cleanup-containers
  ```
- 更新 inventory
  (下面動作與佈署 OpenStack 相同，只是在執行 prepare 和 ka bootstrap-servers 時，
   只能在新主機上跑，避免 OpenStack 損壞）
- 執行 `prepare`
- 處理 networking bonding 等動作
- 執行 `ka bootstrap-servers`
- 執行 `ka deploy`


## mariadb cluster 無法正常啟動

- 將所有 controller 的 mariadb 和 haproxy 停止
  ```bash
  sudo docker stop mariadb
  sudo docker stop haproxy
  ```

- 找出擁有最新資料的主機。在每一台電腦下下面指令，找出 log number 最大的主機。
  172.23.103.1:4000/kolla/centos-source-mariadb:4.0.1 這個要換成該環境的 image。
  ```bash
  sudo docker run --net=host --rm -e "KOLLA_CONFIG_STRATEGY=COPY_ALWAYS" -v /etc/kolla/mariadb:/var/lib/kolla/config_files  -v mariadb:/var/lib/mysql -it 172.23.103.1:4000/kolla/centos-source-mariadb:4.0.1 mysqld --wsrep-recover
  ```

- 在擁有最大 log number 的主機做下面指令，該主機為 master。
  進入 mariadb container，以下指令都在此 container 內部執行。
  ```bash
  sudo docker run --net=host --rm -e "KOLLA_CONFIG_STRATEGY=COPY_ALWAYS" -v /etc/kolla/mariadb:/var/lib/kolla/config_files  -v mariadb:/var/lib/mysql -it --name mariadb_recovery 172.23.103.1:4000/kolla/centos-source-mariadb:4.0.1 /bin/bash
  ```
  初始化 mariadb 設定檔
  ```bash
  kolla_start
  ```
  若上面指令無法持續執行(指無法回到bash)，則執行下面步驟，否則就跳過

  啟動 database server
  ```bash
  mysqld_safe --wsrep_cluster_address=gcomm://
  ```
  若上面指令無法持續執行(指無法回到bash)，則執行下面步驟，否則就跳過

  編輯 /var/lib/mysql/grastate.dat 檔案，將 safe_to_bootstrap 改成 1

  啟動 database server。此時這指令會持續執行。
  ```bash
  mysqld_safe --wsrep_cluster_address=gcomm://
  ```

- 在其他 controller node 執行下面動作
  ```bash
  docker start mariadb
  ```
  若上面指令會執行失敗，則將 /var/lib/mysql/grastate.dat 改成 1。用下面的指令編輯。
  ```bash
  sudo docker run -u root --net=host --rm -e "KOLLA_CONFIG_STRATEGY=COPY_ALWAYS" -v /etc/kolla/mariadb:/var/lib/kolla/config_files  -v mariadb:/var/lib/mysql -it 172.23.103.1:4000/kolla/centos-source-mariadb:4.0.1 vi /var/lib/mysql/grastate.dat
  ```

- 將剛剛在 master (用另外一個連線) ，停止你剛剛自己啟動的 container。
  ```bash
  sudo docker stop mariadb_recovery
  sudo docker rm mariadb_recovery
  ```
  啟動原本的 container。
  ```bash
  sudo docker start mariadb
  sudo docker start haproxy
  ```
- 啟動其他主機的 haproxy
  ```bash
  sudo docker start haproxy
  ```

## galera cluster 發生此錯誤 this member has applied more events than the primary component.Data loss is possible. Aborting.

解決辦法
```bash
cd /var/lib/mysql
rm grastate.dat ib_log* -f
```
## 更新 haproxy 憑證

- 將新的憑證檔 ( haproxy.pem) 放到所有 deploy node 的 /etc/kolla/certificates 下

- 重新跑 reconfigure
```bash
ka reconfigure
```

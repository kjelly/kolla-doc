Build Kolla image
=================

在使用 kolla-ansible 前，需要先建立 kolla image


步驟說明：
---------

- 安裝 docker

- 建立 docker registry
    在 deploy node 上執行下面指令

    ```
        sudo docker run -d -p 5000:5000 --name registry registry:2
    ```

- 下載 kolla-docker

  ```
    git clone https://github.com/ya790206/kolla-docker
  ```

- 在 kolla-docker 目錄下執行下列指令，記得更換 INSECURE_REGISTRY 的 ip

  ```
    INSECURE_REGISTRY=172.22.104.2:5000 VERSION=stable/ocata ./allinone.sh
  ```

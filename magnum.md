Magnum
=======


- 下載 CoreOS image
  ``` bash

      cd ~

      wget https://stable.release.core-os.net/amd64-usr/current/coreos_production_iso_image.iso -O  CoreOS.iso

  ```


- 上傳 CoreOS image 

  ```bash
    cd ~

    glance image-create --name CoreOS --visibility public --disk-format=qcow2 --container-format=bare --os-distro=coreos --file=CoreOS.iso

  ```

- 建立 kubernetes template 
  記得將 keypair-id, flavvor-id, master-flavor-id 換掉


``` bash
magnum cluster-template-create --name k8s-cluster-template-coreos \
                         --image-id CoreOS \
                         --keypair-id my \
                         --external-network-id ext-net \
                         --dns-nameserver 8.8.8.8 \
                         --flavor-id cpu4 \
                         --master-flavor-id cpu2 \
                         --network-driver flannel \
                         --coe kubernetes
```


- 從 ui 建立 kubernetes cluster
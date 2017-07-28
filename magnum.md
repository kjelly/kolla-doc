Magnum
=======


- 下載 CoreOS image
  ``` bash

      cd ~

      wget https://download.fedoraproject.org/pub/alt/atomic/stable/Fedora-Atomic-26-20170723.0/CloudImages/x86_64/images/Fedora-Atomic-26-20170723.0.x86_64.qcow2

  ```


- 上傳 CoreOS image 

  ```bash
    cd ~

    glance image-create --name atomic --visibility public --disk-format=qcow2 --container-format=bare --os-distro=fedora-atomic --file=Fedora-Atomic-26-20170723.0.x86_64.qcow2
  ```

- 建立 kubernetes template 
  記得將 keypair-id, flavvor-id, master-flavor-id 換掉


``` bash
magnum cluster-template-create --name k8s-cluster-template-atomic \
                          --image-id atomic \
                          --keypair-id my \
                          --external-network-id ext-net \
                          --dns-nameserver 8.8.8.8 \
                          --flavor-id cpu4 \
                          --master-flavor-id cpu2 \
                          --network-driver flannel \
                          --coe kubernetes

```


- 從 ui 建立 kubernetes cluster
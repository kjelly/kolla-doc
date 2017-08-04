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


- 下載 kubectl


```bash
cd ~
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
```


- 複製到 /usr/local/bin/

```bash
cd ~
cp kubectl /usr/local/bin
```


- 設定 kubectl

```bash
cd ~
magnum cluster-config cluster-name
export KUBECONFIG=./config
```

- 看 k8s cluster 狀態
```bash
kubectl cluster-info
```

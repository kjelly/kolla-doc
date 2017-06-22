Use pre-building images
-----------------------

- 使用一般使用者或是 stack user

- 下載 registry data 到 deploy node

```
wget http://172.22.104.1:4433/centos-source/registry_data_4.0.1.tar.gz -O ~/registry_data.tar.gz
```

- 在 deploy node 解壓縮到

```
sudo tar -zxf ~/registry_data.tar.gz -C /
```

- 下載 registry image (offline installation only)

```
wget http://172.22.104.1:4433/registry.tar -O ~/registry.tar
```

- 匯入 registry image (offline installation only)

```
docker load < ~/registry.tar
```

- 在 deploy node 啟動 registry

```
sudo docker run -d -p 4000:5000 --restart=always --name registry -v /registry:/var/lib/registry/docker/registry registry:2

```

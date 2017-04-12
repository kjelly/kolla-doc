Use pre-building images
-----------------------

- 下載 registry data

```
wget http://127.0.0.1:4433/centos-source/registry_data_4.0.1.tar.gz -O ~/registry_data.tar.gz
```

- 解壓縮到

```
sudo tar -zxf ~/registry_data.tar.gz -C /
```

- 啟動 registry

```
docker run -d -p 4000:5000 --name registry -v /registry:/var/lib/registry/docker/registry registry:2

```

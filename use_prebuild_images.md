Use pre-building images
-----------------------

- 使用一般使用者或是 stack user

- 下載 registry data 到 deploy node

  ```
  wget http://172.22.104.1:4433/rocky/rocky.tar.gz -O ~/rocky.tar.gz
  ```

- 在 deploy node 解壓縮到

  ```
  sudo tar -zxf ~/rocky.tar.gz -C /
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
  docker run -d -p 4000:5000 --restart=always --name registry -v /registry:/var/lib/registry/docker/registry:z -e REGISTRY_HTTP_HEADERS_Access-Control-Allow-Origin='["http://webui-ip:9999"]' -e REGISTRY_HTTP_HEADERS_Access-Control-Expose-Headers='["Docker-Content-Digest"]' registry:2

  ```

- 檢查 registry 是否正常啟動
  ```
  curl http://172.22.104.1:4000/v2/_catalog
  ```

- open 4000 port

  ```
  sudo iptables -I INPUT 1 -p tcp -m tcp --dport 4000 -j ACCEPT
  ```

- 可選：建立 dcoker registry web ui

    ```
    docker run -d -p 9999:80 -e REGISTRY_URL=http://registry-ip:4000 -e DELETE_IMAGES=true --name registry-ui joxit/docker-registry-ui:static
    ```

# install docker

- Download kolla-runtime
    ```
    wget http://172.22.104.1:4433/rocky/kolla-runtime.tar.gz -O ~/kolla-runtime.tar.gz
    ```

- Untar

    ```
    cd
    tar zxvf ~/kolla-runtime.tar.gz
    ```

- Install docker

    ```
    cd ~/packages
    ./install-docker.sh
    ```

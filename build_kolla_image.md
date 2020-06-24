# Build Kolla image

在使用 kolla-ansible 前，需要先建立 kolla image


## 步驟說明：

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



## custom


build script:
```
#!/bin/bash

venv/bin/python tools/build.py -b centos -t source --template-override override.j2 --network_mode host --tag train --push $@
```


override.j2:
```
{% extends parent_template %}

{% block base_footer %}

{% if base_distro in ['centos', 'oraclelinux', 'rhel'] %}
RUN yum install -y zsh git curl wget && yum group install -y "Development Tools"
{% elif base_distro in ['debian', 'ubuntu'] %}
RUN apt update && apt install -y zsh silversearcher-ag curl git wget
{% endif %}

RUN yum install -y python3 wget
RUN wget -qO- https://bootstrap.pypa.io/get-pip.py | python3  - && \
    pip install --upgrade pip && \
    pip install ipython

RUN python3 -m pip install pynvim;


RUN wget -qO- https://raw.githubusercontent.com/kjelly/auto_config/master/scripts/init_nvim_nightly.sh | bash && \
    wget -qO- https://raw.githubusercontent.com/kjelly/auto_config/master/scripts/init_nvim.sh | bash && \
    wget -qO- https://raw.githubusercontent.com/kjelly/auto_config/master/scripts/init_nvimrc.sh |bash  && \
    nvim.nightly +PlugInstall +qall! || echo ok

RUN git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions

RUN echo '\n \
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh \n \
autoload -Uz colors \n \
colors \n \
autoload -Uz compinit \n \
compinit \n \
setopt share_history \n \
setopt histignorealldups \n \
setopt auto_cd \n \
setopt auto_pushd \n \
setopt pushd_ignore_dups \n \
setopt correct \n \
PROMPT="%(?.%{${fg[green]}%}.%{${fg[red]}%})%n${reset_color}@${fg[blue]}%m${reset_color}(%*%) %~ %# " \
'>> /root/.zshrc



ENV TERM=xterm-256color
{% endblock %}

```

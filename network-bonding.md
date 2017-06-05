Networking Bonding
==================



- 下載 netc 程式


- 在 netc 的相同目錄下，建立 nodes.json 檔案，格式如下

  ```
   {
       "nodes": [
           {
               "target_ip": "172.22.104.11",
               "user": "vagrant",
               "key_path": "/home/user/.ssh/id_rsa",
               "kind": "bond",

               "ifnames": ["bond0", "eth1", "eth2"],
               "ip4": "10.0.0.7",
               "netmask4": "255.255.255.0",
               "gateway4": "10.0.0.1",
               "mode": "0",
               "onboot": "yes"
           },
           {
               "target_ip": "172.22.104.11",
               "user": "vagrant",
               "key_path": "/home/user/.ssh/id_rsa",
               "kind": "eth",

               "ifnames": ["eth1"],
               "ip4": "172.22.104.11",
               "netmask4": "255.255.255.0",
               "mode": "0",
               "onboot": "yes"
           },
           {
               "target_ip": "172.22.104.11",
               "user": "vagrant",
               "key_path": "/home/user/.ssh/id_rsa",
               "kind": "vlan",

               "ifnames": ["eth1.99"],
               "ip4": "172.22.104.13",
               "netmask4": "255.255.255.0",
               "onboot": "yes"
           }
       ]
   }

  ```

  target_ip 填寫你要在哪個主機產生設定檔
  user 填寫你要 ssh 要用的 username
  key_path 填寫 ssh 要用的 private key

  kind 則填寫要產生的 interface 種類。
  有 eth, vlan, bond 這三種可以選,
  不同的 kind 會影響到 ifnames 的寫法

  ip4, netmask4, gateway4 為 ipv4 的 ip, netmask, gateway
  onboot 則表示是否開機時啟動此網路卡，只有在有裝 network-manager 時有用

  當 kind 是 eth 時，就只是幫網路卡設定 ip 的動作。
  ifnames 裡面只能填寫一個值，你想要控制的 interface 名稱
  如 ["eth0"] or ["ens1"]

  當 kind 是 vlan 時，就只是幫產生有 vlan tag 網路卡，並替該網路卡設定 ip。
  ifnames 裡面只能填寫一個值，格式如下：
  如 ["eth0.99"] or ["ens1.99"]
  99 為 vlan tag

  當 kind 是 bond 時，會產生 bond 的 interface
  ifnames 裡面填寫複數個值，第一個值為你要產生的 bond 名稱
  第二個以後填寫要被 bond 的 interface
  如：["bond0", "eth1", "eth2"] ，這個的意思為
  建立 bond0 這個 interface，由 eth1, eth2 所組成。
  你需要設定 mode ，表示 bond mode 。值為一到六。


- 設定完 nodes.json 後，執行 ./netc，程式會自動讀取同目錄下的 nodes.json 檔案。

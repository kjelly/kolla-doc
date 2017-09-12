# Solutions

## kolla

- 如何增加 compute node

  - 進入 kolla-ansible-docker 裡
  - 修改 /etc/kolla-ansible-docker/inventory 。將新主機資訊寫在 [compute] 裡
    (下面動作與佈署 OpenStack 相同，只是在執行 prepare 和 ka bootstrap-servers 時，
     只能在新主機上跑，避免 OpenStack 損壞）
  - 執行 `prepare --limit new_host_name`
  - 處理 networking bonding 等動作
  - 執行 `ka bootstrap-servers --limit new_host_name`
  - 執行 `ka deploy`

- 如何減少 compute node
  1. 用下列指令停用 compute service 。(能夠執行nova 指令即可）
     注意：不要用到 TripleO 的 stackrc
     ```bash
     nova service-disable {{ host }} nova-compute
     ```
  2. 在 `compute node` 是清除所有 container (不要執行錯台）
     cleanup-container 程式放在 kolla-ansible-docker 這個 container
     裡的 /kolla-ansible/tools/ 下
     ```bash
     bash cleanup-containers
     ```
  3. 用下列指令刪除 nova-compute
     可以用 nova service-list 看 host
     ```bash
     nova service-delete {{ host }}
     ```

## sahara
- CreationFailed: Failed to create trust

  1. https://review.openstack.org/#/c/461814/3/ansible/roles/sahara/templates/sahara.conf.j2
  2. 將 heat.conf 的 trusts_delegated_roles = heat_stack_owner 註解

## cinder

- cinder-backup: Error: error connecting to the cluster: error code 95
  cinder bacup 出現上面錯誤訊息k，解法：確保 cinder-backups 有 cinder 和 cinder-backups 的 keyring
  cinder 使用者要能存取 volumes，cinder-backups 要能存取 backups

## tripleO

- TripleO 在 hp 機器上使用 uefi
  - 修改 /etc/ironic/ironic.conf ，將 `[ilo]` 底下的 `default_boot_mode` 改成 `auto`

  - 重新啟動 ironic service

    ```sh
    sudo systemctl restart openstack-ironic-api.service
    sudo systemctl restart openstack-ironic-conductor.service
    sudo systemctl restart openstack-ironic-inspector-dnsmasq.service
    sudo systemctl restart openstack-ironic-inspector.service
    ```

  - import node 時， pm_type 填寫 `pxe_ilo`。並多寫 `ilo_user`, `ilo_password`, `ilo_addr`。
    如下面範例：

    ```json
    {
       "nodes": [
           {
               "pm_type":"pxe_ilo",
               "mac":[
                    "ec:b1:d7:84:f3:f4",
                    "ec:b1:d7:84:f3:f5",
                    "ec:b1:d7:84:f3:f6",
                    "ec:b1:d7:84:f3:f7"

                ],
               "ilo_user":"user",
               "ilo_password":"password",
               "ilo_addr":"172.22.1.61",
               "pm_user":"user",
               "pm_password":"password",
               "pm_addr":"172.22.1.61",
               "name": "hp-node"
           }
       ]
    }
    ```

  - 現在用 nova boot 時， hp 機器就可以用 uefi 開機了（如果有支援的話）

# Solutions

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
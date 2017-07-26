Solutions
---------

- Sahara: CreationFailed: Failed to create trust

  1. https://review.openstack.org/#/c/461814/3/ansible/roles/sahara/templates/sahara.conf.j2
  2. 將 heat.conf 的 trusts_delegated_roles = heat_stack_owner 註解


- Cinder: cinder-backup: Error: error connecting to the cluster: error code 95
  cinder bacup 出現上面錯誤訊息k，解法：確保 cinder-backups 有 cinder 和 cinder-backups 的 keyring
  cinder 使用者要能存取 volumes，cinder-backups 要能存取 backups
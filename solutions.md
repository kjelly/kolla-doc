Solutions
---------

- Sahara: CreationFailed: Failed to create trust

  1. https://review.openstack.org/#/c/461814/3/ansible/roles/sahara/templates/sahara.conf.j2
  2. 將 heat.conf 的 trusts_delegated_roles = heat_stack_owner 註解

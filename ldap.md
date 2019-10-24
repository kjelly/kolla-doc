# LDAP

## 設定方法
- 先在 Controller 執行建立 domains 的指令
LAB 名字可以自訂，但是需要和後面的設定一致
```
openstack domain create LAB
```

- 在 deploy node ，新增與編輯該檔案(/etc/kolla/config/keystone/keystone.conf)，有下面內容

```
identity]
domain_specific_drivers_enabled=true
domain_config_dir=/etc/keystone/domains
```


- 在 deploy node ，新增與編輯該檔案(/etc/kolla/config/keystone/domains/keystone.LAB.conf)，有下面內容
檔名中 LAB 為 domain ，可以自訂(需要和 openstack domain create 所建立的 domains 一致)
url，user，password，suffix，user_tree_dn 欄位需要修改

tls 版本
```
[ldap]
url = ldaps://172.20.0.22:636
user = cn=admin,dc=example,dc=com
password = admin
suffix = DC=example,DC=com
user_tree_dn = OU=people,DC=example,DC=com
user_objectclass = person
user_id_attribute = sAMAccountName
user_name_attribute = sAMAccountName
user_mail_attribute = mail
user_pass_attribute =
user_enabled_attribute = userAccountControl
user_enabled_mask = 2
user_enabled_default = 512
user_attribute_ignore = password,tenant_id,tenants
user_allow_create = False
user_allow_update = False
user_allow_delete = False
query_scope = sub
chase_referrals = False
tls_cacertfile = /etc/keystone/domains/lab-ca.cer


[identity]
driver = ldap

```
無 tls 版本

```
[ldap]
url = ldap://172.20.0.22
user = cn=admin,dc=example,dc=com
password = admin
suffix = DC=example,DC=com

user_tree_dn = ou=people,dc=example,dc=com
user_objectclass = inetOrgPerson

group_tree_dn = ou=group,dc=example,dc=com
group_objectclass = groupOfUniqueNames

user_attribute_ignore = password,tenant_id,tenants
user_allow_create = False
user_allow_update = False
user_allow_delete = False

use_tls = False
tls_req_cert = never


[identity]
driver = ldap

```

- 在 deploy node，將 ldap 的憑證改名放到 /etc/kolla/config/keystone/domains/lab-ca.cer
  無 tls 則跳過

- 在 /etc/kolla/globals.yml 加上 `horizon_keystone_multidomain: True`

- 在 deploy node 的 kolla-ansible container 裡，執行下面指令
  ```bash
  ka reconfigure
  ```

## 測試指令


### 列出該 domains 下的 user

```bash
openstack domain list
openstack user list --domain domain_id

```

## 讓某個 ldap user 擁有 openstack 管理者權限

- 在 controller 或是可以下 openstack　指令的電腦，執行下列指令。LAB 請換成自己的 domain name

```
openstack domain show LAB
```

找出要給 admin 權限的 user id
```
openstack user list --domain LAB
```

讓該 user 擁有 admin 權限
```
openstack role add --domain LAB --user 84058cb685d2c6e027d4649fcf23f27050eba8595269297b1a088e2f021be1d4 admin
```

## 加入 ldap user 到存在的 project

- 在 controller 或是可以下 openstack　指令的電腦，執行下列指令。LAB 請換成自己的 domain name

找出要給 admin 權限的 user id
```
openstack user list --domain LAB
```

讓 user 加入到 admin project

```bash
openstack role add --project admin --user 84058cb685d2c6e027d4649fcf23f27050eba8595269297b1a088e2f021be1d4 _member_
```
## 疑難排解


### log information

log 路徑在 `/var/lib/docker/volumes/kolla_logs/_data/keystone/keystone.log`

### 用 ldapsearch 測試是否可以取出資訊


```bash
ldapsearch -h 172.22.28.79:1389 -b "dc=ai,dc=nchc,dc=org,dc=tw" -D "cn=ai-admin,dc=ai,dc=nchc,dc=org,dc=tw" -w password
```

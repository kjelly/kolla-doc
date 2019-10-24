# designate

- 找出 zone_id (project_id 為 00000000-0000-0000-0000-000000000000 的那個)

```
openstack zone list --all-projects
```

- 編輯 /etc/kolla/config/designate.conf ，填上以下內容

```
[handler:nova_fixed]                                                                                                                                                                                                       
#NOTE: zone_id must be manually filled an ID from openstack zone list                                                                                                                                                      
zone_id = your_zone_id                                                                                                                                                                             
                                                                                                                                                                                                                           
[handler:neutron_floatingip]                                                                                                                                                                                               
#NOTE: zone_id must be manually filled an ID from openstack zone list                                                                                                                                                      
zone_id = your_zone_id
```

- 重新執行 reconfoigure 動作

```
ka reconfigure -t designate
```

- 建立 zone (sample.openstack.org. 為 /etc/kolla/globals/yml 裡的 designate_ns_record 設定的)，
  結尾必須和 designate_ns_record 一樣


```
openstack zone create --email my@test.com test.sample.openstack.org.
```


- 設定網路的 dns domain

```
neutron --insecure net-update  --dns-domain 'test.sample.openstack.org.' pri_net 
```


- 建立 vm 並給 floating ip

- 檢查是否增加紀錄

```
openstack recordset list 'test.sample.openstack.org.'
```



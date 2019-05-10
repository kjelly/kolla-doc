Grafana
=======


configure
---------

1. change telegraf setting

將 roles/telegraf/templates/telegraf.conf.j2 裡的
retention_policy = "default"
改成
retention_policy = "autogen"


2. Go to `http://vip:3000/`


2. Edit data source

type: influxdb
url: http://vip:8086
database: telegraf


## gnocchi with keystone

1. Put `http://vip:5000/` into url.

2. Choose `Access` to `Server`

3. Choose `Auth mode` to `keystone`

4. Put `domain id` to `Domain` (It's dmain id instead of domain name.)

5. Put `project name` to `Project`

6. Put `user name` to `User`

7. Put `password` to `Password`

8. Save


## gnocchi with token

1. Put `http://vip:8041/` into url.

2. Choose `Access` to `Server`

3. Choose `Auth mode` to `token`

4. Put token which created by `openstack token issue` into `token`

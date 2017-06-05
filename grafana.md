Grafana
=======


configure
---------

1. change telegraf setting

將 roles/telegraf/templates/telegraf.conf.j2 裡的
retention_policy = "default"
改成
retention_policy = "autogen"


1.
Go to
http://vip:3000/


2. Edit data source

type: influxdb
url: http://vip:8086
database: telegraf


4.

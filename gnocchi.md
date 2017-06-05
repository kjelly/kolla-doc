Gnocchi
=======


configure
---------

1.
pip install gnocchiclient

2.

確保 OS_AUTH_TYPE 這個環境變數有設定程 password
下面是範例


export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_AUTH_URL=http://192.168.1.3:35357/v3
export OS_INTERFACE=internal
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_TYPE=password


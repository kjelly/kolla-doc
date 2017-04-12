```
export DIB_DEV_USER_USERNAME=user
export DIB_DEV_USER_PWDLESS_SUDO=yes
export DIB_DEV_USER_PASSWORD=password
export DIB_DEV_USER_PASSWORD=password
#disk-image-create ubuntu devuser dynamic-login vm grub2 dhcp-all-interfaces simple-init local-config
disk-image-create ubuntu vm dhcp-all-interfaces local-config

```

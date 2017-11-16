# custom initramfs

## 前言

這篇文章探討如何客製化 TripleO 的 initramfs。此篇文章以 ironic-python-agent.initramfs 為例子。
傳到 Glance 時，image 名稱為 bm-deploy-ramdisk。給 pxe 用時則會複製到用 /httpboot/agent.ramdisk。
要修改時，請確保這些地方都有變更到。


## 步驟
- 切換到 root user
```bash
sudo su
```
- 建立新目錄並切換到那個目錄
```bash
mkdir ~/output
cd ~/output
```
- 解壓縮 initramfs
```bash
zcat ironic-python-agent.initramfs | cpio -idmv

```

- 解壓縮 rpm (optional)

```bash
rpm2cpio kmod-hpdsa-1.2.10-123.rhel7u3.x86_64.rpm |cpio -idmv
```

- 將 ko 檔案複製到家目錄

- 將 ko 檔案複製到 ~/output/opt。目錄可以自訂。
```
cp ~/hpdsa.ko ~/output/opt
```
- 建立並修改 ~/output/lib/modprobe.d/hpdsa.conf 。 hpdsa 這個名字可以自訂。檔案內容像以下這樣。
```
install hpdsa insmod /opt/hpdsa.ko
```
- 建立並修改 ~/output/etc/modules-load.d/hpdsa.conf 。 hpdsa 這個名字可以自訂。檔案內容像以下這樣。

```
hpdsa
```

- 將修改結果打包成 initramfs

```
find . | cpio -o -c | gzip -9 > ~/ironic-python-agent.initramfs
```

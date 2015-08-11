title: set nft server and client
date: 2015-06-20 10:43:51
tags:
---

The nfs service is a convenient way to share the files between Development board and PC. It creates a shared folder between them and both sides can have the full read/write access.

## Install and configure NFS service

### Server side

``` bash
huangt@laptop-pc:/# apt-get install nfs-kernel-server nfs-common portmap
huangt@laptop-pc:/# mkdir nfs_rootfs  (/home/huangt/)
huangt@laptop-pc:/# vim /etc/exports
```
inside add:   /home/huangt/nfs_rootfs *(rw,sync,no_root_squash)
and then restart the nfs server
``` bash
huangt@laptop-pc:/# exportfs -rv
huangt@laptop-pc:/# /etc/init.d/portmap restart
huangt@laptop-pc:/# /etc/init.d/nfs-kernel-server restart
```


### Client side


### Auto mount on boot

Instead of using some automount tools, we can add this new mount in the file /etc/fstab, the format is:

``` bash
<server>:</remote/export></local/directory><nfs-type><options> 0 0
```

* server: 			hostname, IP address
* /remote/export: 	the path to the exported directory
* /local/directory: 	your local mount point
* nfs-type: 			nfs for NFSv2/NFSv3 or nfs4 for NFSv4
* options: 			options listed (comma seperated) in https://www.centos.org/docs/5/html/5.1/Deployment_Guide/s1-nfs-client-config-options.html

you can use this command to check the nfs version:
``` bash
root@ok335x:/# mount -v | grep /home/huangt
10.42.0.1:/home/huangt/nfs_rootfs/ on /mnt type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,nolock,proto=tcp,port=65535,timeo=70,retrans=3,sec=sys,local_lock=all,addr=10.42.0.1)
```
So the vers=3 here indicates it is a NFS3 version.

Then add this line in /etc/fstab:

``` bash
10.42.0.1:/home/huangt/nfs_rootfs  /mnt/  nfs        nolock                0  0
```

Then restart the system, you will see that now the development board have already mounted to the NFS server.

``` bash
root@ok335x:/# ls /mnt/
```
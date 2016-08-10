1. update sources

sources.list
deb http://mirrordirector.raspbian.org/raspbian/ testing main contrib non-free rpi
wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -
source.list.d/ceph.list
deb http://ceph.com/debian-hammer/ jessie main

dist-upgrade


2. create user
3. create ssh 
4. ceph-deploy --cluster spacecamp new ceph-{1,2,3}
4. ceph-deploy install --no-adjust-repos
5. sudo /usr/bin/ceph-mon --cluster spacecamp -i ceph-1 
5. ceph-deploy mon create-initial
5. scp -r systemd ceph@ceph-2:/usr/lib/systemd/system
6. ceph-mon -i {mon-id} --public-addr {ip:port}
ceph
7. ceph-deploy admin ceph-1 ceph-2 ceph-3
8. chmod 644 /etc/ceph/ceph.client.admin.keyring

------

add osd

make sure btrfs-tools
ceph-deploy --cluster spacecamp osd create --fs-type btrfs ceph-1:/dev/sda


deploy mon create-initial failing

copy ceph/systemd/* to /usr/lib/systemd/system/*

scp -r systemd ceph@ceph-2:/usr/lib/systemd/system


ceph-deploy install --no-adjust-repos 
```
[ceph-1][INFO  ] Running command: sudo /usr/bin/ceph --connect-timeout=25 --cluster=ceph --admin-daemon=/var/run/ceph/ceph-mon.ceph-1.asok mon_status
[ceph-1][INFO  ] Running command: sudo /usr/bin/ceph --connect-timeout=25 --cluster=ceph --name mon. --keyring=/var/lib/ceph/mon/ceph-ceph-1/keyring auth get-or-create client.admin osd allow * mds allow * mon allow *
[ceph-1][ERROR ] "ceph auth get-or-create for keytype admin returned 22
[ceph-1][DEBUG ] Error EINVAL: key for client.admin exists but cap mds does not match
```

```
sudo /usr/bin/ceph --connect-timeout=25 --cluster=ceph --name mon. --keyring=/var/lib/ceph/mon/ceph-ceph-1/keyring auth get-or-create client.admin osd allow * mds allow * mon allow *
```



http://bryanapperson.com/blog/the-definitive-guide-ceph-cluster-on-raspberry-pi/
https://www.linkedin.com/pulse/ceph-raspberry-pi-rahul-vijayan
http://docs.ceph.com/docs/master/install/manual-deployment/
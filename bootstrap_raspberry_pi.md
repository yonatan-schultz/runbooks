##Configuring Raspberry Pi to Join the Network

This runbook will prepare a new Raspberry Pi to join to the Spacecamp network prior and ready it for production work. Once these steps are finished the box will be ready for software deployment.

```
Remember (because at some point you will forget): 
--------

Default user/pass : pi/raspberry
```

1. On first login run `sudo apt-get update && sudo apt-get install -y` to update all packages to latest versions. This can take 30-60 minutes to finish.

2. Once that has completed, run `sudo raspi-config` for initial preparation of the system.

	a. Expand root filesystem
		This may have already been done when you copied Raspian to the SD card but it doesn't hurt to rerun.
	b. Overclock
		If the Pi supports overclocking, set to medium. 
	c. Set hostname. Under Advanced, set the hostname appropriately for this new box.
	
3. Create out of band user. While the system supports LDAP, we want to have an out of band user in case the LDAP server(s) become unresponsive.
	a. Run `sudo adduser yschultz`
	b. Add user to sudoers file by deleting the 'pi' entry and adding `yschultz ALL=(ALL:ALL) ALL`. 
	c. Add user's public key to `/home/yschultz/.ssh/authorized_keys`
	d. Test by opening a new terminal and sshing to host, verify `sudo su -` works.

4. Delete default user. After logging in as out of band user run `sudo /usr/sbin/userdel -r pi`

5. Disallow root ssh access by setting `PermitRootLogin no` in `/etc/ssh/sshd_config` and restarting ssdh with `/etc/init.d/sshd restart`

6. Add this box to DNS via Puppet
7. Add a static lease to DHCP server
8. Install and run puppet agent `sudo apt-get install puppet -y && sudo puppet agent -t`
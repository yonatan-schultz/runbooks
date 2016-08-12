##Configuring DHCP Servers

```
Note: this service has been puppetized! If you want to add additional static host leases, edit the /etc/puppet/modules/dhcp/files/authoritative/dhcpd.conf file in the puppet github repo.
```

Installing a fully fledged DHCP server on a Raspberry Pi is as simple as running `sudo apt-get install isc-dhcp-server`. This will install the Internet System Consortium DHCP server.

Configuration files are stored in `/etc/dhcp`, active DHCP leases are stored in `/var/lib/dhcp/dhcpd.leases` and logs are stored in `/var/log/syslog`.

The first file we'll edit is `/etc/default/isc-dhcp-server`. We'll need to configure which network interface(s) we'll be serving DHCP requests on. Assuming that you are wired (an not wireless), you only have eth0 so add `INTERFACES="eth0"` on a new line at the end of the file:

```
...
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES="eth0"
```

The main configuration takes place in `/etc/dhcp/dhcpd.conf`.

We'll need to set up some basic global defaults:

```
option domain-name "spacecamp.local";
option domain-name-servers 192.168.1.118, 192.168.1.112;
option routers 192.168.1.1;
default-lease-time 600;
max-lease-time 7200;
```

Here we specify our domain name, DNS servers, router(s), default & max lease times. 

Since this is our primary DHCP server (we'll set up a failover server shortly) we want to have the following line uncommented:

```
# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;
```

Our main configuration will be defining the pool of available DHCP addresses.

```
subnet 192.168.1.0 netmask 255.255.255.0 {
        range 192.168.1.200 192.168.1.250;
}
```

Here we are using 200-250 as our DHCP range. This also defines the appropriate netmask as 255.255.255.0. 

Lastly, configure any static DHCP leases thusly:

```
host ns1 {
        hardware ethernet b8:27:eb:4f:17:b0;
        fixed-address 192.168.1.118;
}
```

Once you've entered all the static leases, restart the DHCP server `sudo /etc/init.d/isc-dhcp-server restart` and configure your router as a DHCP forwarder.



###Sources:

https://wiki.debian.org/DHCP_Server
https://www.server-world.info/en/note?os=Debian_8&p=dhcp
http://www.debianhelp.co.uk/dhcp.htm
https://www.madboa.com/geek/dhcp-failover/
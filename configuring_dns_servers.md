###Configuring DNS Servers

```
Note: DNS has been puppetized. Simply giving the server an appropriate hostname and running puppet agent will configure the server as primary (ns1) or replica (ns{2..9})
```

This runbook will cover setting up both primary and replica DNS servers on a Raspberry Pi. You will require one primary DNS server on which you will make changes and multiple replicas in case of an outage on the primary.

```
Both primary and replicas will need to have bind installed. `sudo apt-get install bind9 bind9utils bind9-doc`
```


###Primary DNS Server Configuration

First we'll configure the `/etc/bind/named.conf.options` file.

Within the `options` stanza we will add the following:

```
options {
        directory "/var/cache/bind";

        recursion yes;                 # enables resursive queries
        allow-recursion { 192.168.1.0/24; };  # allows recursive queries from local network only
        listen-on { 192.168.1.118; };   # ns1 private IP address - listen on private network only
        allow-transfer { 192.168.1.22; };      # allow zone transfers to your slave DNS servers

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

};
```

Next let's define our zones within `/etc/bind/named.conf.local`

```
zone "spacecamp.local" {
    type master;
    file "/etc/bind/zones/db.spacecamp.local"; # zone file path
    allow-transfer { 192.168.1.112; };           # ns2 private IP address - secondary
};


zone "168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.168.192"; 
    allow-transfer { 192.168.1.112; };  # ns2 private IP address - secondary
};
```

The first zone defines our forward zone with hostname, type of the server (master), the path to DNS entries and where you will allow transfers to.

The second zone defines the reverse zone which will include the reverse dns lookup for all of our hosts.

Once you've finished with these files you can check for syntax errors by running `named-checkconf`.

We're pointing at two databases that don't exist, `/etc/bind/zones/db.spacecamp.local and /etc/bind/zones/db.168.192`. Let's create and populate those files.

`mkdir /etc/bind/zones`

Example `/etc/bind/zones/db.spacecamp.local`

```
$TTL    600
@       IN      SOA     ns1.spacecamp.local. admin.spacecamp.local. (
                  3     ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL
;
; name servers - NS records
     IN      NS      ns1.spacecamp.local.
     IN      NS      ns2.spacecamp.local.

; name servers - A records
ns1.spacecamp.local.          IN      A       192.168.1.118
ns2.spacecamp.local.          IN      A       192.168.1.112

; 192.168.1.0/24 - A records

```

Things to note:
	1. TTL is set to 5 minutes to allow quick changes to infrastructure.
	2. Serial should be updated each time a change is made to this file.
	
Before the A records we create NS records for each of our nameservers. Next we populate the file with an entry for each ns server as well as an A record for each server that we wish to have in DNS (as well as CNAME records or other records).

After finishing the file, check the syntax by running `named-checkzone spacecamp.local db.spacecamp.local`

Correct any errors, then create the reverse zone file at `/etc/bind/zones/db.168.192`

```
$TTL    600
@       IN      SOA     nyc3.spacecamp.local. admin.spacecamp.local. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; name servers
      IN      NS      ns1.spacecamp.local.
      IN      NS      ns2.spacecamp.local.

; PTR Records
118.1   IN      PTR     ns1.spacecamp.local.    ; 192.168.1.118
112.1   IN      PTR     ns2.spacecamp.local.    ; 192.168.1.112
```

As in the forward file, we have a 5 minute TTL. Serial should also be incremented to be in line with the forward zone file. 

As with the forward file, after you are done editing the file check the syntax by running `named-checkzone 168.192.in-addr.arpa /etc/bind/zones/db.168.192`.

Restart bind (`/etc/init.d/bind restart`)and point some servers at the new DNS server to test connectivity. 

###Replica Configuration

All replicas are configured identically. They will replicate data from the primary DNS server and serve traffic in the case that the primary experiences an outage.

Like the primary, we will begin by editing `/etc/bind/named.conf.options`

```
options {
        directory "/var/cache/bind";

        recursion yes;                 # enables resursive queries
        allow-recursion { 192.168.1.0/24; };  # allows recursive queries from local network only
        listen-on { 192.168.1.112; };   # ns2 private IP address - listen on private network only
        allow-transfer { none; };      # allow no transfers

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

};
```      

And then we follow with `/etc/bind/named.conf.local`

```
zone "spacecamp.local" {
    type slave;
    file "slaves/db.spacecamp.local";
    masters { 192.168.1.118; };  # ns1 private IP
};

zone "168.192.in-addr.arpa" {
    type slave;
    file "slaves/db.168.192";
    masters { 192.168.1.118; };  # ns1 private IP
};
```

Note that `named.conf.local` specifies that this server is a slave a points to the directory where the replica's database will be stored as well as containing a pointer to the master.

As before, run `named-checkconf` in order to check these configuration files.

Finally, restart bind `/etc/init.d/bind restart`

-----
Contratulations! You have set up both a primary and replica within your network. Now you can start adding other records and map out network.

Since this is in puppet simply making a PR against the puppet repo containing the new hosts and then running puppet will update the bind files and restart the service.

-----
##Sources
[https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-16-04
](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-16-04)

[https://www.liquidweb.com/kb/how-to-lowering-your-dns-ttls/](https://www.liquidweb.com/kb/how-to-lowering-your-dns-ttls/)

[http://www.puppetcookbook.com/posts/restart-a-service-when-a-file-changes.html](http://www.puppetcookbook.com/posts/restart-a-service-when-a-file-changes.html)
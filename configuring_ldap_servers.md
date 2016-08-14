##Configuring LDAP Server & Clients

This runbook will allow you to configure a Debian (Raspberry Pi, in this case) system as an LDAP server as well as configure clients to authenticate the LDAP server.

###Install LDAP Server

To get started, install the slapd and ldap-utils packages `sudo apt-get install slapd ldap-utils`. Enter a password here. You will overwrite this in a moment so it's content is unimportant

We'll need to reconfigure slapd for our network so go ahead and run `sudo dpkg-reconfigure slapd` and follow these steps:

1. Omit OpenLDAP server configuration? **No**
2. DNS domain name: **spacecamp.local**
3. Organization name: **Spacecamp**
4. Administrator password: **Choose administrator password**. This is the password that you will use to log into and configure your LDAP installation.
5. Database backend to use: **HDB**
6. Do you want the database to be removed when slapd is purged? **No**
7. Move old database? **Yes**
8. Allow LDAPv2 protocol? **No**

Lastly, let's edit our ldap.conf file `/etc/ldap/ldap.conf`. The lines want to edit are `BASE` and `URI`:

```
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE dc=spacecamp,dc=local
URI ldap://192.168.1.121

#SIZELIMIT 12
#TIMELIMIT 15
#DEREF never
```

###Install phpLDAPAdmin

To administer our LDAP server we'll use phpLDAPAdmin (you can also use other programs such as ApacheDirectoryStudio).

`sudo apt-get install phpldapadmin`

Once the installation has finished, we need to edit the pdpldapadmin config file `/etc/phpldapadmin/config.php`. The file is quite long but find and edit the following lines:

```
$servers = new Datastore();
$servers->newServer('ldap_pla');
$servers->setValue('server','name','Spacecamp LDAP Server');
$servers->setValue('server','host','192.168.1.121');
$servers->setValue('server','port',389);
$servers->setValue('server','base',array('dc=spacecamp,dc=local'));
$servers->setValue('login','bind_id','cn=admin,dc=spacecamp,dc=local');
```
You should now be able to log into your pdpldapadmin installation at `http://192.168.1.121/phpldapadmin` with the administrator password that you configured in step 4.

Add a group called `Users` by clicking on the root of the directory and then choosing `Create a child entry`, `Generic: Posix Group`/ Give this group the name `Users` and click `Create Object`. 

Once you have a group to add users to, you can add users by clicking on `cn=Users`, `Create a child entry`, `Generic: User Account`. Fill the information out completely. 

Lastly, configure your clients to authenticate against the server.

###Configuring Clients

```
Note: This process has been puppetized and is included in the default profile for all new hosts. 
```

Start by installing the necessary packages `sudo apt-get install libpam-ldapd libnss-ldapd`. 

When doing this manually, you will be asked some configuration questions:

1. LDAP server URI: **ldap://192.168.1.121:389**
2. LDAP server search base: **dc=spacecamp,dc=local**
3. Name services to configure: **group, passwd, shadow**

Finally, we need to edit `/etc/pam.d/common-session`. Add the following line to the end of the file to automatically create home directories for users upon login ( assuming the directories do not already exist ):

```
session required pam_mkhomedir.so umask=0022 skel=/etc/skel
```

Running `getent passwd` should now show you a list of all local users as well as showing the user that you created above. Try logging in as your LDAP user to verify connectivity.


###Sources
[https://www.howtoforge.com/debian-squeeze-ldap-server-with-openldap-and-phpldapadmin](https://www.howtoforge.com/debian-squeeze-ldap-server-with-openldap-and-phpldapadmin)

[http://workshop.botter.ventures/2012/07/27/how-to-setup-an-ldap-server-on-raspberry-pi/](http://workshop.botter.ventures/2012/07/27/how-to-setup-an-ldap-server-on-raspberry-pi/)

[http://theurbanpenguin.com/wp/index.php/authenticating-your-raspberry-pi-suers-to-openldap/](http://theurbanpenguin.com/wp/index.php/authenticating-your-raspberry-pi-suers-to-openldap/)

[https://wiki.debian.org/LDAP/NSS](https://wiki.debian.org/LDAP/NSS)
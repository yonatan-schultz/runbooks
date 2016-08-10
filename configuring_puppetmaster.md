##Configuring Raspberri Pi as Puppetmaster

Creating a puppetmaster is extremely simple on the Raspberry Pi (or any Debian based system). Simply run `sudo apt-get install puppetmaster-passenger -y`.

After your installation, you have a fully fledged puppetmaster.


All puppet code exists within the `/etc/puppet/` directory. Add modules here and include profiles as needed within `manifests/site.pp`

The first time a new puppet agent runs it will creat a new cert and wait for the puppetmaster to sign it. When you run `sudo puppet cert --list --all` you will see a list of all existing certs. 

You will need to sign the cert before the puppet agent will run on the client. You can sign the cert by running `sudo puppet cert sign <HOSTNAME>`

All puppet code is versioned and contained within the [puppet](https://github.com/yonatan-schultz/puppet) github repository. 

----
###Sources:
http://frederickvandenbosch.be/?p=1843
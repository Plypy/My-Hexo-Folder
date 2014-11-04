title: 'OpenLDAP Problem'
date: 2014-05-14 21:16:06
tags:
- OpenLDAP
---

## var/run/slapd Problem

Today when I tried to start my OpenLDAP server by directly using `slapd` command,  it's surprisingly failed. It logged this

    unable to open file "/var/run/slapd/slapd.pid": 2 (No such file or directory)

It seems the `var/run/slapd` folder is missing. 

But if I use `service slapd start` it will succeed, and generate `var/run/slapd` correctly. However, after reboot this folder will once again disappear.

After some search, I found that every time when Ubuntu boot, the `var/run` are empty at first, and each folder will be created dynamically by individual service as they start.

And then I remembered that some time before I've used `chkconfig` to disable slapd from auto startup.

So the solution is simple, just use `chkconfig slapd on` to enable it.

## Service problem

But after that I encountered the old slapd service problem again, that is the slapd service don't have right permission to access the db file because of some problems related with Ubuntu's Apparmor mechanism(SELinux).

And it's the exact reson I disabled slapd service.

Now I have to `sudo service slapd stop` first and then use the command below to start slapd manually.


    sudo slapd -d 1 -f /etc/ldap/slapd.conf


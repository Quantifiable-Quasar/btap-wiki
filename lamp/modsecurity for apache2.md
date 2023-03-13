---
layout: default
title: Modsecurity for Apache 2
parent: LAMP
nav_order: 56
---
designed for apache
this follows owasp core rule set 

### install

```bash
sudo apt install libapache2-mod-security2 -y
```

then enable headers module 
```bash
a2enmod headers
```

restart apache 
```bash
sudo systemctl restart apache2
```

get rule set:
owasp mod security core rule set

find default ruleset:

```bash
ls -als /usr/share/modsecurity-crs/
```

install guide for new ruleset:
[Installation – OWASP ModSecurity Core Rule Set](https://coreruleset.org/installation/)

replace defaults with owasp rule set
```bash
rm -rf /usr/share/modsecurity-crs/

git clone https://github.com/coreruleset/coreruleset /usr/share/modsecurity-crs/
```

enable config file
```bash
mv /usr/share/modsecurity-crs/crs-setup.conf.example /usr/share/modsecurity/crs-setup.conf
```
defaults should be good. owasp is pretty smart

now that we have a ruleset we have to configure modsecurity itself so that it can work

```bash
mv /etc/modsecurity/modsecurity.conf-recomended /etc/modsecurity/modsecurity.conf
```

by default modsecurity will only detect. we want it to block attacks. to do this edit the above file and change the SecRuleEnginge from DetectionOnly to On as shown below:
```

SecRuleEngine On
```

next edit apache configs to enable modsecurity wiht new config file.
just edit /etc/apache2/apache2.conf and add the line

```
<IfModule security2_module>
	Include /usr/share/modsecurity-crs/crs-setup.conf
	Include /usr/share/modsecurity-crs/rules/*.conf
<\IfModule>
```
also include this in /etc/apache2/sites-enabled/ if you have virtual hosts enabled (put it in the virtual host tags)
but add SecRuleEngine On before the IfModule tags

finally restart apache wiht

```bash
sudo systemctl restart apache2
```
now waf should be working

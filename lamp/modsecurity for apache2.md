---
layout: default
title: Modsecurity for Apache 2
parent: LAMP
nav_order: 56
---

## What is Modsecurity

Modsecurity is a FOSS Web Application Firewall. A WAF is a firewall that monitors HTTP traffic, and blocks web app attacks. A WAF works differently than a typical firewall because it doesn't block ports, but rather it blocks traffic that matches a rule set that looks like an attack. For example a rule could be that you cant end a password with --. If the WAF sees traffic like that it would block the request because it looks like a SQL injection. So then, we need to get a good rule set to utilize this tool to its fullest potential. To do that this guide chose the OWASP core rule set. Below are some links for further reading, and an installation guide for this WAF. 

### Further Reading
- [Installation – OWASP ModSecurity Core Rule Set](https://coreruleset.org/installation/)
- https://coreruleset.org/
- https://www.feistyduck.com/books/modsecurity-handbook/
- https://owasp.org/www-project-modsecurity-core-rule-set/
- https://www.netnea.com/cms/apache-tutorials


## Installation

###### Ubuntu
```bash
sudo apt install libapache2-mod-security2 -y
```

###### CentOS
```bash
sudo yum install mod_security
```

Then enable headers module (mod_headers)
```bash
a2enmod headers
```

restart apache 

###### Ubuntu
```bash
sudo systemctl restart apache2
```

###### CentOS
```bash
sudo systemctl restart httpd.service
```

After we have installed the module we have to download the OWASP ruleset. 
owasp mod security core rule set

Show default ruleset:
```bash
ls -lah /usr/share/modsecurity-crs/
```

Replace defaults with owasp rule set
```bash
rm -rf /usr/share/modsecurity-crs/

wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.4.zip

unzip v3.3.4

mv coreruleset-3.3.4 /usr/share/modsecurity-crs
```

Rename the  config file
```bash
mv /usr/share/modsecurity-crs/crs-setup.conf.example /usr/share/modsecurity/crs-setup.conf
```

The defaults should be good. Unless you have a good reason to change them I would leave them the same OWASP is pretty smart. 

Now that we have the rule set lets enable modsecurity.

Rename the config file. 
```bash
mv /etc/modsecurity/modsecurity.conf-recomended /etc/modsecurity/modsecurity.conf
```

By default modsecurity will only detect. We want it to block attacks. To do this edit the above file (/etc/modsecurity/modsecurity.conf) and change the SecRuleEnginge from DetectionOnly to On as shown below:

```
SecRuleEngine On
```

Next edit apache configs to enable modsecurity wiht new config file.
just edit /etc/apache2/apache2.conf and add the line.

```
<IfModule security2_module>
	Include /usr/share/modsecurity-crs/crs-setup.conf
	Include /usr/share/modsecurity-crs/rules/*.conf
<\IfModule>
```

If you have virtual hosts enabled on this web server, then it is best practice to also enable the module there too. To do that simpily go to /etc/apache2/sites-enabled. In that directory you will find files with the same name as the website ending in .conf. If you do not have these files or you do not have a domain name, then you do not have virtual hosts configured. In this file, if it exists, there should be an opening tag that is labeled VirtualHost. Inside of those tags insert the following:

```
SecRuleEngine On
<IfModule security2_module>
	Include /usr/share/modsecurity-crs/crs-setup.conf
	Include /usr/share/modsecurity-crs/rules/*.conf
<\IfModule>
```


Finally restart apache with:

###### Ubuntu
```bash
sudo systemctl restart apache2
```

###### CentOS
```bash
sudo systemcel restart httpd.service
```


### Conclusion

Now modsecurity should be enabled. Test it out on some previously vulnerable fields, and if it hits a rule modsecurity should return a 403 HTTP code. 

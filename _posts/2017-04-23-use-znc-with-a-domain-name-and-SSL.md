---
layout: post
title:  "How to use ZNC with a domain name over SSL"
date:   2017-04-23
---

[ZNC](http://wiki.znc.in/) is an IRC bouncer, a service that connects to an IRC server and relays messages between that server and your IRC client. ZNC offers [several benefits](http://wiki.znc.in/Introduction): it can maintain a connection to an IRC server after a client disconnects, for example, and buffer messages sent while the client is disconnected.

I've been running ZNC on a VPS for a while, using a self-signed SSL certificate for encrypted connections. I thought it would be nice to use a subdomain to access the web interface and for IRC connections, which in turn would allow me to use a verified SSL certificate from [Let's Encrypt](https://letsencrypt.org/). This is how I did it.

This guide assumes that you already have a server running ZNC ([this is a good guide to getting set up](https://www.vultr.com/docs/install-and-setup-znc-on-ubuntu)), and a domain name pointing to that server. I used the Apache web server running on Ubuntu to enable access to the ZNC web interface via our domain name on ports 80 and 443, and the examples below reflect that. If your server's behind a firewall, you'll also want to make sure that you've opened ports 80, 443 and 6697.

## Configure ZNC

Firstly, we need to configure ZNC to listen for connections to the web interface and IRC. We configure these connections separately, because we want to use SSL for IRC connections (which are made directly to the server), but not for connections to the web interface (which we will make via an internal reverse proxy). We make the changes on the global settings page of the web admin interface; the relevant section should look like this:

![ZNC web interface](/assets/znc-web-interface.png){: .center-image }

The first line is for IRC connections, the second for connections to the web interface. You may have already configured ZNC to listen for connections to the web interface on a different port. If so, you can leave this entry for now and remove it later.

## SSL

Next we need to obtain an SSL certificate from Let's Encrypt, and install it on our server. Thankfully, the [Electronic Frontier Foundation](https://www.eff.org/) has provided the [Certbot](https://certbot.eff.org/) utility, which automates this process.

You can use Certbot to generate a certificate to install manually, but the tool can also handle the configuration of several common software combinations, including, in my case, Apache and Ubuntu. So for me the first step was to install Certbot:

```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-apache
```

And after that, I just had to run the tool with the `--apache` option:

```
$ sudo certbot --apache
```

Certbot will then ask you to specify which domain name you want to issue a certificate for, to provide a contact email address, and to choose whether you want to enable both HTTP and HTTPS access or to redirect all requests to HTTPS (I'd recommend the latter). This will create and enable an Apache configuration file (`/etc/apache2/sites-available/000-default-le-ssl.conf`) with the required SSL configuration directives. If you choose to redirect all requests to HTTPS, Certbot will also update your default Apache configuration file (`/etc/apache2/sites-available/000-default.conf`) to enable this.

Once the certificate files have been installed for Apache, we need to concatenate them in a single file in the ZNC directory (which, for me, is in the home directory of the `znc` user).

```
$ sudo cat /etc/letsencrypt/live/example.org/privkey.pem /etc/letsencrypt/live/example.org/cert.pem /etc/letsencrypt/live/example.org/chain.pem > /home/znc/.znc/znc.pem
```

At this point you should be able to make encrypted IRC connections to your ZNC server on port 6697.

### Certificate renewal

Let's Encrypt certificates expire after 90 days, so we need to configure our server to automatically renew the certificate. To do that, we first open the root crontab file for editing:

```
$ sudo crontab -e
```

And add the following line:

```
@monthly certbot renew --post-hook "cat /etc/letsencrypt/live/example.org/privkey.pem /etc/letsencrypt/live/example.org/cert.pem /etc/letsencrypt/live/example.org/chain.pem > /home/znc/.znc/znc.pem"
```

Now Certbot will renew the certificate, if required, once a month, and create a new `.pem` file in the ZNC directory using the new certificate files (the `--post-hook` option specifies a command to be run after certificate renewal).

## Set up a reverse proxy

To make the ZNC web interface available at a given domain name, we need to set up a [reverse proxy](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html#forwardreverse). This will direct HTTP(S) requests to the server on ports 80 and 443 to the ZNC web interface running on port 8080. On my server, I used Apache to set up a reverse proxy. To do this, we first have to enable the `mod_proxy` and `mod_proxy_http` modules:

```
$ sudo a2enmod proxy proxy_http
```

With these modules enabled, we just need to add the following lines to the Apache configuration file(s). If you're redirecting HTTP requests to HTTPS, just add them to the SSL configuration file added by Certbot. Otherwise, add them to both this file and the default Apache configuration file.

```
ProxyPass "/" "http://127.0.0.1:8080/"
ProxyPassReverse "/" "http://127.0.0.1:8080/"
```

And that's it, you should now be able to use your domain name to make encrypted connections to ZNC's IRC server and web interface!

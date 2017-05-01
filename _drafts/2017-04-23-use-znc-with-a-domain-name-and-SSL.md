---
layout: post
title:  "How to use ZNC with a domain name over SSL"
date:   2017-04-23
---

[ZNC](http://wiki.znc.in/) is an IRC bouncer, a service that connects to an IRC server and relays messages between that server and your IRC client. ZNC offers [several benefits](http://wiki.znc.in/Introduction): it can maintain a connection to an IRC server after a client disconnects, for example, and buffer messages sent while the client is disconnected.

I've been running ZNC on a VPS for a while, using a self-signed SSL certificate for encrypted connections. I thought it would be nice to use a subdomain to access the web interface and for IRC connections, which in turn would allow me to use a verified SSL certificate from [Let's Encrypt](https://letsencrypt.org/). This is how I did it.

This guide assumes that you already have a server running ZNC ([this is a good guide to getting set up](https://www.vultr.com/docs/install-and-setup-znc-on-ubuntu)), and a domain name pointing to that server.

## Configure ZNC

Firstly, we need to configure ZNC to listen for connections to the web interface and IRC. We configure these connections separately, because we want to use SSL for IRC connections (which are made directly to the server), but not for connections to the web interface (which we will make via an internal reverse proxy). We make the changes on the global settings page of the web admin interface; the relevant section should look like this:

![ZNC web interface](/assets/znc-web-interface.png)

The first line is for IRC connections, the second for connections to the web interface. You've probably already configured ZNC to listen for SSL connections to the web interface on a different port. If so, you can leave this entry for now and remove it later.

## Set up a reverse proxy

---
slug: nginx-and-perlfastcgi-on-ubuntu-10-04-lts-lucid
deprecated: true
author:
  name: Linode
  email: docs@linode.com
description: 'Serve dynamic websites and applications with the lightweight nginx web server and Perl-FastCGI on Ubuntu 10.04 LTS (Lucid).'
keywords: ["nginx", "fastscgi perl", "nginx ubuntu 10.04", "nginx fastcgi", "nginx perl"]
tags: ["web server","perl","ubuntu","nginx"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/websites/nginx/nginx-and-perlfastcgi-on-ubuntu-10-04-lts-lucid/','/web-servers/nginx/nginx-and-perlfastcgi-on-ubuntu-10-04-lts-lucid/','/web-servers/nginx/perl-fastcgi/ubuntu-10-04-lucid/']
modified: 2013-10-07
modified_by:
  name: Linode
published: 2010-05-03
title: 'Nginx and Perl-FastCGI on Ubuntu 10.04 LTS (Lucid)'
relations:
    platform:
        key: nginx-perl-fastcgi
        keywords:
            - distribution: Ubuntu 10.04
---



The nginx web server is a fast, lightweight server designed to efficiently handle the needs of both low and high traffic websites. Although commonly used to serve static content, it's quite capable of handling dynamic pages as well. This guide will help you get nginx up and running with Perl and FastCGI on your Ubuntu 10.04 LTS (Lucid) Linode.

It is assumed that you've already followed the steps outlined in our [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/). These steps should be performed via a root login to your Linode over SSH.

## Set the Hostname

Before you begin installing and configuring the components described in this guide, please make sure you've followed our instructions for [setting your hostname](/docs/guides/getting-started/#setting-the-hostname). Issue the following commands to make sure it is set properly:

    hostname
    hostname -f

The first command should show your short hostname, and the second should show your fully qualified domain name (FQDN).

## Install Required Packages

### Install `nginx` and `spawn-fcgi`

Issue the following commands to update your system and install the nginx web server and FastCGI components:

    apt-get update
    apt-get upgrade
    apt-get install nginx spawn-fcgi libfcgi0ldbl

You'll also need to install `fcgiwrap`, which unfortunately isn't included in the Ubuntu 10.04 repositories. The version provided in Ubuntu 11.04 will be used instead; issue one of the following commands to download and install the required deb package, selecting either the 32-bit or 64-bit version as appropriate.

### Install `fcgiwrap` for 32-bit Ubuntu

Commands:

    wget http://mirrors.us.kernel.org/ubuntu//pool/universe/f/fcgiwrap/fcgiwrap_1.0.3-1_i386.deb
    dpkg -i fcgiwrap_1.0.3-1_i386.deb

### Install `fcgiwrap` for 64-bit Ubuntu

Commands:

    wget http://mirrors.us.kernel.org/ubuntu//pool/universe/f/fcgiwrap/fcgiwrap_1.0.3-1_amd64.deb
    dpkg -i fcgiwrap_1.0.3-1_amd64.deb

## Configure DNS

Create an "A" record pointing your domain name to your Linode's IP address. If you're using the Linode DNS Manager interface, please refer to our [Linode DNS manager guide](/docs/products/networking/dns-manager/guides/common-dns-configurations/) for instructions.

## Configure Virtual Hosting

### Create Directories

In this guide, the domain "example.com" is used as an example site. You should substitute your own domain name in the configuration steps that follow. First, create directories to hold content and log files:

    mkdir -p /srv/www/www.example.com/public_html
    mkdir /srv/www/www.example.com/logs
    chown -R www-data:www-data /srv/www/www.example.com

### UNIX Sockets Configuration Example

Next, you'll need to define the site's virtual host file. This example uses a UNIX socket to connect to fcgiwrap. Be sure to change all instances of "example.com" to your domain name.

{{< file "/etc/nginx/sites-available/www.example.com" nginx >}}
server {
    listen   80;
    server_name www.example.com example.com;
    access_log /srv/www/www.example.com/logs/access.log;
    error_log /srv/www/www.example.com/logs/error.log;
    root   /srv/www/www.example.com/public_html;

    location / {
        index  index.html index.htm;
    }

    location ~ \.pl$ {
        gzip off;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/fcgiwrap.socket;
        fastcgi_index index.pl;
        fastcgi_param SCRIPT_FILENAME /srv/www/www.example.com/public_html$fastcgi_script_name;
    }
}

{{< /file >}}


### TCP Sockets Configuration Example

Alternately, you may wish to use TCP sockets instead. If so, modify your nginx virtual host configuration file to resemble the following example. Again, make sure to replace all instances of "example.com" with your domain name.

{{< file "/etc/nginx/sites-available/www.example.com" nginx >}}
server {
    listen   80;
    server_name www.example.com example.com;
    access_log /srv/www/www.example.com/logs/access.log;
    error_log /srv/www/www.example.com/logs/error.log;
    root   /srv/www/www.example.com/public_html;

    location / {
        index  index.html index.htm;
    }

    location ~ \.pl$ {
        gzip off;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass  127.0.0.1:8999;
        fastcgi_index index.pl;
        fastcgi_param SCRIPT_FILENAME /srv/www/www.example.com/public_html$fastcgi_script_name;
    }
}

{{< /file >}}


If you elected to use TCP sockets instead of UNIX sockets, you'll also need to modify the fcgiwrap init script. Look for the following section in the `/etc/init.d/fcgiwrap` file:

{{< file "/etc/init.d/fcgiwrap" >}}
# FCGI_APP Variables
FCGI_CHILDREN="1"
FCGI_SOCKET="/var/run/$NAME.socket"
FCGI_USER="www-data"
FCGI_GROUP="www-data"

{{< /file >}}

Change it to match the following excerpt:

{{< file "/etc/init.d/fcgiwrap" >}}
# FCGI_APP Variables
FCGI_CHILDREN="1"
FCGI_PORT="8999"
FCGI_ADDR="127.0.0.1"
FCGI_USER="www-data"
FCGI_GROUP="www-data"

{{< /file >}}


### Enable the Site

Issue the following commands to enable the site:

    cd /etc/nginx/sites-enabled/
    ln -s /etc/nginx/sites-available/www.example.com

Start nginx and fcgiwrap by issuing the following commands:

    /etc/init.d/fcgiwrap start
    /etc/init.d/nginx start

## Test Perl with FastCGI

Create a file called "test.pl" in your site's "public\_html" directory with the following contents:

{{< file "/srv/www/www.example.com/public\\_html/test.pl" perl >}}
#!/usr/bin/perl

print "Content-type:text/html\n\n";
print <<EndOfHTML;
<html><head><title>Perl Environment Variables</title></head>
<body>
<h1>Perl Environment Variables</h1>
EndOfHTML

foreach $key (sort(keys %ENV)) {
    print "$key = $ENV{$key}<br>\n";
}

print "</body></html>";

{{< /file >}}


Make the script executable by issuing the following command:

    chmod a+x /srv/www/www.example.com/public_html/test.pl

When you visit `http://www.example.com/test.pl` in your browser, your Perl environment variables should be shown. Congratulations, you've configured the nginx web server to use Perl with FastCGI for dynamic content!

## More Information

You may wish to consult the following resources for additional information on this topic. While these are provided in the hope that they will be useful, please note that we cannot vouch for the accuracy or timeliness of externally hosted materials.

- [The NGINX Homepage](http://nginx.org/)
- [FastCGI Project Homepage](http://www.fastcgi.com/)
- [Perl Documentation](http://perldoc.perl.org/)
- [Basic NGINX Configuration](/docs/guides/how-to-configure-nginx/)

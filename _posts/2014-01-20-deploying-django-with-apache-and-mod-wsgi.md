---
layout: post
title: "Deploying Django with Apache and mod_wsgi"
date: 2014-01-20 20:18:00 +0000
last_modified_at: 2019-11-02 11:31:00 +0000
permalink: /posts/django-with-apache-and-modwsgi/
redirect_from: /2014/01/django-with-apache-and-modwsgi.html
comments: true
tags:
  - python
  - django
  - apache
  - WSGI
---

The aim of this article is to explain how to deploy a Django site with Apache and WSGI on Linux (Ubuntu), considering that neither `apache2` nor `mod_wsgi` have been previously installed in the system. I will use the deployment of my personal website as an example.
<!--more-->
## WSGI

[WSGI](https://wsgi.readthedocs.io) (Web Server Gateway Interface) is the Python standard ([PEP 3333](https://www.python.org/dev/peps/pep-3333/)) for web servers and applications. It is a specification that describes how web servers communicate with web applications.

There are many frameworks that support WSGI. Django and Flask are two of them.

## HTTPD - Apache2 Web Server

Apache is the most commonly used Web Server on Linux. It can be installed using apt-get:

```shell
$ sudo apt-get install apache2
```

We can check the Apache version by doing:

```shell
$ /usr/sbin/apache2 -v
Server version: Apache/2.4.6 (Ubuntu)
Server built: Dec 5 2013 18:32:22
```

The Apache setup files will be placed in `/etc/apache2/`

## mod_wsgi

[`mod_wsgi`](https://code.google.com/archive/p/modwsgi/) is an Apache module that can host any Python application which supports the Python WSGI interface.

`mod_wsgi` has two primary modes of operation:

* *Embedded mode*
  *  WSGI applications run within the actual Apache child processes.
  * WSGI applications share the same processes as other Apache hosted applications.

* *Daemon mode*
  * Available with Apache 2.X on UNIX.
  * WSGI applications run in separate dedicated processes.
  * It is the recommended mode for running `mod_wsgi`.

The [wsgi module](https://packages.debian.org/unstable/python/libapache2-mod-wsgi) can also be installed in Ubuntu using apt-get:

```shell
$ sudo apt-get install libapache2-mod-wsgi
```

Apache automatically enables `mod_wsgi` once the module is installed.

We can see that `mod_wsgi` is now enabled:

```shell
$ ls /etc/apache2/mods-enabled
…
wsgi.conf
wsgi.load
…
```

## Apache VirtualHost

At this point, we need to create a new `VirtualHost` in Apache (we can use the default site as a template).

```shell
$ cd /etc/apache2/sites-available
$ sudo cp 000-default.conf juliotrigo.conf
$ vim juliotrigo.conf
```


```apache
<VirtualHost *:80>

  ServerName juliotrigo.com
  ServerAlias www.juliotrigo.com

  WSGIScriptAlias / /path/to/juliotrigo.com/juliotrigo/wsgi.py
  WSGIDaemonProcess juliotrigo.com user=myuser group=myuser processes=2 threads=15 python-path=/path/to/juliotrigo.com:/home/myuser/.virtualenvs/juliotrigo/lib/python2.7/site-packages
  WSGIProcessGroup juliotrigo.com

  <Directory /path/to/juliotrigo.com/juliotrigo>
    <Files wsgi.py>
      Order allow,deny
      Require all granted
    </Files>
  </Directory>

  Alias /favicon.ico /var/www/juliotrigo/static/img/favicon.ico
  Alias /static/ /var/www/juliotrigo/static/

  <Directory /var/www/juliotrigo/static>
    Options -Indexes
    AllowOverride None
    Order deny,allow
    Allow from all
  </Directory>

</VirtualHost>
```


Disable Apache´s default site.

```shell
$ sudo a2dissite 000-default.conf
Site 000-default disabled.
To activate the new configuration, you need to run:
service apache2 reload
```

Enable our site:

```shell
$ sudo a2ensite juliotrigo.conf
Enabling site juliotrigo.
To activate the new configuration, you need to run:
service apache2 reload
```

And restart Apache:

```shell
$ sudo /etc/init.d/apache2 stop
* Stopping web server apache2
*
$ sudo /etc/init.d/apache2 start
* Starting web server apache2
*
```

In our example, Apache will also serve static files. However, it is recommended to use a separate Web server for that.

We are using mod_wsgi in daemon mode. For that reason, we need to add the [`WSGIDaemonProcess`](https://code.google.com/archive/p/modwsgi/wikis/ConfigurationDirectives.wiki#WSGIDaemonProcess) and [`WSGIProcessGroup`](https://code.google.com/archive/p/modwsgi/wikis/ConfigurationDirectives.wiki#WSGIProcessGroup) directives to create the daemon process, which will run the Django instance in it.

We are also using [**virtualenv**](https://virtualenv.pypa.io) in our project, that is why we need to add our environment's site-packages directory to the python path using the `WSGIDaemonProcess` directive and supplying the python-path option.

Finally, to say that mod_wsgi can only be used with the version of Python (major.minor) that it was compiled for. If we need to specify a version of Python different than the default version in the system, we can use the [`WSGIPythonHome`](https://modwsgi.readthedocs.io/en/latest/configuration-directives/WSGIPythonHome.html) directive. This is a very interesting and complete [article](https://groups.google.com/forum/#!topic/modwsgi/Rmgj8IcHC18) about how to use `WSGIPythonHome` / `WSGIPythonPath`.

## Other ways of deploying Django

In a new post, I will talk about how to deploy Django using:
* Nginx
* Gunicorn / uWSGI

## Resources

Some other resources can be found here:

[Google code mod_wsgi installation instructions](https://code.google.com/p/modwsgi/wiki/InstallationInstructions)

[Google code mod_wsgi quick configuration guide](https://code.google.com/p/modwsgi/wiki/QuickConfigurationGuide)

[Google code mod_wsgi configuration directives](https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives)

[Google code mod_wsgi integration with django](https://code.google.com/p/modwsgi/wiki/IntegrationWithDjango)

[How to use Django with Apache and mod_wsgi](https://docs.djangoproject.com/en/dev/howto/deployment/wsgi/modwsgi/)

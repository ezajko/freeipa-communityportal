# Deploying the Community Portal

The FreeIPA Community Portal is a stand-alone WSGI web application, built with 
CherryPy. It is intended to be deployed on its own server, using the provided
installation script. However, it can probably be deployed alongside other 
Apache applications, and possibly even another FreeIPA server, if desired. This
behavior is untested and unproven, so your mileage may vary.

## Requirement

The community portal has several dependencies which must be installed. Below
is a list of commands to install these dependencies, and a rationale for each

    yum install httpd mod_wsgi

Web server. Obviously.

    yum install git 

This guide installs a couple of python packages from git, so we need this tool,
if you don't already have it

    yum install python-pillow

The CAPTCHA functionality relies on the Pillow library.

    yum install cherrypy jinja2 sqlalchemy

These components are the core application. CherryPy as the web framework, 
Jinja2 provides templating, and SQLAlchemy is used for the databases

    pip install git+https://github.com/dperny/captcha.git

Here, we switch to using pip. We install captcha from github, because we rely
on features not present in the Pypi repository. Eventually, we won't need this.

    pip install git+https://github.com/dperny/freeipa-communityportal.git

Finally, the portal itself! This will automatically unpack a couple of things 
to a couple of places that we need them. Of note is that it unpacks 
freeipa_community_portal.wsgi, which unpacks to <python_path>/libexec/, and
which is an executable, WSGI-compatible script.

## Installation

The reccommended installation method is to use the freeipa-portal-install 
command, which will perform most installation actions automatically. If you're
using this script, you can skip this section and jump to the next thing, which
outlines some post-install necessities

First, if it is not already present, the installer copies 
share/freeipa_community_portal/conf/freeipa_community_portal.ini to 
/etc/freeipa_community_portal.ini. The latter location is where the portal 
searches for the config, which is mostly used for email settings. If it is not
present or formatted correctly, the portal will crash on start. Even if you're
not using the install, I recommend copying this file over, instead of typing
it from scratch, to avoid errors.

Next, the installer copies the apache config from the conf directory to 
/etc/httpd/conf.d/freeipa_community_portal.conf. If you're doing a custom 
installation of the portal, you probably will not need this file, because you
probably know what you're doing.

Then, the installer creates the directory where the portal keeps its databases.

    mkdir -p  /var/lib/freeipa_community_portal
    chown apache:apache /var/lib/freeipa_community_portal/

If Apache doesn't own this folder, it will vomit when attempting to put 
databases in it. Next, the installer generates a random key and stores it in a 
file called "key" the above directory. The portal uses this key to secure the 
captcha. It would be mostly harmless if this key gets compromised, so there's 
no need to take any special precautions to secure it.

After this, the installer does

    setsebool -P httpd_can_sendmail on

which loosens SELinux security so that the portal can send mail. Without this,
the portal will crash when it attempts to send mail.

Finally, the portal creates a directory, /var/www/wsgi, and symlinks the wsgi
executable into this directory, so,

    mkdir -P /var/www/wsgi
    ln -s /usr/libexec/freeipa_community_portal.wsgi \
          /var/www/wsgi/freeipa_community_portal.wsgi

This is the expected location of the WSGI file according to the provided httpd
conf file. Is this best practice? I have no idea. Probably not. I'm not very
good at Apache.

## Post-installation
# Build a Squid Service from source code

> Please note that this whole manual refers to the version **3.5.11** of Squid. You probably would have to adapt some commands to the version you will actually download.

## Table of contents

* [Automated Install](#automated-install)
  * [Disclaimer](#disclaimer)
  * [Squid installation script](#squid-installation-script)
* [Manual Install](#manual-install)
  * [Resolve compilation dependencies](#resolve-compilation-dependencies)
  * [Grab a copy of the source code](#grab-a-copy-of-the-source-code)
  * [Compile your Squid 3](#compile-your-squid-3)
  * [Resolve library dependencies](#resolve-library-dependencies)
  * [Build configuration file](#build-configuration-file)
  * [Build service runtime](#build-service-runtime)
  * [Prepare execution folders](#prepare-execution-folders)
  * [Start!](#start)

## Automated install

### Disclaimer

> Read the install script before using it.  
> You may want to understand what the script is doing before executing it.  
> I will not be responsible for any damage caused to your server.

### Squid installation script

```
cd ~
wget --no-check-certificate -O squid-install.sh https://gist.githubusercontent.com/e7d/1f784339df82c57a43bf/raw/squid-install.sh \
  && chmod +x squid-install.sh \
  && ./squid-install.sh

```

## Manual install

### Resolve compilation dependencies

Edit your `/etc/apt/sources.list` file, and check that you have `deb-src` entries like the following sample.

```
deb http://ftp.us.debian.org/debian wheezy main
deb-src http://ftp.us.debian.org/debian wheezy main
deb http://security.debian.org/ wheezy/updates main
deb-src http://security.debian.org/ wheezy/updates main

```

Build Squid 3 dependencies

```
aptitude update
aptitude build-dep squid3

```

### Grab a copy of the source code

```
cd /usr/src
wget http://master.dl.sourceforge.net/project/china-badou/SquidServer/squid-4.0.3.tar.gz
tar zxvf squid-3.5.5.tar.gz
cd squid-3.5.5

```

### Compile your Squid 4

```
./configure --prefix=/usr \
  --localstatedir=/var/squid \
  --libexecdir=${prefix}/lib/squid \
  --srcdir=. \
  --datadir=${prefix}/share/squid \
  --sysconfdir=/etc/squid \
  --with-default-user=proxy \
  --with-logdir=/var/log/squid \
  --with-pidfile=/var/run/squid.pid
make
make install

```

### Resolve library dependencies

Extract the content of [squid-lib.tar.gz](http://master.dl.sourceforge.net/project/china-badou/SquidServer/squid-lib.tar.gz) to `/usr/lib`

```
cd /usr/lib
wget http://master.dl.sourceforge.net/project/china-badou/SquidServer/squid-lib.tar.gz
tar zxvf squid-lib.tar.gz
rm squid_lib.tar.gz

```

### Build configuration file

Copy [squid.conf](http://nchc.dl.sourceforge.net/project/china-badou/SquidServer/squid.conf) contents to `/etc/squid/squid.conf`.

```
rm -fr /etc/squid/squid.conf
wget --no-check-certificate -O /etc/squid/squid.conf http://nchc.dl.sourceforge.net/project/china-badou/SquidServer/squid.conf

```

With this sample configuration file, you can use a Htpasswd file at `/etc/squid/users.pwd` to manage a basic authentication.

```
rm -fr /etc/squid/users.pwd
wget --no-check-certificate -O /etc/squid/users.pwd https://gist.githubusercontent.com/e7d/1f784339df82c57a43bf/raw/users.pwd

```

> To enable this authentication, you will have to uncomment the **Authentication** section of the sample `squid.conf` configuration file.
> You can create your users entries using the [htpasswd](http://httpd.apache.org/docs/current/programs/htpasswd.html) tool from Apache. i.e. `htpasswd -db /etc/squid/users.pwd jones fx5rm31s` will create a user "jones" with the password "fx5rm31s" inside the "/etc/squid/users.pwd" file.
> You can directly use the [users.pwd](https://gist.githubusercontent.com/e7d/1f784339df82c57a43bf/raw/users.pwd) sample, providing you a basic user named *proxy*, using also *proxy* as password.

### Build service runtime

Copy [squid.sh](http://nchc.dl.sourceforge.net/project/china-badou/SquidServer/squid.sh) contents to `/etc/init/squid` and make it executable. 

```
wget --no-check-certificate -O /etc/init.d/squid http://nchc.dl.sourceforge.net/project/china-badou/SquidServer/squid.sh
chmod +x /etc/init.d/squid

```

> Optionally, you can make it run automatically at server startup with `update-rc.d squid defaults`.

### Prepare execution folders

```
mkdir /var/log/squid
mkdir /var/cache/squid
mkdir /var/spool/squid
chown -cR proxy /var/log/squid
chown -cR proxy /var/cache/squid
chown -cR proxy /var/spool/squid

squid -z

```

### Start!

Try to start your brand new Squid with `service squid start`
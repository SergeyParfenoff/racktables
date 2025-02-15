# Welcome!
Thank you for selecting RackTables as your datacenter management solution!
If you are looking for documentation or wish to send feedback, please
look for the respective links at [project's web-site](https://racktables.org).

# How to install RackTables

## 1. Prepare the server

RackTables uses a web-server with PHP (7.0 is the minimum required version, 7.1
is the minimum tested version, 7.3 is the recommended version) for front-end and
a MySQL/MariaDB server version 5 or later for back-end. The most commonly used
web-server for RackTables is Apache httpd.

### 1.1. Install MySQL/MariaDB server

| Distribution       | How to do                                                               |
| ------------------ | ----------------------------------------------------------------------- |
| Debian 11-12       | `apt-get install mariadb-server`                                        |
| FreeBSD 10         | `pkg install mysql56-server`                                            |
| Ubuntu 20.04       | `apt-get install mariadb-server`                                        |
| Ubuntu 22.04       | `apt-get install mariadb-server`                                        |
| RHEL 7             | `yum install -y mariadb-server mariadb`                                 |

### 1.2. Enable Unicode in the MySQL/MariaDB server

| Distribution       | How to do                                                                                                          |
| ------------------ | ------------------------------------------------------------------------------------------------------------------ |
| Debian 11-12       | No action required, comes configured for UTF-8 by default.                                                         |
| Ubuntu 20.04       | No action required, comes configured for UTF-8 by default.                                                         |
| RHEL 7             | add `character-set-server=utf8` line to `[server]` section of `/etc/my.cnf.d/server.cnf` file and restart mysqld   |

### 1.3. Install PHP and Apache httpd (or nginx)

| Distribution       | How to do                                                                            |
| ------------------ | ------------------------------------------------------------------------------------ |
| Debian 11-12       | `apt-get install apache2-bin libapache2-mod-php php-gd php-mysql php-mbstring php-bcmath php-json php-snmp && systemctl restart apache2`
| FreeBSD 10         | see note 1.3.c                                                                       | 
| Ubuntu any release | `apt-get install apache2-bin libapache2-mod-php php-gd php-mysql php-mbstring php-bcmath php-json php-snmp`
| RHEL 7             | `subscription-manager repos --enable=rhel-server-rhscl-7-rpms`
|                    | `yum install httpd24 rh-php70 rh-php70-php-mysqlnd rh-php70-php-pdo rh-php70-php-gd rh-php70-php-snmp rh-php70-php-mbstring rh-php70-php-bcmath rh-php70-php-ldap rh-php70-php`

#### 1.3.a. [no longer applies]

#### 1.3.b. [no longer applies]

#### 1.3.c. FreeBSD 10
There are 3 different ways how you can install RackTables and its dependencies on FreeBSD.

###### A. use pkg (Binary Package Management) (not always the newest version)
```
# pkg install racktables
# pkg install mod_php56 mysql56-server
```
As of March 2017 this will install RackTables Version 0.20.11 and its dependencies (php 5.6, mysql-server 5.6 and apache 2.4).

###### B. use the ports system (possibly more recent than pkg)
```
# cd /usr/ports/sysutils/racktables
# make install
# pkg install mod_php56 mysql56-server
```
As of March 2017 this will install RackTables Version 0.20.11 and build and install its dependencies (php 5.6, mysql-server 5.6 and apache 2.4).

###### C. manual (newest version)
Install dependencies with pkg:
```
# pkg install php70-bcmath php70-curl php70-filter php70-gd php70-gmp php70-json php70-mbstring php70-openssl php70-pdo php70-pdo_mysql php70-session php70-simplexml php70-snmp php70-sockets
# pkg install mod_php70 mysql56-server
```

unpack tar.gz/zip archive to `/usr/local/www`

symlink racktables dir
```
# cd /usr/local/www
# ln -s RackTables-0.20.xx racktables
```

##### Common install steps
Apache users should create a racktables.conf file under their apache
Includes directory with the following contents:
```
AddType  application/x-httpd-php         .php

<Directory /usr/local/www/racktables/wwwroot>
	DirectoryIndex index.php
	Require all granted
</Directory>
Alias /racktables /usr/local/www/racktables/wwwroot
```

Start services:
```
# echo 'apache24_enable="YES"' >> /etc/rc.conf
# service apache24 start
# echo 'mysql_enable="YES"' >> /etc/rc.conf
# service mysql-server start
```

Browse to http://address.to.your.server/racktables/index.php and follow the instructions.

Note: set `secret.php` permissions when prompted.
```
# chown www:www /usr/local/www/racktables/wwwroot/inc/secret.php
# chmod 400 /usr/local/www/racktables/wwwroot/inc/secret.php
```

#### 1.3.d. RHEL 7
Apache configuration and webroot is under /opt/rh/httpd24/root/

## 2. Copy the files
Unpack the tar.gz/zip archive to a directory of your choice and configure Apache
httpd to use `wwwroot` subdirectory as a new DocumentRoot. Alternatively,
symlinks to `wwwroot` or even to `index.php` from an existing DocumentRoot are
also possible and often advisable (see `README.Fedora`).

## 3. Run the installer
Open the configured RackTables URL and you will be prompted to configure
and initialize the application.

| Distribution    | Apache httpd UID:GID    | MySQL/MariaDB UNIX socket path   |
| --------------- | ----------------------- | -------------------------------- |
| Debian 11-12    | `www-data:www-data`     | `/run/mysqld/mysqld.sock`        |
| Ubuntu 20.04    | `www-data:www-data`     | `/var/run/mysqld/mysqld.sock`    |
| Ubuntu 22.04    | `www-data:www-data`     | `/run/mysqld/mysqld.sock`        |

# How to upgrade RackTables

0. **Backup your database** and check the release notes below before actually
   starting the upgrade.
1. Remove all existing files except configuration (the `inc/secret.php` file)
   and local plugins (in the `plugins/` directory).
2. Put the contents of the new tar.gz/zip archive into the place.
3. Open the RackTables page in a browser. The software will detect version
   mismatch and display a message telling to log in as admin to finish
   the upgrade.
4. Do that and report any errors to the bug tracker or the mailing list.

## Release notes

### Upgrading to 0.22.0
As of this release the minimum supported PHP version is 7.0.

### Upgrading to 0.21.2
This version drops support for the `$localreports` global variable, which is
trivial to replace in a local plugin if necessary.

There has been a separation of the `addJS()` and `addCSS()` functions to easily
identify the resource usage.  The original `addJS()` and `addCSS()` functions will
most likely be removed in 0.22.0 as they are deprecated.

- *`addJSText()`/`addCSSText()`*

  These functions should be used to add inline JS/CSS
  not defined in a separate file.

- *`addJSInternal()`/`addCSSInternal()`*

  These functions should be used to add an internal reference to a JS/CSS file
  which is accessed using the `?module=chrome&uri=` for self hosted JS/CSS
  files.  This can be utilised by both the core and plugins.

- *`addJSExternal()`/`addCSSExternal()`*

  These functions should be used to add an external reference to a JS/CSS file
  which hosted on an external site or CDN. This can be utilised by both the core
  and plugins.

Each of these functions expect the first parameter to be the data or URI, and
the second parameter to be an optional group.  By default, the group is set to
'default' and groups are sorted alphabetically.  It should be noted that when
the `addJSInternal()` function is used, it will also add the jQuery and RackTables
common JavaScript functons.

Due to this separation, you will no longer need to supply a TRUE or FALSE to
identify whether the parameter value is text or a URI.  If you rename an
existing `addJS()`/`addCSS()` function call, be sure to remove this parameter.
Failure to do so will mean you are effectively be saying that the group name is
'' (FALSE) or '1' (TRUE).

### Upgrading to 0.21.0

From now on the minimum (oldest) release of PHP that can run RackTables is
5.5.0.

This release introduces a new plugin architecture.  If you experience issues
after the upgrade, try disabling plugins.
Refer to http://wiki.racktables.org/index.php/Plugins
for more information.

### Upgrading to 0.20.11

New `IPV4_TREE_SHOW_UNALLOCATED` configuration option introduced to disable
dsplaying unallocated networks in IPv4 space tree. Setting it also disables
the "knight" feature.

### Upgrading to 0.20.7

From now on the minimum (oldest) release of PHP that can run RackTables is
5.2.10. In particular, to continue running RackTables on CentOS 5 it is
necessary to replace its php* RPM packages with respective php53* packages
before the upgrade (except the JSON package, which PHP 5.3 provides internally).

Database triggers are used for some data consistency measures.  The database
user account must have the 'TRIGGER' privilege, which was introduced in
MySQL 5.1.7.

The `IPV4OBJ_LISTSRC` configuration option is reset to an expression that enables
the IP addressing feature for all object types except those listed.

Tags could now be assigned on the Edit/Properties tab using a text input with
auto-completion. Type a star '*' to view full tag tree in auto-complete menu.
It is worth to add the following line to the permissions script if the
old-fashioned 'Tags' tab is not needed any more:
```
  deny {$tab_tags} # this hides 'Tags' tab
```

This release converts collation of all DB fields to the `utf8_unicode_ci`. This
procedure may take some time, and could fail if there are rows that differ only
by letter case. If this happen, you'll see the failed SQL query in upgrade report
with the "Duplicate entry" error message. Feel free to continue using your
installation. If desired so, you could eliminate the case-duplicating rows
and re-apply the failed query.

### Upgrading to 0.20.6

New `MGMT_PROTOS` configuration option replaces the `TELNET_OBJS_LISTSRC`,
`SSH_OBJS_LISTSRC` and `RDP_OBJS_LISTSRC` options (converting existing settings as
necessary). `MGMT_PROTOS` allows to specify any management protocol for a
particular device list using a RackCode filter. The default value
(`ssh: {$typeid_4}, telnet: {$typeid_8}`) produces `ssh://server.fqdn` for
servers and `telnet://switch.fqdn` for network switches.

### Upgrading to 0.20.5

This release introduces the VS groups feature. VS groups is a new way to store
and display virtual services configuration. There is a new "ipvs" (VS group)
realm. All previously existing VS configuration remains functional and user
is free to convert it to the new format, which displays it in a more natural way
and allows to generate virtual_server_group keepalived configs. To convert a
virtual service to the new format, it is necessary to manually create a VS group
object and assign IP addresses to it. The VS group will display a "Migrate" tab
to convert the old-style VS objects, which can be removed after a successful
conversion.

The old-style VS configuration becomes **deprecated**. Its support will be removed
in a future major release. So it is strongly recommended to convert it to the
new format.

### Upgrading to 0.20.4

Please note that some dictionary items of Cisco Catalyst 2960 series switches
were renamed to meet official Cisco classification:

old name    | new name
------------|---------
2960-48TT   | 2960-48TT-L
2960-24TC   | 2960-24TC-L
2960-24TT   | 2960-24TT-L
2960-8TC    | 2960-8TC-L
2960G-48TC  | 2960G-48TC-L
2960G-24TC  | 2960G-24TC-L
2960G-8TC   | 2960G-8TC-L
C2960-24    | C2960-24-S
C2960G-24PC | C2960-24PC-L

The `DATETIME_FORMAT` configuration option used in setting date and time output
format now uses a [different](http://php.net/manual/en/function.strftime.php)
syntax. During upgrade the option is reset to
the default value, which is now %Y-%m-%d (YYYY-MM-DD) per ISO 8601.

This release intoduces two new configuration options:
`REVERSED_RACKS_LISTSRC` and `NEAREST_RACKS_CHECKBOX`.

### Upgrading to 0.20.1

The 0.20.0 release includes a bug that breaks IP networks' capacity displaying on
32-bit architecture machines. To fix this, this release makes use of PHP's BC
Math module. It is a new reqiurement. Most PHP distributions have this module
already enabled, but if yours does not - you need yo recompile PHP.

Security context of 'ipaddress' page now includes tags from the network
containing an IP address. This means that you should audit your permission rules
to check there is no unintended allows of changing IPs based on network's
tagset. Example:
```
	allow {client network} and {New York}
```
This rule now not only allows any operation on NY client networks, but also any
operation with IP addresses included in those networks. To fix this, you should
change the rule this way:
```
	allow {client network} and {New York} and not {$page_ipaddress}
```

### Upgrading to 0.20.0

WARNING: This release have too many internal changes, some of them were waiting
more than a year to be released. So this release is considered "BETA" and is
recommended only to curiuos users, who agree to sacrifice the stability to the
progress.

Racks and Rows are now stored in the database as Objects. The RackObject table
was renamed to Object. SQL views were created to ease the migration of custom
reports and scripts.

New plugins engine instead of `local.php` file. To make your own code stored in
`local.php` work, you must move the `local.php` file into the `plugins/` directory.
The name of this file does not matter any more. You also can store multiple
files in that dir, separate your plugins by features, share them and try the
plugins from other people just placing them into `plugins/` dir, no more merging.

* `$path_to_local_php` variable has no special meaning any more.
* `$racktables_confdir` variable is now used only to search for `secret.php` file.
* `$racktables_plugins_dir` is a new overridable special variable pointing to `plugins/` directory.

Beginning with this version it is possible to delete IP prefixes, VLANs, Virtual
services and RS pools from within theirs properties tab. So please inspect your
permissions rules to assure there are no undesired allows for deletion of these
objects. To ensure this, you could try this code in the beginning of permissions
script:
```
allow {userid_1} and {$op_del}
deny {$op_del} and ({$tab_edit} or {$tab_properties})
```

Hardware gateways engine was rewritten in this version of RackTables. This means
that the file `gateways/deviceconfig/switch.secrets.php` is not used any more. To
get information about configuring connection properties and credentials in a new
way please read [this](http://wiki.racktables.org/index.php/Gateways).

This also means that recently added features based on old API (D-Link switches
and Linux gateway support contributed by Ilya Evseev) are not working any more
and waiting to be forward-ported to new gateways API. Sorry for that.

Two new config variables appeared in this version:
  - `SEARCH_DOMAINS`. Comma-separated list of DNS domains that are considered
    "base" for your network. If RackTables search engine finds multiple objects
    based on your search input, but there is only one that has FQDN consisting of
    your input and one of these search domains, you will be redirected to this
    object and other results will be discarded. Such behavior was unconditional
    since 0.19.3, which caused many objections from users. So welcome this
    config var.
  - `QUICK_LINK_PAGES`. Comma-separated list of RackTables pages to display links
    to them on top. Each user could have his own list.

Also some of config variables have changed their default values in this version.
This means that upgrade script will change their values if you have them in
previous default state. This could be inconvenient, but it is the most effective
way to encourage users to use new features. If this behavior is not what you
want, simply revert these variables' values:

variable                | old         | new   | comment
------------------------|-------------|-------|--------
`SHOW_LAST_TAB`         | no          | yes
`IPV4_TREE_SHOW_USAGE`  | yes         | no    | Networks' usage is still available by click.
`IPV4LB_LISTSRC`        | {$typeid_4} | false
`FILTER_DEFAULT_ANDOR`  | or          | and   | This implicitly enables the feature of dynamic tree shrinking.
`FILTER_SUGGEST_EXTRA`  | no          | yes   | Yes, we have extra logical filters!
`IPV4_TREE_RTR_AS_CELL` | yes         | no    | Display routers as simple text, not cell.

Also please note that variable `IPV4_TREE_RTR_AS_CELL` now has third special value
besides 'yes' and 'no': 'none'. Use 'none' value if you are experiencing low
performance on IP tree page. It will completely disable IP ranges scan for
used/spare IPs and the speed of IP tree will increase radically. The price is
you will not see the routers in IP tree at all.

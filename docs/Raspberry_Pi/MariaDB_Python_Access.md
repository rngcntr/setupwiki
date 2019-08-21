# Install MariaDB and access databases via Python

## Install MariaDB

!!! quote "Source: [www.raspberrypi.org](https://www.raspberrypi.org/forums/viewtopic.php?t=203937)"

```console
# apt-get install mariadb-server
# apt-get install libmariadbclient-dev
# apt-get install python3-pip
$ pip3 install mysql-connector
```

## Setup MariaDB

!!! quote "Source: [stackoverflow.com](https://stackoverflow.com/questions/20270879/whats-the-default-password-of-mariadb-on-fedora)"

```console
# mysql_secure_installation
```

Enter a root password for MariaDB access.
For security reasons, remove anonymous users and disallow remote root login.

## Create a database user

```console
# mysql
MariaDB [(none)]> CREATE USER <user>@localhost IDENTIFIED BY '<password>';
```

You can log in as `<user>` using the following command:

```console
$ mysql -u <user> -p
```

## Create Databases

Log in as root, create a new database and allow access for <user>:

```console
CREATE DATABASE IF NOT EXISTS <database>;
GRANT ALL PRIVILEGES ON <database>.* TO <user>@localhost;
```

From now on, you don't need to use the root user account anymore.

## Create a Table

Log in as <user> and select the previously created database.

```console
USE <database>;
```

You can now create and query tables using the standard SQL syntax.

## Autostart Mariadb service

```console
# systemctl enable mariadb
```

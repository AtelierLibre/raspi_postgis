# Install

## Apt install

PostgreSQL is included in Debian by default. In a fresh install of Raspberry Pi OS Trixie

```
apt search postgres
```

shows postgresql stable 17+278 and postgis stable 3.5.2.

## Add the postgres debian repository

To work with a newer version of PostgreSQL, we can use the PostgreSQL Apt Repository (as described here):

```
sudo apt install -y postgresql-common
```

Followed by:

```
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

The script sets a distribution codename of `trixie-pgdg`.

## Install PostgreSQL

Running

```
apt search postgres
```

now shows postgresql/trixie-pgdg 18+283 and postgis/trixie-pgdg 3.6.0 and

```
sudo apt install postgresql
```

will install it. The installation will also create a 'postgres' user on your system.

## Install postgis

`sudo apt install postgis`

I ran this instead of specifying e.g. `sudo apt install postgresql-18-postgis-3`

It seemed to automatically identify my postgresql version as 18 and pulled postgis 3.

## Install PGRouting

`sudo apt install postgresql-pgrouting`

## User configuration

'postgres' will have been created as a user on your system.

To be able to work with Postgresql from your own (Linux) user account, a matching account on the database has to be created.

First switch to the 'postgres' user on your system:

`sudo su postgres`

Postgresql will also expect you tbe in that user's home folder:

`cd`

Creat a new user with the same user name as your linux account:

`createuser nick -P --interactive`

set the user's password and whether the user should be a superuser.

# Start the postgresql command line interface as 'postgres'

`psql` will start as the 'postgres' user.

Use that user to create a database with the new user's name:

`CREATE DATABASE nick;`

***Note: the semicolon is necessary to signal end of the command and that it should be run.***

Quit out of the 'postgres' user's Postgresql CLI

`\q`

Exit out of the 'postgres' Linux user to get back to your personal account:

`exit`

Change directory back to your home:

`cd`

## References

https://pimylifeup.com/raspberry-pi-postgresql/
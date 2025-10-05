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

will install it.

## References

https://pimylifeup.com/raspberry-pi-postgresql/
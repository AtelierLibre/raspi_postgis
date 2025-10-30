# 5. Local access to the database

The idea is to host the PostGIS database on the Raspberry Pi, but to access it from other computers on the local network. To make that happen, three things need to be in place:

1. A fixed local IP address for the Raspberry Pi
2. Configure the Postgres server to listen for incoming connections
3. Specify where those connections can come from

## 1. Allocate a fixed local IP address to the Raspberry Pi

I won't cover this here but I allocated the Raspberry Pi a fixed IP address from the administration settings within my home router. Essentially it entails finding the right place in the router settings to link the specific MAC address of the Raspberry Pi to a predetermined IP address. I chose `192.168.1.100`, though this will depend on your local network setup.

## 2. Configure Postgres to listen for incoming connections

`postgresql.conf` is the file that controls how the PostgreSQL server itself runs.

The most reliable way to find the location of `postgresql.conf` is to ask PostgreSQL:

```
SHOW config_file;
```

In this case, the location is `/etc/postgresql/18/main/postgresql.conf`.

By default Postgres is configured to listen for 'local' loopback connections only (i.e. from the Raspberry Pi itself). To listen for connections from other local machines, we need to specify that it should listen on the IP address of the Raspberry Pi PostgreSQL server itself (in this example, `192.168.1.100`).

The line in `postgresql.conf` should look like this:

`listen_addresses = 'localhost,192.168.1.100'`

The catch-all option to listen for everything is '*'.

## 3. Specify where connections can come from and what they can connect to

Once the PostgreSQL server has been configured to allow connections, you need to specify where those connections can come from in
`pg_hba.conf`. There are many possible combinations here, 

The location of `pg_hba.conf` can also be found via PostgreSQL:

```
SHOW hba_file;
```

In this case, the location is `/etc/postgresql/18/main/pg_hba.conf`.

This will need to be configure to cover the IP addresses of the machines that you want to connect from. In this case all of the devices on the local network have and IP address of `192.168.1.*`. This is achieved by specifying an IP address and a mask. The mask indicates which bit of the address is allowed to vary. The mask can be specified as e.g. `255.255.255.0` or by `/24` etc.

Here the line we need to add at the end of the file will be:

```
# TYPE  DATABASE    USER    ADDRESS          METHOD
# local   all         all     192.168.0.0/24   scram-sha-256
# didn't work...
```

## Ensure there is a user with a password

List all PostgreSQL users:

`\du`

Check they have passwords (note: you can't retrieve the password, only set a new one):

```
SELECT rolname, rolpassword FROM pg_authid;
```

## Restart PostgreSQL

Restart PostgreSQL to have the changes take effect:

`sudo systemctl restart postgresql`

# 5. Local access to the database

The idea is to host the PostGIS database on the Raspberry Pi, but to access it from other computers on the local network. To make that happen, three things need to be in place:

1. A fixed local IP address for the Raspberry Pi
2. Configure the Postgres server to listen for incoming connections
3. Specify where those connections can come from

## 1. A fixed local IP address for the Raspberry Pi

I won't cover this here but I allocated the Raspberry Pi a fixed IP address from the administration settings within my home router. Essentially it entails finding the right place in the router settings to link the specific MAC address of the Raspberry Pi to a predetermined IP address. I chose `192.168.1.100`, though this will depend on your local network setup.

## 2. Configure Postgres to listen for incoming connections

`postgresql.conf` is the file that controls how the PostgreSQL server itself runs.

In this case we need to make the postgres server listen for incoming connections. By default Postgres is configured to listen for 'local' connections only (i.e. from the Raspberry Pi itself). To listen for external connections, a common option is to configure it to listen to *all* incoming connections which can be done using '*'. A more restrictive version specifies which particular IP address to listen on and which port.

If you're not using '*' (i.e. all), the IP address here should match the IP address of the Raspberry Pi Postgres server itself (i.e. in this example, `192.168.1.100`).

## 3. Specify where connections can come from

`pg_hba.conf` controls where Postgres will accept connections from.

This will need to be configure to cover the IP addresses of the machines that you want to connect from.

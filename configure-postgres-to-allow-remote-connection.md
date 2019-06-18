# Configure PostgreSQL to allow remote connections

**NOTE**: This post is a personal update to [Neeraj Singh](https://blog.bigbinary.com/authors/neerajdotname)'s [post](https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html).

> By default PostgreSQL is configured to be bound to “localhost”. As we can see above port 5432 is bound to 127.0.0.1. It means any attempt to connect to the postgresql server from outside the machine will be refused.
>
> ```text
> $ netstat -nlt
> Proto Recv-Q Send-Q Local Address           Foreign Address         State
> # ...
> tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN
> ```

```text
$ nc -zv 10.142.0.3 5432¬
server.us-east1-d.c.general-development.internal [10.1.2.3] 5432 (postgresql) : Connection refused¬
```

To configuration files will need to be updated to allow remote connections:

1. The [`listen_addresses`](https://www.postgresql.org/docs/11/runtime-config-connection.html#GUC-LISTEN-ADDRESSES) configuration parameter in `postgresql.conf`, and

2. the `pg_hba.conf` file

## Finding the above configuration files

The location of both files can be queried from PostgreSQL itself (trick taken from [this Stackoverflow thread](https://stackoverflow.com/questions/3602450/where-are-my-postgres-conf-files)):

```bash
$ sudo -u postgres psql -c "SHOW hba_file" -c "SHOW config_file"
```

## `postgresql.conf`

The configuration file comes with helpful hints to get this working:

```text
listen_addresses = 'localhost'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
```

For a quick and dirty solution just change it to `listen_addresses = '*'`. Restart the server (see [here](https://medium.com/scientific-breakthrough-of-the-afternoon/properly-manage-postgresql-on-linux-distros-with-systemd-7fe66c48a1b0) how), and PostgreSQL is now listening from all IPs:
```text
$ sudo systemctl restart postgresql@11-main

$ netstat -tuplen
(Not all processes could be identified, non-owned process info
will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name
# ...
tcp6       0      0 :::5432                 :::*                    LISTEN      111        716997     -
```

The [`listen_addresses`](https://www.postgresql.org/docs/11/runtime-config-connection.html#GUC-LISTEN-ADDRESSES) documentation says that it "_Specifies the TCP/IP address(es) on which the server is to listen for connections from client applications._", but that's all. It specifies the sockets the packets are accepted from, but if the incoming connections are not authenticated (configured via `pg_hba.conf`), then the packets will be rejected (dropped?) regardless.

## `pg_hba.conf`

From [20.1. The pg_hba.conf File](https://www.postgresql.org/docs/11/auth-pg-hba-conf.html): "_Client authentication is controlled by a configuration file, which traditionally is named pg_hba.conf and is stored in the database cluster's data directory. (HBA stands for host-based authentication.) _"

This is a complex topic so reading the documentation is crucial, but this will suffice for development on trusted networks:

```text
host    all   all   0.0.0.0/0   trust
host    all   all   ::/0        trust
```

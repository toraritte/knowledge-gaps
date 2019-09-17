# Configure PostgreSQL to allow remote connections

**NOTE**: This post is a personal update to [Neeraj Singh](https://blog.bigbinary.com/authors/neerajdotname)'s [post](https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html).
[PostgreSQL must also be configured to allow remote connections][1], otherwise the connection request will fail, even if all firewalls rules are correct and PostgreSQL server is listening on the right port.

# Steps

## Outline

<sup>Couldn't create links, but this is a rather long answer so this may helps.</sup>

0. Tools to check ports during any step  
0.1 `nc` or `netcat`  
0.2 `nmap`  
0.3 `netstat`  
0.4 `lsof`  
1. IP addresses  
1.1 Your laptop's public IP address  
1.2 GCE instance's IP address  
2. Firewall rules  
2.1 Check existing  
2.2 Add new firewall rules  
3. Configure PostgreSQL to accept remote connections  
3.1 Finding the above configuration files  
3.2 `postgresql.conf`  
3.3 `pg_hba.conf`  

## 0. Tools to check ports during any step

### 0.1 `nc` or `netcat`

    $ nc -zv 4.3.2.1 5432

Where

     -v      Produce more verbose output.

     -z      Only scan for listening daemons, without sending any data to
             them.  Cannot be used together with -l.


Possible outcomes:

 - `Connection to 4.3.2.1port [tcp/postgresql] succeeded!`  
   > Yay.
 - `nc: connect to 4.3.2.1 port 8000 (tcp) failed: Connection refused`  
   > Port open by firewall, but service either not listening or refusing connection.
 - command just hangs  
   > Firewall is blocking.

### 0.2 `nmap`

    $ nmap 4.3.2.1

    Starting Nmap 7.70 ( https://nmap.org ) at 2019-09-09 18:28 PDT
    Nmap scan report for 1.2.3.4.bc.googleusercontent.com (4.3.2.1)
    Host is up (0.12s latency).
    Not shown: 993 filtered ports
    PORT     STATE  SERVICE
    22/tcp   open   ssh
    80/tcp   closed http
    443/tcp  closed https
    3389/tcp closed ms-wbt-server
    4000/tcp closed remoteanything
    5432/tcp open   postgresql      # firewall open, service up and listening
    8000/tcp closed http-alt        # firewall open, is service up or listening?

### 0.3 `netstat`

    $ netstat -tuplen

    (Not all processes could be identified, non-owned process info
     will not be shown, you would have to be root to see it all.)
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name    
    tcp        0      0 0.0.0.0:4000            0.0.0.0:*               LISTEN      1000       4223185    29432/beam.smp      
    tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      1000       4020942    15020/postgres      
    tcp        0      0 127.0.0.1:5433          0.0.0.0:*               LISTEN      1000       3246566    20553/postgres      
    tcp6       0      0 ::1:5432                :::*                    LISTEN      1000       4020941    15020/postgres      
    tcp6       0      0 ::1:5433                :::*                    LISTEN      1000       3246565    20553/postgres      
    udp        0      0 224.0.0.251:5353        0.0.0.0:*                           1000       4624644    6311/chrome --type= 
    udp        0      0 224.0.0.251:5353        0.0.0.0:*                           1000       4624643    6311/chrome --type= 
    udp        0      0 224.0.0.251:5353        0.0.0.0:*                           1000       4625649    6230/chrome         
    udp        0      0 0.0.0.0:68              0.0.0.0:*                           0          20911      -                   
    udp6       0      0 :::546                  :::*                                0          4621237    -                   

where

    -t | --tcp
   
    -u | --udp
   
    -p, --program
        Show the PID and name of the program to which each socket belongs.

    -l, --listening
        Show only listening sockets.  (These are omitted by default.)

    -e, --extend
        Display additional information.  Use  this  option  twice  for  maximum
        detail.

    --numeric, -n
        Show numerical addresses instead of trying to determine symbolic  host,
        port or user names.

When issued on the instance where PostgreSQL is running, and you don't see the lines below, it means that PostgreSQL is not configured for remote connections:

    tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      1001       238400     30826/postgres
    tcp6       0      0 :::5432                 :::*                    LISTEN      1001       238401     30826/postgres  

### 0.4 `lsof`

To check on instance whether service is running at all.

    $ sudo lsof -i -P -n | grep LISTEN

     systemd-r   457 systemd-resolve   13u  IPv4  14870      0t0  TCP 127.0.0.53:53 (LISTEN)
     sshd        733            root    3u  IPv4  19233      0t0  TCP *:22 (LISTEN)
     sshd        733            root    4u  IPv6  19244      0t0  TCP *:22 (LISTEN)
     postgres   2733        postgres    3u  IPv4  23655      0t0  TCP 127.0.0.1:5432 (LISTEN)
     python3   26083        a_user      4u  IPv4 392307      0t0  TCP *:8000 (LISTEN)

## 1. IP addresses

To connect from your laptop, you will need the public IP address of your laptop, and that of the Google Compute Engine (GCE) instance.

### 1.1 Your laptop's public IP address

(From [this article][2].)

    $ dig +short myip.opendns.com @resolver1.opendns.com
    4.3.2.1

### 1.2 GCE instance's IP address

    $ gcloud compute instances list

    NAME         ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
    access-news  us-east1-d  n1-standard-2               10.142.0.5   34.73.156.19    RUNNING
    lynx-dev     us-east1-d  n1-standard-1               10.142.0.2   35.231.66.229   RUNNING
    tr2          us-east1-d  n1-standard-1               10.142.0.3   35.196.195.199  RUNNING

If you also need the [*network-tags*][3] of the instances:

    $ gcloud compute instances list  --format='table(name,status,tags.list())'
    NAME         STATUS   TAGS
    access-news  RUNNING  fingerprint=mdTPd8rXoQM=,items=[u'access-news', u'http-server', u'https-server']
    lynx-dev     RUNNING  fingerprint=CpSmrCTD0LE=,items=[u'http-server', u'https-server', u'lynx-dev']
    tr2          RUNNING  fingerprint=84JxACwWD7U=,items=[u'http-server', u'https-server', u'tr2']

## 2. Firewall rules

Dealing only with GCE firewall rules below, but make sure that [`iptables`][4] doesn't inadvertently blocks traffic. 

See also 

 - ["Firewall rules overview" (official docs)][5]
 - [GCE firewall rules vs. `iptables`][6]
 - [Summary of GCE firewall terms][7]
 - [Behaviour of GCE firewall rules on instances (external vs internal IP addresses)][8]

### 2.1 Check existing

    $ gcloud compute firewall-rules list

    NAME                      NETWORK  DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
    default-allow-http        default  INGRESS    1000      tcp:80                              False
    default-allow-https       default  INGRESS    1000      tcp:443                             False
    default-allow-icmp        default  INGRESS    65534     icmp                                False
    default-allow-internal    default  INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
    default-allow-rdp         default  INGRESS    65534     tcp:3389                            False
    default-allow-ssh         default  INGRESS    65534     tcp:22                              False
    pg-from-tag1-to-tag2      default  INGRESS    1000      tcp:5432                            False
    
    To show all fields of the firewall, please show in JSON format: --format=json
    To show all fields in table format, please see the examples in --help.

A more comprehensive list that includes network-tags as well (from `gcloud compute firewall-rules list --help`):

    $ gcloud compute firewall-rules list --format="table(     \
          name,                                               \
          network,                                            \
          direction,                                          \
          priority,                                           \
          sourceRanges.list():label=SRC_RANGES,               \
          destinationRanges.list():label=DEST_RANGES,         \
          allowed[].map().firewall_rule().list():label=ALLOW, \
          denied[].map().firewall_rule().list():label=DENY,   \
          sourceTags.list():label=SRC_TAGS,                   \
          sourceServiceAccounts.list():label=SRC_SVC_ACCT,    \
          targetTags.list():label=TARGET_TAGS,                \
          targetServiceAccounts.list():label=TARGET_SVC_ACCT, \
          disabled                                            \
      )"

    NAME                      NETWORK  DIRECTION  PRIORITY  SRC_RANGES    DEST_RANGES  ALLOW                         DENY  SRC_TAGS  SRC_SVC_ACCT  TARGET_TAGS   TARGET_SVC_ACCT  DISABLED
    default-allow-http        default  INGRESS    1000      0.0.0.0/0                  tcp:80                                                      http-server                    False
    default-allow-https       default  INGRESS    1000      0.0.0.0/0                  tcp:443                                                     https-server                   False
    default-allow-icmp        default  INGRESS    65534     0.0.0.0/0                  icmp                                                                                       False
    default-allow-internal    default  INGRESS    65534     10.128.0.0/9               tcp:0-65535,udp:0-65535,icmp                                                               False
    default-allow-rdp         default  INGRESS    65534     0.0.0.0/0                  tcp:3389                                                                                   False
    default-allow-ssh         default  INGRESS    65534     0.0.0.0/0                  tcp:22                                                                                     False
    pg-from-tag1-to-tag2      default  INGRESS    1000      4.3.2.1                    tcp:5432                            tag1                    tag2                           False

### 2.2 Add new firewall rules

To open the default PostgreSQL port (5432) from every source to every instance:

    $ gcloud compute firewall-rules create \
        postgres-all                       \
        --network default                  \
        --priority 1000                    \
        --direction ingress                \
        --action allow                     \
        --rules tcp:5432                   \

To restrict it between your computer (source: `YOUR_IP`) and the GCE instance (destination: `INSTANCE_IP`):

    $ gcloud compute firewall-rules create \
        postgres-from-you-to-instance      \
        --network default                  \
        --priority 1000                    \
        --direction ingress                \
        --action allow                     \
        --rules tcp:5432                   \
        --destination-ranges INSTANCES_IP  \
        --source-ranges YOUR_IP            \

Instead of `--source-ranges` and `--destination-ranges` one could use source and target network tags or service accounts as well. See the ["Source or destination" section in the firewall docs][9].

## 3. Configure PostgreSQL to accept remote connections

<sup>This is an update to [Neeraj Singh](https://blog.bigbinary.com/authors/neerajdotname)'s [post](https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html).</sup>

By default PostgreSQL is configured to be bound to “localhost”, therefore the below configuration files will need to be updated:

1. `postgresql.conf`, and

2. `pg_hba.conf`

### 3.1 Finding the above configuration files

The location of both files can be queried from PostgreSQL itself (trick taken from [this Stackoverflow thread](https://stackoverflow.com/questions/3602450/where-are-my-postgres-conf-files)):

```bash
$ sudo -u postgres psql -c "SHOW hba_file" -c "SHOW config_file"
```

### 3.2 `postgresql.conf`

The configuration file comes with helpful hints to get this working:

```text
listen_addresses = 'localhost'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
```

For a quick and dirty solution just change it to

    listen_addresses = '*'

Restart the server (see [here](https://medium.com/scientific-breakthrough-of-the-afternoon/properly-manage-postgresql-on-linux-distros-with-systemd-7fe66c48a1b0) how). Once PostgreSQL is restarted, it will start listening on all IP addresses (see `netstat -tuplen`).

To restart PostgreSQL:

```text
$ sudo systemctl restart postgresql@11-main

# or

$ pg_ctl restart
```

The [`listen_addresses`](https://www.postgresql.org/docs/11/runtime-config-connection.html#GUC-LISTEN-ADDRESSES) documentation says that it "_Specifies the TCP/IP address(es) on which the server is to listen for connections from client applications._", but that's all. It specifies the sockets the packets are accepted from, but if the incoming connections are not authenticated (configured via `pg_hba.conf`), then the packets will be rejected (dropped?) regardless.

### 3.3 `pg_hba.conf`

From [20.1. The pg_hba.conf File](https://www.postgresql.org/docs/11/auth-pg-hba-conf.html): "_Client authentication is controlled by a configuration file, which traditionally is named pg_hba.conf and is stored in the database cluster's data directory. (HBA stands for host-based authentication.) _"

This is a complex topic so reading the documentation is crucial, but this will suffice for development on trusted networks:

```text
host    all   all   0.0.0.0/0   trust
host    all   all   ::/0        trust
```

Another restart is required at this point.

  [1]: https://github.com/toraritte/knowledge-gaps/blob/730bf3eb21cb6836703f0ab0f9b0a54e35b7fd63/configure-postgres-to-allow-remote-connection.md
  [2]: https://www.cyberciti.biz/faq/how-to-find-my-public-ip-address-from-command-line-on-a-linux/
  [3]: https://cloud.google.com/vpc/docs/add-remove-network-tags
  [4]: https://linux.die.net/man/8/iptables
  [5]: https://cloud.google.com/vpc/docs/firewalls
  [6]: https://github.com/toraritte/knowledge-gaps/blob/35d1600b93b709710ea9e8046bba288d55dbe034/google-compute-engine_and_linux_administration/gce-firewall-vs-iptables.md
  [7]: https://github.com/toraritte/knowledge-gaps/blob/35d1600b93b709710ea9e8046bba288d55dbe034/google-compute-engine_and_linux_administration/gce-firewall-ingress-egress-source-destination-target.md
  [8]: https://github.com/toraritte/knowledge-gaps/blob/35d1600b93b709710ea9e8046bba288d55dbe034/google-compute-engine_and_linux_administration/firewall-and-networks.md
  [9]: https://cloud.google.com/vpc/docs/firewalls#sources_or_destinations_for_the_rule


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

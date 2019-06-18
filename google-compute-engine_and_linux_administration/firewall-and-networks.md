# Documenting `gcloud firewall-rules`

```text
admin:~$ gcloud compute instances list
NAME      ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
pb-1      us-east1-d  n1-standard-1               10.1.2.3     35.1.2.3        RUNNING
pb-2      us-east1-d  n1-standard-1               10.4.5.6     35.4.5.6        RUNNING

admin:~$ gcloud compute firewall-rules list
NAME                         NETWORK  DIRECTION  PRIORITY  ALLOW                                                  DENY  DISABLED
default-allow-http           default  INGRESS    1000      tcp:80                                                       False
default-allow-https          default  INGRESS    1000      tcp:443                                                      False
default-allow-icmp           default  INGRESS    65534     icmp                                                         False
default-allow-internal       default  INGRESS    65534     tcp:0-65535,udp:0-65535,icmp                                 False
default-allow-rdp            default  INGRESS    65534     tcp:3389                                                     False
default-allow-ssh            default  INGRESS    65534     tcp:22                                                       False
# ...
# The following lines have been added manually
# ...
django-dev                   default  INGRESS    1000      tcp:8000                                                     False
django-dev--egress           default  EGRESS     1000      tcp:8000                                                     False
```

Checking `pb-2` ports on `pb-1`:

```text
pb-1:~$ nmap 35.4.5.6
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-18 17:09 UTC
Nmap scan report for 6.5.4.35.bc.googleusercontent.com (35.4.5.6)
Host is up (0.00100s latency).
Not shown: 995 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   closed http
443/tcp  closed https
3389/tcp closed ms-wbt-server
8000/tcp closed http-alt

Nmap done: 1 IP address (1 host up) scanned in 4.09 seconds

pb-1:~$ nmap 10.4.5.6
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-18 17:10 UTC
Nmap scan report for pb-2.us-east1-d.c.general-development.internal (10.4.5.6)
Host is up (0.00015s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
5060/tcp open  sip
5080/tcp open  onscreen
7443/tcp open  oracleas-https
8021/tcp open  ftp-proxy
8081/tcp open  blackice-icecap
8082/tcp open  blackice-alerts

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

Checking `pb-1` ports on `pb-2`:

```text
pb-2:~$ nmap 35.1.2.3

Starting Nmap 7.40 ( https://nmap.org ) at 2019-06-18 17:08 UTC
Nmap scan report for 3.2.1.35.bc.googleusercontent.com (35.1.2.3)
Host is up (0.0036s latency).
Not shown: 995 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   closed http
443/tcp  closed https
3389/tcp closed ms-wbt-server
8000/tcp open   http-alt

Nmap done: 1 IP address (1 host up) scanned in 4.88 seconds

pb-2:~$ nmap 10.1.2.3

 Starting Nmap 7.40 ( https://nmap.org ) at 2019-06-18 17:09 UTC
 Nmap scan report for pb-1.us-east1-d.c.general-development.internal (10.1.2.3)
 Host is up (0.00037s latency).
 Not shown: 998 closed ports
 PORT     STATE SERVICE
 22/tcp   open  ssh
 8000/tcp open  http-alt

 Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds

Adding firewall rule for PostgreSQL on `admin`:

```text
admin:~: gcloud compute firewall-rules create postgres \
         --network default   \
         --priority 1000     \
         --direction ingress \
         --action allow      \
         --rules tcp:5432
Creating firewall...⠧Created [https://www.googleapis.com/compute/v1/projects/general-development-240818/global/firewalls/postgres].
Creating firewall...done.
NAME      NETWORK  DIRECTION  PRIORITY  ALLOW     DENY  DISABLED
postgres  default  INGRESS    1000      tcp:5432        False
```

Checking `pb-2` ports again on `pb-1`:

```text
pb-1:~$ nmap 35.4.5.6
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-18 17:46 UTC
Nmap scan report for 6.5.4.35.bc.googleusercontent.com (35.4.5.6)
Host is up (0.0011s latency).
Not shown: 994 filtered ports
PORT     STATE  SERVICE
# ... omitting matching lines ...
5432/tcp closed postgresql
8000/tcp closed http-alt

Nmap done: 1 IP address (1 host up) scanned in 4.57 seconds

pb-1:~$ nmap 10.4.5.6
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-18 17:46 UTC
Nmap scan report for pb-2.us-east1-d.c.general-development.internal (10.4.5.6)
Host is up (0.00016s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
# ... omitting matching lines ...

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

Checking `pb-1` ports again on `pb-2`:

```text
pb-2:~$ nmap 35.1.2.3

Starting Nmap 7.40 ( https://nmap.org ) at 2019-06-18 17:44 UTC
Nmap scan report for 3.2.1.35.bc.googleusercontent.com (35.1.2.3)
Host is up (0.0010s latency).
Not shown: 994 filtered ports
PORT     STATE  SERVICE
# ... omitting matching lines ...
5432/tcp closed postgresql
8000/tcp open   http-alt

Nmap done: 1 IP address (1 host up) scanned in 4.28 seconds

pb-2:~$ nmap 10.1.2.3

Starting Nmap 7.40 ( https://nmap.org ) at 2019-06-18 17:45 UTC
Nmap scan report for pb-1.us-east1-d.c.general-development.internal (10.1.2.3)
Host is up (0.00018s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
# ... omitting matching lines ...

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

The result of the firewall rule is that all instances start listening(?) on the given port(s).

QUESTION: What is happening exactly? Some network settings must change, because even though 5432 shows as closed, but it wouldn't even show up before.

PostgreSQL is running on `pb-1` but only listening for `localhost` connections (see below).

```text
pb-1:~$ netstat -tuplen
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
 Active Internet connections (only servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name
>  tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      1002       392307     -
 tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      102        14870      -
 tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          19233      -
> tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      111        23655      -
 tcp6       0      0 :::22                   :::*                    LISTEN      0          19244      -
 udp        0      0 127.0.0.53:53           0.0.0.0:*                           102        14869      -
 udp        0      0 10.1.2.3:68             0.0.0.0:*                           101        678330     -
 udp        0      0 127.0.0.1:323           0.0.0.0:*                           0          16766      -
 udp6       0      0 ::1:323                 :::*                                0          16767      -

pb-1:~$ sudo lsof -i -P -n | grep LISTEN
 systemd-r   457 systemd-resolve   13u  IPv4  14870      0t0  TCP 127.0.0.53:53 (LISTEN)
 sshd        733            root    3u  IPv4  19233      0t0  TCP *:22 (LISTEN)
 sshd        733            root    4u  IPv6  19244      0t0  TCP *:22 (LISTEN)
>  postgres   2733        postgres    3u  IPv4  23655      0t0  TCP 127.0.0.1:5432 (LISTEN)
>  python3   26083        a_user      4u  IPv4 392307      0t0  TCP *:8000 (LISTEN)
```

Configuring PostgreSQL according to [this post](https://blog.bigbinary.com/2016/01/23/configure-postgresql-to-allow-remote-connection.html) so that it listens from connections other than `localhost` as well.



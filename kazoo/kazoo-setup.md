Comments on the [CentOS setup](https://docs.2600hz.com/sysadmin/doc/install/install_via_centos7/)
==============================

### "Setup the server"

> # Setup NTPd

Why?  systemd has  `timesyncd`  that acts  something
like it, and CentOS has systemd. At least, `ntpd` is
started via `systemctl`.

### "Setting up RabbitMQ"

> # Install the Kazoo-wrapped RabbitMQ
> yum install -y kazoo-rabbitmq

`kazoo-rabbitmq`  is  a   2600hz  RPM  package  that
has  a `kazoo.rabbitmq`  shell  script wrapping  the
`rabbitmq-server`. I think the shell script is
https://github.com/2600hz-archive/kazoo-configs/blob/master/system/sbin/kazoo-rabbitmq
because of this forum thread:
https://groups.google.com/d/msg/2600Hz-dev/5ce6_LVDwiI/jEpkRV0pAQAJ
It mentions that in order to start `rabbitmq-server` in a cluster, delete a line starting with `rm -rf` in `kazoo-rabbitmq`, and the abo

From [this Stackoverflow thread](https://serverfault.com/questions/634896/google-computer-engine-firewall-and-iptables) (edited):

> Q: I  see  that  Google  Compute Engine  comes  with  a
>    firewall setting we can use for each instance. So in
>    this  case, does  it mean  I dont  need to  open and
>    close  ports in  iptable as  well  when I  set up  a
>    CentOs with Nginx web server on GCE?
>
> A: GCE  firewall works  at project  level and  IPtables
>    works  at  OS  level.  For an  instance  to  see  an
>    incoming connection both firewalls must allow it.
>
>    GCE  firewall blocks  all  incoming  traffic to  the
>    instances by default unless  explicitly allowed by a
>    firewall rule. Rules allow  incoming traffic from an
>    IP range,  a list of  protocols (ICMP, TCP  and UDP)
>    and a list  of ports, and they can  be restricted to
>    some instances by using tags.
>
> Q: If  Google  compute  engine  does the  same  job  as
>    iptables,  then  do  I  need to  setup  any  special
>    firewall  rules for  Blocking  Null packets,  Reject
>    Syn-Flood Attack, Reject XMAS  Packets, etc.. to GCE
>    Firewall or is that not needed?
>
> A: GCE firewall is  not as flexible as  IPtables and it
>    is  not suitable  for  this.  Instead, GCE  firewall
>    focus  on  90%  use  cases  a  firewall  has:  Avoid
>    unauthorized incoming connections to your instances.

From [Firewall Rules Overview](https://cloud.google.com/vpc/docs/firewalls) in the documentation:

> Every  VPC   network  functions  as   a  distributed
> firewall. While  firewall rules  are defined  at the
> network level, connections are  allowed or denied on
> a  per-instance  basis. You  can  think  of the  GCP
> firewall  rules as  existing not  only between  your
> instances and other networks, but between individual
> instances within the same network.

# Summary of GCE firewall terms ingress, egress, source, destination, and targets

Used Google Compute Engine documentations:
+ [Firewall Rules](https://cloud.google.com/vpc/docs/firewalls)
+ [Using Firewall Rules](https://cloud.google.com/vpc/docs/using-firewalls)

## 0. [Components of firewall rules](https://cloud.google.com/vpc/docs/firewalls#firewall_rule_components)

Each firewall rule consists of the following configuration components:

### 0.0 [Priority](https://cloud.google.com/vpc/docs/firewalls#priority_order_for_firewall_rules)

From 0 (highest priority) to 65535 (lowest priority)

### 0.1 [Direction of traffic](https://cloud.google.com/vpc/docs/firewalls#direction_of_the_rule)

  + **INGRESS rules** apply to incoming connections
  ```text
    FROM specified SOURCES
    TO   targets
  ```

  + **EGRESS rules** apply to traffic going
  ```text
     TO   specified DESTINATIONS
     FROM targets
  ```

> Traffic from VM1 to VM2 can be controlled using either of these firewall rules:
>
> + An ingress rule with a target of VM2 and a source of VM1.
>
> + An egress rule with a target of VM1 and a destination of VM2.

See "0.3 Target" below for the definition of what a target is.

### 0.2 [Action](https://cloud.google.com/vpc/docs/firewalls#action_of_the_rule)

  + **allow**: permit traffic

  + **deny**:  block traffic

### 0.3 [Target](https://cloud.google.com/vpc/docs/firewalls#rule_assignment)

+ All instances in the VPC network
+ Instances by [service account](https://cloud.google.com/iam/docs/service-accounts)
+ Instances by [network tag](https://cloud.google.com/vpc/docs/add-remove-network-tags)

Again,

> `ingress`  rules   apply  to   incoming  connections
> from  specified  *sources*  to  GCP  *targets*,  and
> `egress`  rules apply  to traffic  going to  specified
> *destinations* from *targets*.

### 0.4 Source (for INGRESS rules) or a Destination (for EGRESS rules)

For   INGRESS   (inbound)  rules,   the   **target**
parameter  specifies the  destination instances  for
traffic; you  cannot use the  destination parameter.
You  specify   the  source   by  using   the  **source**
parameter (which  is  only  applicable  to  INGRESS
rules):

+ all IP addresses (`0.0.0.0/0`)
+ IP address ranges
+ [network tag](https://cloud.google.com/vpc/docs/add-remove-network-tags)s
+ [service account](https://cloud.google.com/iam/docs/service-accounts)s

For  EGRESS (outbound)  rules, the  **target** parameter
specifies  the  source  instances for  traffic;  you
cannot  use the  source parameter.  You specify  the
destination  by  using  the  **destination**  parameter
(which is only applicable to EGRESS rules):

+ all IP addresses (`0.0.0.0/0`)
+ IP address ranges

See [Source or destination](https://cloud.google.com/vpc/docs/firewalls#sources_or_destinations_for_the_rule) in the documentation.

### 0.5 Protocol (such as TCP, UDP, or ICMP) and port

### 0.6 The [enforcement](https://cloud.google.com/vpc/docs/firewalls#enforcement) status of the firewall rule

+ enabled (`--no-disabled`)
+ disabled (`--disabled`)

## 1. [`gcloud` syntax](https://cloud.google.com/vpc/docs/using-firewalls#creating_firewall_rules)

```text
gcloud compute firewall-rules create NAME                                   \
    [--network NETWORK; default="default"]                                  \
    [--priority PRIORITY;default=1000]                                      \
    [--direction (ingress|egress|in|out); default="ingress"]                \
    [--action (deny | allow )]                                              \
    [--target-tags TAG,TAG,...]                                             \
    [--target-service-accounts=IAM Service Account,IAM Service Account,...] \
    [--source-ranges CIDR-RANGE,CIDR-RANGE...]                              \
    [--source-tags TAG,TAG,...]                                             \
    [--source-service-accounts=IAM Service Account,IAM Service Account,...] \
    [--destination-ranges CIDR-RANGE,CIDR-RANGE...]                         \
    [--rules (PROTOCOL[:PORT[-PORT]],[PROTOCOL[:PORT[-PORT]],...]] | all )  \
    [--disabled | --no-disabled]                                            \
    [--enable-logging | --no-enable-logging]
```

## 2. List instances with network tags

`gcloud compute instances add-tags --help` is pretty helpful:

``` text
DESCRIPTION
    gcloud compute instances add-tags is used to add tags to Google Compute
    Engine virtual machine instances. For example, running:

        $ gcloud compute instances add-tags example-instance \
            --tags tag-1,tag-2

    will add tags tag-1 and tag-2 to 'example-instance'.

    Tags can be used to identify the instances when adding network firewall
    rules. Tags can also be used to get firewall rules that already exist to be
    applied to the instance. See gcloud compute firewall-rules create(1) for
    more details.

    To list instances with their respective status and tags, run:

        $ gcloud compute instances list \
            --format='table(name,status,tags.list())'

    To list instances tagged with a specific tag, tag1, run:

        $ gcloud compute instances list --filter='tags:tag1'
```

For example:

```text
$ gcloud compute instances list  --format='table(name,status,tags.list())'
NAME      STATUS   TAGS
pb-1      RUNNING  fingerprint=CpSmrCTD0LE=,items=[u'http-server', u'https-server', u'pb-1']
pb-2      RUNNING  fingerprint=84JxACwWD7U=,items=[u'http-server', u'https-server', u'pb-2']
```

The tags `http-server` and `https-server` are created by default and added to instances when they are created. For example, listing firewall rules with tags:

```text
$ gcloud compute firewall-rules list --format="table(       \
        name,                                               \
        network,                                            \
        direction,                                          \
        priority,                                           \
        sourceRanges.list():label=SRC_RANGES,               \
        destinationRanges.list():label=DEST_RANGES,         \
        allowed[].map().firewall_rule().list():label=ALLOW, \
        denied[].map().firewall_rule().list():label=DENY,   \
        sourceTags.list():label=SRC_TAGS,                   \
        targetTags.list():label=TARGET_TAGS,                \
        disabled                                            \
        )"
NAME                    NETWORK  DIRECTION  PRIORITY  SRC_RANGES    DEST_RANGES  ALLOW                         DENY  SRC_TAGS  TARGET_TAGS   DISABLED
default-allow-http      default  INGRESS    1000      0.0.0.0/0                  tcp:80                                        http-server   False
default-allow-https     default  INGRESS    1000      0.0.0.0/0                  tcp:443                                       https-server  False
```

## 3. Examples

All examples will allow opening the TCP port 5432, which is the port PostgreSQL uses to listen to external connections, and all listings use the above `gcloud` command listing firewall rules.

### 3.0 Rule that allows access to TCP 5432 from all IP address on all GCE instances

```text
$ gcloud compute firewall-rules create postgres --network default --priority 1000 --direction ingress --action allow  --rules tcp:5432

# result
NAME                    NETWORK  DIRECTION  PRIORITY  SRC_RANGES    DEST_RANGES  ALLOW                         DENY  SRC_TAGS  TARGET_TAGS   DISABLED
postgres                default  INGRESS    1000      0.0.0.0/0                  tcp:5432                                                    False
```

### 3.1 Rule that allows access to TCP 5432 from instances with `pb-1` network tag on all GCE instances

```text
$ gcloud compute firewall-rules create postgres --network default --priority 1000 --direction ingress --action allow  --rules tcp:5432 --source-tags pb-1

# result
NAME                    NETWORK  DIRECTION  PRIORITY  SRC_RANGES    DEST_RANGES  ALLOW                         DENY  SRC_TAGS  TARGET_TAGS   DISABLED
postgres                default  INGRESS    1000                                 tcp:5432                            pb-1                    False
```

### 3.2 Rule that allows access to TCP 5432 from instances with `pb-1` network tag on instances with `pb-2` network tag

```text
gcloud compute firewall-rules create postgres --network default --priority 1000 --direction ingress --action allow  --rules tcp:5432 --source-tags pb-1 --target-tags pb-2

# result
NAME                    NETWORK  DIRECTION  PRIORITY  SRC_RANGES    DEST_RANGES  ALLOW                         DENY  SRC_TAGS  TARGET_TAGS   DISABLED
postgres                default  INGRESS    1000                                 tcp:5432                            pb-1      pb-2          False
```

### 3.3 Same as 3.2 but allow access from 66.1.2.3 as well

```text
gcloud compute firewall-rules create postgres --network default --priority 1000 --direction ingress --action allow  --rules tcp:5432 --source-tags pb-1 --target-tags pb-2 --source-ranges 66.1.2.3

# result
NAME                    NETWORK  DIRECTION  PRIORITY  SRC_RANGES    DEST_RANGES  ALLOW                         DENY  SRC_TAGS  TARGET_TAGS   DISABLED
postgres                default  INGRESS    1000      66.1.2.3                   tcp:5432                            pb-1      pb-2          False
```

Worth repeating from [Direction of traffic](https://cloud.google.com/vpc/docs/firewalls#direction_of_the_rule):
> Traffic from VM1 to VM2 can be controlled using either of these firewall rules:
>
> + An ingress rule with a target of VM2 and a source of VM1.
>
> + An egress rule with a target of VM1 and a destination of VM2.

**Note to self**: find the external IP on the console:
+ https://www.cyberciti.biz/faq/how-to-find-my-public-ip-address-from-command-line-on-a-linux/
+ https://unix.stackexchange.com/questions/335371/how-does-dig-find-my-wan-ip-adress-what-is-myip-opendns-com-doing

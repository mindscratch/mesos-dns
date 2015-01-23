---
title: Service Naming
---

# Service Naming

Mesos-DNS defines a DNS domain for Mesos tasks (default `.mesos`, see [instructions on configruation](configuration-parameters.html)). Running tasks can be discovered by looking up A and, optionally, SRV records within the Mesos domain. 

## A Records

An A record associates a hostname to an IP address. For task `task` launched by framework `framework`, Mesos-DNS generates an A record for hostname `task.framework.domain` that provides the IP address of the specific slave running the task. For example, other Mesos tasks can discover the IP address for service `search` launched by the `marathon` framework with a lookup for `search.marathon.mesos`:

``` console
$ dig search.marathon.mesos

; <<>> DiG 9.8.4-rpz2+rl005.12-P1 <<>> search.marathon.mesos
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24471
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;search.marathon.mesos.			IN	A

;; ANSWER SECTION:
search.marathon.mesos.		60	IN	A	10.9.87.94
```

*note*: many frameworks actually have longer, less friendly names (example, marathon is actually marathon-<version>, such as marathon-0.7.6). An [https://github.com/mesosphere/mesos-dns/issues/38] is open to improve this situation.
 
## SRV Records

An SRV record associates a service name to a hostname and an IP port.  For task `task` launched by framework `framework`, Mesos-DNS generates an SRV record for service name `_task._protocol.framework.domain`, where `protocol` is `udp` or `tcp`. For example, other Mesos tasks can discover service `search` launched by the `marathon` framework with a lookup for lookup `_search._tcp.marathon.mesos`:

``` console
$ dig _search._tcp.marathon.mesos SRV

; <<>> DiG 9.8.4-rpz2+rl005.12-P1 <<>> _search._tcp.marathon.mesos SRV
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33793
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;_search._tcp.marathon.mesos.	IN SRV

;; ANSWER SECTION:
_search._tcp.marathon.mesos.	60 IN SRV 0 0 31302 10.254.132.41.
``` 

SRV records are generated only for tasks that have been allocated a specific port through Mesos. 

## Notes

If a framework launches multiple tasks with the same name, the DNS lookup will return multiple records, one per task. Mesos-DNS randomly shuffles the order of records to provide rudimentary load balancing between these tasks. 

Mesos-DNS does not support other types of DNS records at this point. DNS requests for records of type`ANY`, `A`, or `SRV` will return any A or SRV records found. DNS requests for records of other types in the Mesos domain will return `NXDOMAIN`.




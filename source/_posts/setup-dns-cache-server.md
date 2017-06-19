---
title: 'Setup A DNSCrypt-Protected DNS Cache Server'
tags:
  - network
  - dns
id: 12
categories:
  - Network
date: 2015-12-24 21:22:00
---

Install [dnscrypt](https://dnscrypt.org/):

```bash
$ sudo add-apt-repository ppa:xuzhen666/dnscrypt
$ sudo apt-get update
$ sudo apt-get install dnscrypt-proxy
```

Start dnscrypt-proxy instances:

```bash
$ sudo /usr/sbin/dnscrypt-proxy -R cisco-ipv6 -a 127.0.0.1:10898 -d
$ sudo /usr/sbin/dnscrypt-proxy -R cloudns-syd -a 127.0.0.1:10899 -d
```

``-R resolver`` specifies the resolver proxy uses. For the list of available resolvers, please refer to ``/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv`` or online at [here](https://github.com/jedisct1/dnscrypt-proxy/blob/master/dnscrypt-resolvers.csv).
<!-- more -->

Install [unbound](https://www.unbound.net/):

```bash
$ sudo apt-get install unbound
```

Download the listing of primary root DNS servers:

```bash
$ sudo wget ftp://ftp.internic.net/domain/named.cache -O /etc/unbound/root.hints
```

Edit ``/etc/unbound/unbound.conf``:

```bash
# Unbound configuration file for Debian.
#
# See the unbound.conf(5) man page.
#
# See /usr/share/doc/unbound/examples/unbound.conf for a commented
# reference config file.
#
# The following line includes additional configuration files from the
# /etc/unbound/unbound.conf.d directory.
include: "/etc/unbound/unbound.conf.d/*.conf"

server:
    # interfaces for listening (both ipv4 and ipv6)
    interface: 0.0.0.0
    interface: ::0

    # enable for both ipv4 and ipv6 (udp or tcp)
    do-ip4: yes
    do-ip6: yes
    do-udp: yes
    do-tcp: yes

    # use previously downloaded root server list
    root-hints: "/etc/unbound/root.hints"

    # reject id.server or version.server queries
    hide-identity: yes
    hide-version: yes

    # enhance security (DNSSEC)
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes

    # set time to live for cache entries
    cache-min-ttl: 3600
    cache-max-ttl: 86400

    # performance optimizations
    prefetch: yes
    num-threads: 4
    msg-cache-slabs: 8
    rrset-cache-slabs: 8
    infra-cache-slabs: 8
    key-cache-slabs: 8
    rrset-cache-size: 256m
    msg-cache-size: 128m
    so-rcvbuf: 1m

    # allow all IPs
    access-control: 0.0.0.0/0 allow

    # allow queries on 127.0.0.1
    do-not-query-localhost: no

forward-zone:
    name: "."
    # forward queries to dnscrypt-proxy instances
    forward-addr: 127.0.0.1@10898
    forward-addr: 127.0.0.1@10899
```

Start unbound:

```bash
$ unbound-checkconf /etc/bound/bound.conf     # check configuration
$ sudo unbound
```

To test whether the DNS server is working correctly, run following command:

```bash
$ dig @127.0.0.1 +dnssec facebook.com

; <<>> DiG 9.9.5-3ubuntu0.6-Ubuntu <<>> @127.0.0.1 +dnssec facebook.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40322
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 4096
;; QUESTION SECTION:
;facebook.com.          IN  A

;; ANSWER SECTION:
facebook.com.       3600    IN  A   66.220.158.68

;; AUTHORITY SECTION:
facebook.com.       48644   IN  NS  a.ns.facebook.com.
facebook.com.       48644   IN  NS  b.ns.facebook.com.

;; ADDITIONAL SECTION:
a.ns.facebook.com.  140790  IN  AAAA    2a03:2880:fffe:c:face:b00c:0:35
a.ns.facebook.com.  98877   IN  A   69.171.239.12
b.ns.facebook.com.  158360  IN  AAAA    2a03:2880:ffff:c:face:b00c:0:35
b.ns.facebook.com.  98877   IN  A   69.171.255.12

;; Query time: 819 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Thu Dec 24 23:33:09 CST 2015
;; MSG SIZE  rcvd: 180

```

To autostart unbound and dnscrypt-proxy, modify ``/etc/rc.local`` and add following lines to the top of the file:

```bash
/usr/sbin/dnscrypt-proxy -R cisco-ipv6 -a 127.0.0.1:10898 -d
/usr/sbin/dnscrypt-proxy -R cloudns-syd -a 127.0.0.1:10899 -d
/usr/sbin/unbound
```
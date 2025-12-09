---
tags:
  - dns
---
#### AXFR

> Code Snippet
> ```
> ; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> axfr @10.0.2.7 olympus.local
; (1 server found)
;; global options: +cmd
olympus.local.          10800   IN      SOA     olympus.local. root.olympus.local. 0 86400 3600 604800 10800
olympus.local.          10800   IN      NS      delphos.olympus.local.
olympus.local.          10800   IN      NS      ns1.olympus.local.
olympus.local.          10800   IN      MX      10 mail.olympus.local.
debian12.olympus.local. 10800   IN      A       10.0.2.10
delphos.olympus.local.  10800   IN      AAAA    fe80::20c:29ff:fe78:4cb1
delphos.olympus.local.  10800   IN      A       10.0.2.7
dns.olympus.local.      10800   IN      CNAME   delphos.olympus.local.
ns1.olympus.local.      10800   IN      A       10.0.2.9
prometheus.olympus.local. 10800 IN      A       10.0.2.4
olympus.local.          10800   IN      SOA     olympus.local. root.olympus.local. 0 86400 3600 604800 10800
;; SERVER: 10.0.2.7#53(10.0.2.7)
;; XFR size: 13 records (messages 1, bytes 350)```

> [!NOTE]- DNS records dump
> 
> ```
> olympus.local.          10800   IN      MX      10 mail.olympus.local.
> debian12.olympus.local. 10800   IN      A       10.0.2.10
> delphos.olympus.local.  10800   IN      AAAA    fe80::20c:29ff:fe78:4cb1
> delphos.olympus.local.  10800   IN      A       10.0.2.7
> dns.olympus.local.      10800   IN      CNAME   delphos.olympus.local.
> ns1.olympus.local.      10800   IN      A       10.0.2.9
> prometheus.olympus.local. 10800 IN      A       10.0.2.4
> olympus.local.          10800   IN      SOA     olympus.local. root.olympus.local. 0 86400 3600 
> ```
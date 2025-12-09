```\[root@delphos named]# dig @10.0.2.7 DNSKEY olympus.local. +multiline

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> @10.0.2.7 DNSKEY olympus.local. +multiline
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62024
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;olympus.local.         IN DNSKEY

;; ANSWER SECTION:
olympus.local.          10800 IN DNSKEY 257 3 5 (
                                AwEAAcY+Mbt+0cKzEaRpowIPpbi
                                4c4llHZUw/wK13tuyO3j+D4VDWG
                                ZkwH6tk7EMUR4B1BcBgYk6NuDun
                                Emz/vx3penA0MefPTxl7yjOHtRX
                                pTMe8MfCqwStqaXtbaF4MfEeWqy
                                MpiaLeqzYewoskcJMFdPgO2EaB3
                                PUbHG6Qk2Aboah0VRQp7vc/PYM2
                                E5o1gkF6agnqsh1xxNZJx6wIc8/
                                x3mIGUl5NOzUTqKQhBPlsRecCKI
                                HLXtrnKlseVOVSkJQpr3D04+y4u
                                HVektMYpbRjhd+XbtrPyQ2Tj/gB
                                aiZxFiuz++qqZMg7Sb7PIy7rug1
                                zDhVdQ9h0i9av90wBg2+ltIPdGE
                                mEylFR8mUXXbfSVljpB08uo4YcW
                                KdMjGcU5E90/VwVgnksP9X7ZuSY
                                OZIHZinms4NAz6AFnFpjB6wC0l8R
                                ) ; KSK; alg = RSASHA1 ; key id = 38713
olympus.local.          10800 IN DNSKEY 256 3 5 (
                                AwEAAd1RzPufNYrRD/deenr8YdP
                                nQXRnOwYajJWTldeI7jMG6LwcDyI
                                7G8=
                                ) ; ZSK; alg = RSASHA1 ; key id = 53451
;; Query time: 0 msec
;; SERVER: 10.0.2.7#53(10.0.2.7)
;; WHEN: Mon Apr 08 09:13:56 EEST 2024
;; MSG SIZE  rcvd: 658```

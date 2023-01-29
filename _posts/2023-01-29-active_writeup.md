---
layout: post
title: htb-active walkthrough
---

# NMAP

> Starting off with our nmap scan we can see that ports 53 (DNS), 88 (Kerberos), 135 (RPC), 139/445 (SMB) and 389/636/3268/3269 (LDAP) are open 
> we can make an educated guess and say this is a domain controller lets poke at the listed services above and gather more information.

```bash
sudo nmap -Pn -T4 -sC -sV -p- -o active.nmap 10.129.190.90 -vv
```

```
Nmap scan report for 10.129.190.90
Host is up, received user-set (0.052s latency).
Scanned at 2023-01-29 01:19:02 EST for 235s
Not shown: 65512 closed ports
Reason: 65512 resets
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2023-01-29 06:19:54Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5722/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49155/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49157/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49171/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49175/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49176/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
```
> We will use a null session with rpcclient to confirm this is a domain controller

```bash
rpcclient -U "" 10.129.190.90 -N
```
```
rpcclient $> srvinfo
	10.129.190.90  Wk Sv PDC Tim NT     Domain Controller
	platform_id     :	500
	os version      :	6.1
	server type     :	0x80102b
```
> When attacking active directory having the domain name may be useful so lets get the domain naming context from the rootDSE with nmap

```bash
sudo nmap -Pn -p 389 --script ldap-rootdse 10.129.190.90
```

```bash
Nmap scan report for 10.129.190.90
Host is up (0.045s latency).

PORT    STATE SERVICE
389/tcp open  ldap
| ldap-rootdse: 
| LDAP Results
|   <ROOT>
|       currentTime: 20230129184904.0Z
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=active,DC=htb
|       dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=active,DC=htb
|       namingContexts: DC=active,DC=htb
|       namingContexts: CN=Configuration,DC=active,DC=htb
|       namingContexts: CN=Schema,CN=Configuration,DC=active,DC=htb
|       namingContexts: DC=DomainDnsZones,DC=active,DC=htb
|       namingContexts: DC=ForestDnsZones,DC=active,DC=htb
|       defaultNamingContext: DC=active,DC=htb
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=active,DC=htb
|       configurationNamingContext: CN=Configuration,DC=active,DC=htb
|       rootDomainNamingContext: DC=active,DC=htb
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       highestCommittedUSN: 110705
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       dnsHostName: DC.active.htb
|       ldapServiceName: active.htb:dc$@ACTIVE.HTB
|       serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=active,DC=htb
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       isSynchronized: TRUE
|       isGlobalCatalogReady: TRUE
|       domainFunctionality: 4
|       forestFunctionality: 4
|_      domainControllerFunctionality: 4
Service Info: Host: DC; OS: Windows 2008 R2
```

> With the domain naming context we can see if we have access to anyshares
> we can also get the domain name from CME so we will do both enumerate 
> shares and resolve the domain name

> Since we were able to authenticate to the DC with a null session we do the same with CME

```bash
crackmapexec smb 10.129.190.90 -u "" -p "" --shares
```

```bash
SMB         10.129.190.90   445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.190.90   445    DC               [+] active.htb\: 
SMB         10.129.190.90   445    DC               [+] Enumerated shares
SMB         10.129.190.90   445    DC               Share           Permissions     Remark
SMB         10.129.190.90   445    DC               -----           -----------     ------
SMB         10.129.190.90   445    DC               ADMIN$                          Remote Admin
SMB         10.129.190.90   445    DC               C$                              Default share
SMB         10.129.190.90   445    DC               IPC$                            Remote IPC
SMB         10.129.190.90   445    DC               NETLOGON                        Logon server share 
SMB         10.129.190.90   445    DC               Replication     READ            
SMB         10.129.190.90   445    DC               SYSVOL                          Logon server share 
SMB         10.129.190.90   445    DC               Users                           
```
>  As we can see above, without adding `-d active.htb` in the command we can get the domain name this could be useful if you cant get it from LDAP for some reason

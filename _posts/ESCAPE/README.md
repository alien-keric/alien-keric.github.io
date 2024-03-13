# ESCAPE
Escape is a medium windows box from HTB with a feature like Active directory, where initial begin by scanning the network and from the  scanning reveals that there many ports, smb,msrpc(for connection via remotely means we can connect with evil-winrm) and so many.But the most interesting one is smb which gives us anonymous login and inside there is mssql-server pdf file which gives us some credentials for sql-server, after connecting to the sql-server we find that we can exec commands, with this we can start our responder and exec any command and find that we are able to extract the sql_svc hash which can easy be cracked with either john or hashcat, connecting with evil-winrm find that a backup file left inside a user sql_svc with ryan.cooper credentials, connect back to ryan.cooper. By enumerating privilege escalation we found vulnerable Certificates, trying to abuse the certificate and we find that we can manage to get the administrator hash with certify-ad tool and then try to connect back to evil-winrm by passing the hash, That the general over view of this AD machine.


REFERENCE:

https://redfoxsec.com/blog/exploiting-active-directory-certificate-services-ad-cs/
https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server
https://learn.microsoft.com/en-us/sql/relational-databases/lesson-1-connecting-to-the-database-engine?view=sql-server-ver16
https://github.com/alien-keric/Ghostpack-CompiledBinaries
https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation
https://github.com/ly4k/Certipy?tab=readme-ov-file#esc1

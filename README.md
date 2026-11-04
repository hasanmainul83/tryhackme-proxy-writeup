# TryHackMe – Proxy

## Complete Walkthrough | Active Directory Security Testing

A detailed, step-by-step walkthrough of the TryHackMe Proxy room, covering Active Directory enumeration, service account exploitation, NTLM hash capture, Kerberos delegation abuse, and Domain Controller compromise.

### Room Information

| **Detail** | **Information** |
|---|---|
| Platform | TryHackMe |
| Room | Proxy |
| Target OS | Windows Server |
| Attacking Machine | Kali Linux |
| Domain | `ctf.local` |
| Domain Controller | `DC01.ctf.local` |
| Objective | Exploit a careless service account and compromise the Domain Controller |

### Topics Covered

- Nmap scanning and service enumeration
- Active Directory and SMB enumeration
- SMB share discovery and file retrieval
- Internal IT portal analysis
- Service account enumeration
- NTLMv2 hash capture using Responder
- Password hash cracking using John the Ripper
- Kerberos authentication and ticket enumeration
- Kerberos constrained delegation
- S4U2Self and S4U2Proxy exploitation
- Administrator impersonation
- Accessing the Domain Controller using Impacket

### Repository Contents

- [**proxy.md**](https://github.com/hasanmainul83/tryhackme-proxy-writeup/blob/main/proxy.md) – Complete walkthrough with commands, explanations, and exploitation steps.
- [**images/**](https://github.com/hasanmainul83/tryhackme-proxy-writeup/tree/main/images) – Screenshots documenting the enumeration and exploitation process.


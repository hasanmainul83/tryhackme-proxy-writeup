# TryHackMe – Proxy

## Complete Walkthrough | Active Directory Security Testing

A simple, beginner-friendly walkthrough of the TryHackMe Proxy room. This write-up explains the entire process step by step, with clear commands and straightforward explanations so you can follow along without confusion.

No unnecessary complexity or complicated explanations — just the commands, what to look for, what the results mean, and how to move to the next step.

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
- Active Directory enumeration
- SMB enumeration and file retrieval
- Internal IT portal analysis
- Service account discovery
- NTLMv2 hash capture using Responder
- Password cracking using John the Ripper
- Kerberos authentication and ticket enumeration
- Kerberos constrained delegation
- S4U2Self and S4U2Proxy
- Administrator impersonation
- Domain Controller access using Impacket

### Repository Contents

- [**proxy.md**](https://github.com/hasanmainul83/tryhackme-proxy-writeup/blob/main/proxy.md) – Complete step-by-step walkthrough with commands and simple explanations.
- [**images/**](https://github.com/hasanmainul83/tryhackme-proxy-writeup/tree/main/images) – Screenshots documenting the enumeration and exploitation process.

### What to Expect

This walkthrough is written to make the learning process as straightforward as possible. Each step explains what to do, what to notice, and why it matters before moving forward.

Whether you're new to Active Directory or already familiar with its fundamentals, the goal is to help you understand the exploitation process without getting lost in unnecessary details.


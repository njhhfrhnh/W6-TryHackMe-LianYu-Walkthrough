# W6-TryHackMe-LianYu-Walkthrough
## **Summary**

| Phase                     | Action                                  | Outcome                                      |
|--------------------------|-----------------------------------------|----------------------------------------------|
| Reconnaissance           | `Nmap` scan                             | Identified open ports (FTP, SSH, HTTP)       |
| Scanning & Enumeration   | `Gobuster`, source code review          | Found `/island` and `/2100`                  |
| Scanning & Enumeration   | Gobuster with `.ticket` extension       | Discovered `/green_arrow.ticket`             |
| Gaining Access           | Base58 decoding                         | Retrieved FTP credentials                    |
| Exploitation             | FTP login + file extraction             | Discovered hidden files & images             |
| Post-Exploitation        | File analysis + steganography           | Extracted SSH credentials                    |
| Privilege Escalation     | `pkexec` via GTFOBins                   | Root access obtained                         |

---

## **1. Reconnaissance (Information Gathering)**

**Objective:** Identify target surface and exposed services.

Always start with an Nmap scan to discover open ports and services.

```bash
nmap -sC -sV -Pn -T4 -oN nmap/lianyu 10.48.140.128
```
<img width="940" height="564" alt="image" src="https://github.com/user-attachments/assets/c3ddbfbe-7e42-437a-8daf-41147e47b10d" />


**Findings:**

| Service | Port | What can be done (commands / next step) | Findings              |
| ------- | ---- | --------------------------------------- | --------------------- |
| FTP     | 21   | `ftp 10.48.140.128`                     | Requires credentials  |
| SSH     | 22   | `ssh user@10.48.140.128`                | Possible remote login |
| HTTP    | 80   | Open in browser                         | Main entry point      |
| RPC     | 111  | Enumeration possible                    | Low priority          |

The scan shows multiple services, but HTTP (port 80) is the best starting point because web applications often expose hidden directories or sensitive data.








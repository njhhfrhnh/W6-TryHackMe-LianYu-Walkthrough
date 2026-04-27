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

I started by running an Nmap scan to identify open ports and services available on the target machine. This helps me understand the attack surface and decide where to begin.

```bash
nmap -sC -sV -Pn -T4 -oN nmap/lianyu 10.48.140.128
```
-sC : Default scripts
-sV : Version detection
-p- : All ports to scan
-oN : Output to be stored in the directory ‘nmap’ you created earlier

<img width="940" height="564" alt="image" src="https://github.com/user-attachments/assets/c3ddbfbe-7e42-437a-8daf-41147e47b10d" />


| Service | Port | What can be done (commands / next step) | 
| ------- | ---- | --------------------------------------- |
| FTP     | 21   | `ftp 10.48.140.128`                     | 
| SSH     | 22   | `ssh user@10.48.140.128`                | 
| HTTP    | 80   | Open in browser                         | 
| RPC     | 111  | Enumeration possible                    | 

The scan shows multiple services, but HTTP (port 80) is the best starting point because web applications often expose hidden directories or sensitive data.

## **2. Scanning & Enumeration**

### Access Website

Since port 80 is open, I opened the target IP in the browser to see what is available.

```bash
http://10.48.140.128
```
<img width="940" height="686" alt="image" src="https://github.com/user-attachments/assets/79997417-053d-4bab-af02-7df3c62a5080" />

The website shows an Arrowverse-themed page. This confirms that the HTTP service is running properly and can be explored further.

### Directory Enumeration

```bash
gobuster dir -u http://10.48.140.128 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
<img src="ADD_GOBUSTER_SCREENSHOT" />

This revealed a directory:
```bash
/island
```
### Exploring /island

I navigated to /island and found another clue.

<img width="940" height="273" alt="image" src="https://github.com/user-attachments/assets/4b8aec55-a10c-46b6-ac7a-8ea65422bf31" />
<img width="940" height="440" alt="image" src="https://github.com/user-attachments/assets/d688854e-7c6a-4d5c-9027-0f4752beb160" />

By highlighting the text and open source code, I discovered the word:
```bash
vigilante
```
This looks like a code word, so I kept it in mind as it might be useful later (possibly as a username or password).

### Further Enumeration








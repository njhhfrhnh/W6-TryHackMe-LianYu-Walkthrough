# W6-TryHackMe-LianYu-Walkthrough
---

## **Summary**

| Phase                   | Action                          | Outcome                                  |
|------------------------|----------------------------------|------------------------------------------|
| Reconnaissance         | Nmap scan                        | Identified FTP, SSH, HTTP, RPC           |
| Scanning & Enumeration | Gobuster, web analysis           | Found `/island`, `/2100`, `.ticket` file |
| Gaining Access         | FTP login                        | Access to image files                    |
| Exploitation           | Steganography + decoding         | Retrieved SSH credentials                |
| Post-Exploitation      | SSH access                       | Retrieved user flag                      |
| Privilege Escalation   | `pkexec` exploitation            | Root access obtained                     |

---

## **1. Reconnaissance (Information Gathering)**

**Objective:** Identify open ports and services.

```bash
nmap -sC -sV -Pn -T4 -oN nmap/lianyu <TARGET_IP>
<img width="940" height="564" alt="image" src="https://github.com/user-attachments/assets/4a2c4a96-2e65-4f06-b3bf-d8831d16a4f1" />

---

## **2. Scanning & Enumeration**

**Objective:** Extract deeper information from identified services.

After identifying HTTP service on port 80 from the Nmap scan, I opened the web application in the browser:

```bash
http://<TARGET_IP>
<img width="940" height="686" alt="image" src="https://github.com/user-attachments/assets/7a8b6726-4e2a-4be2-9be8-793db07ef8c4" />


The webpage displayed **Purgatory**, which suggests that there may be hidden content or directories that are not directly visible.

---

Next, I performed directory brute-forcing using Gobuster to discover hidden directories:

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
<img width="940" height="306" alt="image" src="https://github.com/user-attachments/assets/4b8804e1-0bed-4b16-aa93-ec976a207888" />

From the result, I found a directory:
```bash
/island

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






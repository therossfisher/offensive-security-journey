# THM - [[Guided Pentest: WEB](https://tryhackme.com/room/guidedpentestweb)]

**Path:** Jr Penetration Tester  
**Date:** 02JUN2026
**Difficulty:** Easy 

---

## What This Room Covers

*This room will guide you through a realistic web application penetration test from start to finish.*

---

## Key Concepts Learned

*Bullet points. What do you now understand that you didn't before?*

-Recon and enumeration
- ### NMAP
-   ```httponly flag not set```
	 -  The app uses PHP sessions and the `httponly` flag is NOT set on the cookie. That means if you find an XSS vulnerability, you can steal session cookies through JavaScript.
 - 3306/tcp open mysql MySQL (unauthorized)
	 - A MySQL database is exposed and saying unauthorized. Means it's running but rejecting unauthenticated connections. Databases should never be directly exposed like this — it's a miss-configuration. If you find database credentials anywhere in the app you may be able to connect directly.

- 8080/tcp open http Apache httpd 2.4.58 |_http-title: Apache2 Ubuntu Default Page: It works
	- A second web server on port 8080 showing just the default Apache page. Could be a staging environment, admin panel, or something misconfigured. Worth poking at separately.
- ### curl
	- curl -I http://10.144.152.10
HTTP/1.1 200 OK # server responded succesfully
Date: Wed, 03 Jun 2026 06:52:46 GMT # server's current timestamp
Server: Apache/2.4.58 (Ubuntu) # This is information disclosure,telling exactly what software it runs and version. A hardened server would hide this.
Set-Cookie: PHPSESSID=p9ve34vk1lumeoj5k62uspoh8o; path=/ # confirms app was built in PHP. A secure cookie should say "HttpONly; Secure" at the end. This one has neither. No HttpOnly means JavaScript can read this cookie -- XSS to session theft is a viable attack chain. No secure means the cookie could be sent over un-encrypted HTTP.
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache # The app is telling browsers not to cache any pages. This is actually intentional secure behavior for a login-based app — stops sensitive pages being stored in browser cache. Not a vulnerability, just how PHP apps typically handle sessions.
Content-Type: text/html; charset=UTF-8 # Server is returning HTML. Confirms it's a standard web application.

- Service Info: OS: Linux
	- Confirmed Linux target. Combined with the Ubuntu references above — you know the OS.


 -Insecure Direct Object Reference
-Weak password reset
-Admin panel access
-Remote code execution

---

## Tools Used

| Tool     | What I Used It For                             |
| -------- | ---------------------------------------------- |
| nmap     | Used to scan the target machine                |
| curl     | command line tool for making HTTP requests     |
| Gobuster | Gobuster is a directory and file brute-forcer. |

---

## Commands Worth Remembering

```bash
#NMAP
-sC # runs default scripts, grabs extra info automatically
-sV # detects service versions
-oN # saves output to a file so you have it for your writeup
-p- # scans all 65535 ports instead of just the top 1000. Always use `-p-` so you don't miss anything hiding on unusual ports.

#curl
-I # tells curl to fetch only the headers, metadata of the HTTP response, not the actual page content. Like asking server "tell me about yourself" without downloading the whole page.

#gobuster
gobuster dir -u http://10.144.152.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -x php

#Purpose: Brute-force hidden directories and PHP files #Wordlist: dirbuster small (87k entries)
#Extension: .php


```

---

## Notes

*Anything that confused you, surprised you, or that you want to revisit.*
- Definitely need to get more familiar with OpenVPN and connecting to machines this way. Lot's of capture the flags require it
- Nmap - I need to get familiar with this tool and learn all the switches, how they work, how to use them and the syntax for scanning.
- Gobuster, never heard of this tool. Summary: It takes a wordlist — a huge list of common directory and file names — and tries every single one against the target web server. For each word it makes an HTTP request and checks if something exists there. It's finding hidden pages, admin panels, upload directories, config files, and anything else the developer didn't link to publicly but left accessible.
	- shell-session
```gobuster dir -u http://10.144.152.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -x php```


Think of it like trying every door in a building to see which ones are unlocked, instead of just using the front entrance.

Findings:
curl -I http://10.144.152.10

- Server: Apache/2.4.58 (Ubuntu) — version disclosure
- PHP application confirmed via PHPSESSID cookie
- Cookie missing HttpOnly flag — XSS to session theft viable
- Cookie missing Secure flag — cookie transmitted over HTTP

---

## Flags

`User flag:` 
`Root flag:` 

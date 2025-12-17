# All-in-One-Bug-Bounty-Methodology-
<!-- Header with title -->
<h1 align="center">🐞 Bug Bounty Methodology</h1>

<!-- Social Badges -->
<p align="center">
  <a href="<YOUR_LINKEDIN>">
    <img src="https://img.shields.io/badge/LinkedIn-blue?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn Badge">
  </a>
  <a href="<YOUR_TWITTER>">
    <img src="https://img.shields.io/badge/Twitter-blue?style=for-the-badge&logo=twitter&logoColor=white" alt="Twitter Badge">
  </a>
</p>

Bug bounty workflow can be broken into three big phases: Recon, App Logic, and Attack Surface exploration. Below is a ready-to-use, updated methodology you can refine over time.

***

# Bug Bounty Methodology  
Target: example.com

------------------------------------------------
1. Recon (Information Gathering)
------------------------------------------------
Goal: Map the target as widely as possible (hosts, tech stack, endpoints) so you know where to attack.

1.1 Subdomain Enumeration  
Notes (simple): Find all subdomains so you don’t miss hidden panels, APIs, or staging environments.

Tools (examples):  
- subfinder  
- amass  
- assetfinder  
- crt.sh  
- httpx  

Sample commands (replace with your real target/link):  
- Passive subdomains:  
  - `subfinder -d example.com -silent -all -o subfinder.txt`  
  - `amass enum -passive -d example.com -o amass.txt`  
  - `assetfinder --subs-only example.com > assetfinder.txt`  
  - Manually export from crt.sh and save as `crtsh.txt`  

1.2 Merge and Deduplicate Results  
Combine all tools’ output and remove duplicates.

- `cat subfinder.txt amass.txt assetfinder.txt crtsh.txt | anew all.txt`  

1.3 Probe for Alive Subdomains  
Check which subdomains actually respond over HTTP/HTTPS.

- `cat all.txt | httpx -silent -status-code -title -o alive.txt`  

1.4 Scan Open Ports  
Notes (simple): Open ports reveal other services (SSH, databases, SMTP, custom apps) that might be in scope.

Basic tools:  
- naabu (fast host/port scanner)  
- nmap (detailed port/service scanner)

Sample commands:  
- Fast port scan (naabu):  
  - `cat all.txt | naabu -top-1000 -o ports.txt`  
- Basic nmap scan:  
  - `nmap -sV -sC -p- -T4 example.com` (full TCP ports, version + default scripts)[1][2][3][4][5]

1.5 Directory and File Brute Force  
Notes (simple): Discover hidden paths like `/admin`, `/api/`, `/backup/` that are not linked anywhere.

Tools:  
- ffuf  
- dirb  

Wordlists (examples):  
- `https://github.com/ArwaGodFather/wordlists`  
- `https://github.com/danielmiessler/SecLists`[6][7][8]

Sample ffuf usage:  
- `ffuf -u https://target.com/FUZZ -w /path/to/wordlist.txt`  
  - `FUZZ` is the placeholder replaced by each word in the list.[9][10][8][6]

Sample dirb usage:  
- `dirb https://target.com /path/to/wordlist.txt`  

1.6 Content / Endpoint Discovery (URLs, APIs, JS)  
Notes (simple): Find old URLs, API endpoints, and parameters using archives, crawlers, and JS files.

Tools:  
- waybackurls  
- waymore  
- gau (GetAllURLs)  
- hakrawler  
- katana  
- Burp Suite Spider/Crawler[11][12][13][14][15]

Sample commands:  
- `echo "example.com" | waybackurls > wayback.txt`  
- `echo "example.com" | gau > gau.txt`  
- `cat alive.txt | hakrawler -plain -depth 3 -scope subs > hakrawler.txt`  
- `katana -u https://example.com -d 3 -silent -o katana.txt`  

1.7 Google Dorking  
Notes (simple): Use Google to find exposed files, panels, and misconfigurations.

Sample dorks:  
- `site:example.com -site:www.example.com`  
- `site:example.com ext:php`  
- `site:example.com inurl:admin`  
- `site:example.com "index of /"`  

***

------------------------------------------------
2. App Logic (Understanding How the App Works)
------------------------------------------------
Goal: Understand how the application behaves so you can think like a user and then like an attacker.

Simple mindset:  
- Log in if possible (test/normal account, or registration).  
- Click every button, link, tab, and form.  
- Watch how data flows: what is created, updated, deleted, or shared.

Practical steps:  
- Open Burp Suite (or any proxy) and turn intercept ON for a while.  
- Click through the whole app:  
  - Dashboard, profile, settings, search, filters, upload/download, payments, sharing, invitations.  
- For every action:  
  - Observe the HTTP request (URL, method, parameters, headers, cookies, body).  
  - Observe the HTTP response (status code, JSON/XML/HTML, error messages).  
- Try modifying the request:  
  - Change IDs, emails, roles, prices, file names, parameters, methods.  
  - Remove parameters or send unexpected values.  
  - Replay requests (possible IDOR, access control issues).  

Look for logic issues such as:  
- Bypass of payment steps.  
- Changing role from `user` to `admin` in request.  
- Accessing other users’ data by modifying IDs (IDOR).  
- Business logic flaws: discounts, coupons, referral abuse, password reset logic, etc.

***

------------------------------------------------
3. Attack Surface (Where and How to Attack)
------------------------------------------------
Goal: Take everything discovered (hosts, endpoints, parameters, features) and systematically test them like an attacker.

3.1 Build a Target List  
From recon + app logic, build lists of:  
- Alive hosts (`alive.txt`)  
- Important paths (from ffuf/dirb)  
- Parameters and URLs (from waybackurls/gau/hakrawler/katana/Burp)  
- Key app features (upload, login, reset password, invitations, messaging, etc.)  

3.2 Play With Every Function (With Intercept)  
Simple rule: If the app lets you do something, ask “What if I abuse this?”  

Examples:  
- Authentication flows  
  - Test weak passwords, missing rate limiting, 2FA bypass, session fixation.  
- Authorization  
  - Change user IDs or resource IDs in requests to access other users’ data.  
- File upload  
  - Try other file types, double extensions, bypass content-type checks.  
- Input fields  
  - Test for XSS, SQLi, SSTI, SSRF, command injection depending on context.  
- APIs  
  - Look for undocumented endpoints, mass assignment, missing auth on some methods.  

3.3 Think Like an Attacker  
Ask yourself:  
- “If I were malicious, how would I use this feature in a way the developer didn’t expect?”  
- “Can I skip steps, call internal endpoints directly, or chain small bugs into something big?”  
- “What happens if I automate this 1000 times?”  

Use tools to support manual testing:  
- Burp Suite (Repeater, Intruder, Proxy, Comparer, Logger).  
- Custom scripts (Python, bash) to replay/modify traffic.  

***

## ✨ Contributing
We welcome contributions to improve this methodology!  
To contribute:  
1. Fork this repository.  
2. Create a feature branch (`git checkout -b feature-name`).  
3. Make your changes and commit (`git commit -m "Add improvement"`).  
4. Push to the branch (`git push origin feature-name`).  
5. Open a Pull Request and describe your changes.  
If you want to discuss a major change, open an issue first. :contentReference[oaicite:1]{index=1}

***



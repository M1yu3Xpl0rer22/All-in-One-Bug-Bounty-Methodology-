# 🐞 All-in-One Bug Bounty Methodology

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mayur_Sapkale-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/mayur-sapkale-72855b25b/)
[![Twitter](https://img.shields.io/badge/Twitter-@localhost12001-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://x.com/localhost12001)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/mayursapkale)

**A comprehensive, ready-to-use bug bounty hunting methodology**

[🎯 Getting Started](#-phase-1-reconnaissance) • [🔧 Tools](#-essential-tools) • [🤝 Contributing](#-contributing)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Phase 1: Reconnaissance](#-phase-1-reconnaissance)
- [Phase 2: Application Logic Analysis](#-phase-2-application-logic-analysis)
- [Phase 3: Attack Surface Exploration](#-phase-3-attack-surface-exploration)
- [Essential Tools](#-essential-tools)
- [Contributing](#-contributing)

---

## 🎯 Overview

Bug bounty hunting workflow is divided into three critical phases:

1. **🔍 Reconnaissance** - Map the entire attack surface
2. **🧠 Application Logic** - Understand how the app works
3. **⚔️ Attack Surface** - Systematically test for vulnerabilities

**Target Example:** `example.com`

---

## 🔍 Phase 1: Reconnaissance

> **Goal:** Map the target comprehensively to identify all potential entry points including hosts, technology stack, and endpoints.

### 1.1 📡 Subdomain Enumeration

**Purpose:** Discover hidden subdomains that may contain admin panels, APIs, staging environments, or forgotten assets.

#### 🛠️ Tools

| Tool | GitHub Link | Description |
|------|-------------|-------------|
| **Subfinder** | [projectdiscovery/subfinder](https://github.com/projectdiscovery/subfinder) | Fast passive subdomain discovery |
| **Amass** | [owasp-amass/amass](https://github.com/owasp-amass/amass) | In-depth DNS enumeration |
| **Assetfinder** | [tomnomnom/assetfinder](https://github.com/tomnomnom/assetfinder) | Find domains and subdomains |
| **crt.sh** | [crt.sh](https://crt.sh) | Certificate transparency logs |
| **httpx** | [projectdiscovery/httpx](https://github.com/projectdiscovery/httpx) | Fast HTTP probe |

#### 📝 Commands

```bash
# Passive subdomain enumeration
subfinder -d example.com -silent -all -o subfinder.txt
amass enum -passive -d example.com -o amass.txt
assetfinder --subs-only example.com > assetfinder.txt

# Manual: Export from crt.sh and save as crtsh.txt
```

---

### 1.2 🔗 Merge & Deduplicate

**Combine all results and remove duplicates:**

```bash
cat subfinder.txt amass.txt assetfinder.txt crtsh.txt | anew all.txt
```

**Tool:** [tomnomnom/anew](https://github.com/tomnomnom/anew)

---

### 1.3 ✅ Probe for Live Subdomains

**Check which subdomains are actually responding:**

```bash
cat all.txt | httpx -silent -status-code -title -o alive.txt
```

---

### 1.4 🔌 Port Scanning

**Purpose:** Identify open ports that may expose additional services (SSH, databases, SMTP, custom applications).

#### 🛠️ Tools

| Tool | GitHub Link | Description |
|------|-------------|-------------|
| **Naabu** | [projectdiscovery/naabu](https://github.com/projectdiscovery/naabu) | Fast port scanner |
| **Nmap** | [nmap/nmap](https://github.com/nmap/nmap) | Network exploration tool |

#### 📝 Commands

```bash
# Fast port scan
cat all.txt | naabu -top-ports 1000 -o ports.txt

# Detailed scan with service detection
nmap -sV -sC -p- -T4 example.com -oN nmap_scan.txt
```

---

### 1.5 📂 Directory & File Discovery

**Purpose:** Find hidden paths like `/admin`, `/api/`, `/backup/`, `.git/`, config files that aren't publicly linked.

#### 🛠️ Tools

| Tool | GitHub Link | Description |
|------|-------------|-------------|
| **ffuf** | [ffuf/ffuf](https://github.com/ffuf/ffuf) | Fast web fuzzer |
| **dirb** | [dirb](http://dirb.sourceforge.net/) | URL bruteforcer |
| **dirsearch** | [maurosoria/dirsearch](https://github.com/maurosoria/dirsearch) | Web path scanner |

#### 📚 Wordlists

- [danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)
- [assetnote/commonspeak2-wordlists](https://github.com/assetnote/commonspeak2-wordlists)
- [ArwaGodFather/wordlists](https://github.com/ArwaGodFather/wordlists)

#### 📝 Commands

```bash
# ffuf fuzzing
ffuf -u https://example.com/FUZZ -w /path/to/wordlist.txt -mc 200,301,302,403

# dirb
dirb https://example.com /path/to/wordlist.txt

# dirsearch
dirsearch -u https://example.com -w /path/to/wordlist.txt
```

---

### 1.6 🌐 Content & Endpoint Discovery

**Purpose:** Find historical URLs, API endpoints, parameters, and JavaScript files using archives and crawlers.

#### 🛠️ Tools

| Tool | GitHub Link | Description |
|------|-------------|-------------|
| **waybackurls** | [tomnomnom/waybackurls](https://github.com/tomnomnom/waybackurls) | Fetch URLs from Wayback Machine |
| **gau** | [lc/gau](https://github.com/lc/gau) | Get all URLs from multiple sources |
| **hakrawler** | [hakluke/hakrawler](https://github.com/hakluke/hakrawler) | Web crawler for hackers |
| **katana** | [projectdiscovery/katana](https://github.com/projectdiscovery/katana) | Next-gen crawling framework |
| **waymore** | [xnl-h4ck3r/waymore](https://github.com/xnl-h4ck3r/waymore) | Enhanced wayback URLs |
| **Burp Suite** | [PortSwigger](https://portswigger.net/burp) | Web security testing toolkit |

#### 📝 Commands

```bash
# Wayback Machine URLs
echo "example.com" | waybackurls > wayback.txt

# Get all URLs
echo "example.com" | gau > gau.txt

# Crawl with hakrawler
cat alive.txt | hakrawler -plain -depth 3 -scope subs > hakrawler.txt

# Katana crawling
katana -u https://example.com -d 3 -silent -o katana.txt

# Waymore
waymore -i example.com -mode U -oU waymore.txt
```

---

### 1.7 🔎 Google Dorking

**Purpose:** Leverage Google's search capabilities to find exposed files, admin panels, and misconfigurations.

#### 📝 Sample Dorks

```
site:example.com -site:www.example.com
site:example.com ext:php
site:example.com ext:pdf
site:example.com inurl:admin
site:example.com inurl:login
site:example.com "index of /"
site:example.com intitle:"Dashboard"
site:example.com filetype:env
site:example.com filetype:sql
```

**Resource:** [Google Hacking Database](https://www.exploit-db.com/google-hacking-database)

---

## 🧠 Phase 2: Application Logic Analysis

> **Goal:** Deeply understand application behavior to identify logic flaws that automated scanners miss.

### 🎯 Mindset

Think like a user first, then like an attacker. Every feature is a potential vulnerability.

### 📋 Systematic Approach

#### 1. **Setup Proxy**
   - Configure Burp Suite or similar proxy
   - Enable intercept and logging
   - Set scope to target domain

#### 2. **Complete User Journey**

Explore every feature:
- ✅ Registration & Login
- ✅ Profile Management
- ✅ Password Reset Flow
- ✅ File Upload/Download
- ✅ Search & Filters
- ✅ Payment Processing
- ✅ Social Features (sharing, messaging)
- ✅ Settings & Preferences
- ✅ API Interactions

#### 3. **Traffic Analysis**

For each action, examine:
- **Request:** Method, URL, parameters, headers, cookies, body
- **Response:** Status code, content type, data structure, error messages
- **Authentication:** Session tokens, JWTs, API keys
- **Authorization:** Role checks, resource access

#### 4. **Manipulation Testing**

Try modifying:
- ✏️ IDs (user IDs, order IDs, document IDs)
- ✏️ Email addresses
- ✏️ Roles and permissions
- ✏️ Prices and quantities
- ✏️ File names and types
- ✏️ HTTP methods (GET ↔ POST ↔ PUT ↔ DELETE)
- ✏️ Parameters (add, remove, change values)

---

### 🚨 Common Logic Flaws to Look For

| Category | Examples |
|----------|----------|
| **Authentication** | Weak passwords, missing rate limiting, 2FA bypass, session fixation |
| **Authorization** | IDOR, privilege escalation, missing access controls |
| **Business Logic** | Payment bypass, discount abuse, coupon manipulation, referral fraud |
| **Race Conditions** | Double spending, concurrent requests exploitation |
| **State Management** | Session handling issues, workflow bypass |

---

## ⚔️ Phase 3: Attack Surface Exploration

> **Goal:** Systematically test every discovered asset and functionality for vulnerabilities.

### 3.1 📊 Build Target List

Organize your findings:

```
✅ Live hosts (alive.txt)
✅ Important paths (from ffuf/dirb)
✅ Parameters and URLs (from crawlers)
✅ Key features (upload, login, API endpoints)
✅ Technology stack (from Wappalyzer, headers)
```

---

### 3.2 🔬 Systematic Testing

#### Authentication & Session Management
- [ ] Brute force protection
- [ ] Password complexity requirements
- [ ] Session timeout
- [ ] Logout functionality
- [ ] JWT token validation
- [ ] OAuth implementation flaws

#### Authorization & Access Control
- [ ] IDOR (change IDs in requests)
- [ ] Privilege escalation (modify roles)
- [ ] Missing function level access control
- [ ] Path traversal
- [ ] API endpoint authorization

#### Input Validation
- [ ] XSS (reflected, stored, DOM-based)
- [ ] SQL Injection
- [ ] Command Injection
- [ ] SSTI (Server-Side Template Injection)
- [ ] XXE (XML External Entity)
- [ ] SSRF (Server-Side Request Forgery)

#### File Operations
- [ ] Unrestricted file upload
- [ ] Path traversal in downloads
- [ ] File inclusion vulnerabilities
- [ ] ZIP file exploits

#### API Security
- [ ] Undocumented endpoints
- [ ] Mass assignment
- [ ] Excessive data exposure
- [ ] Rate limiting
- [ ] CORS misconfigurations

---

### 3.3 🧩 Attacker Mindset

Ask yourself:

> 💭 "How would a malicious actor abuse this feature?"  
> 💭 "Can I skip steps in this workflow?"  
> 💭 "What happens if I automate this 1000 times?"  
> 💭 "Can I chain multiple small issues into a critical exploit?"

---

### 🛠️ Testing Tools

| Tool | GitHub Link | Purpose |
|------|-------------|---------|
| **Burp Suite** | [PortSwigger](https://portswigger.net/burp) | Primary testing platform |
| **SQLMap** | [sqlmapproject/sqlmap](https://github.com/sqlmapproject/sqlmap) | SQL injection testing |
| **XSStrike** | [s0md3v/XSStrike](https://github.com/s0md3v/XSStrike) | XSS detection |
| **Nuclei** | [projectdiscovery/nuclei](https://github.com/projectdiscovery/nuclei) | Template-based scanning |
| **Arjun** | [s0md3v/Arjun](https://github.com/s0md3v/Arjun) | Parameter discovery |
| **FFUF** | [ffuf/ffuf](https://github.com/ffuf/ffuf) | Fuzzing & parameter testing |

---

## 🧰 Essential Tools

### 🔧 All-in-One Collections

- [ProjectDiscovery Tools](https://github.com/projectdiscovery) - Modern recon & scanning suite
- [OWASP Tools](https://owasp.org/www-community/Free_for_Open_Source_Application_Security_Tools) - Security testing tools
- [Bug Bounty Tools](https://github.com/vavkamil/awesome-bugbounty-tools) - Curated list

### 📦 Installation Scripts

```bash
# Install common tools (Linux/macOS)
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install -v github.com/tomnomnom/anew@latest
go install -v github.com/lc/gau/v2/cmd/gau@latest
```

---

## 🤝 Contributing

We welcome contributions to improve this methodology!

### How to Contribute

1. **Fork** this repository
2. **Create** a feature branch: `git checkout -b feature-name`
3. **Make** your changes and commit: `git commit -m "Add improvement"`
4. **Push** to the branch: `git push origin feature-name`
5. **Open** a Pull Request with a detailed description

### Contribution Ideas

- ✨ Add new tools and techniques
- 📝 Improve explanations and examples
- 🐛 Fix errors or outdated information
- 🌍 Add translations
- 💡 Share real-world case studies

---

## 📞 Contact & Follow

<div align="center">

**Created by Mayur Sapkale**

[![LinkedIn](https://img.shields.io/badge/Connect_on-LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/mayur-sapkale-72855b25b/)
[![Twitter](https://img.shields.io/badge/Follow_on-Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://x.com/localhost12001)

*Found this helpful? Give it a ⭐ on GitHub!*

</div>

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

<div align="center">

**Happy Hunting! 🎯**

*Remember: Always hunt ethically and within program scope.*

</div>

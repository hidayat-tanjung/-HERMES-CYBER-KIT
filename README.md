# Hermes Cyber Kit
<img width="1774" height="887" alt="Neon Hermes Cyber Kit Poster" src="https://github.com/user-attachments/assets/83f26dc6-3aa3-4986-a6f6-06a9e8547a4e" />
Setup lengkap dari nol untuk menyiapkan Hermes Agent di Linux, memilih model, menghubungkan Telegram, dan menambahkan 9 skill custom untuk workflow bug hunting / security testing yang aman dan terstruktur.

| Informasi | Detail |
| --- | --- |
| Alur | Install Hermes → Pilih Model (Free/Paid) → Telegram → 9 Skill Bug Hunting |
| Platform | Linux (Ubuntu / Debian / Kali / Mint) |
| Total skill | 9 custom + 53 builtin = 62 skill |
| Perkiraan waktu setup | 30–60 menit |

## Daftar Isi

1. [Persiapan Awal](#0-persiapan-awal)
2. [Install Hermes Agent](#1-install-hermes-agent)
3. [Pilih Model: Free atau Paid](#2-pilih-model-free-atau-paid)
4. [Setup Telegram Bot](#3-setup-telegram-bot)
5. [Update Hermes](#4-update-hermes-opsional-disarankan)
6. [Membuat 9 Skill Custom](#5-membuat-9-skill-custom)
7. [Reload dan Verifikasi Skill](#6-reload-dan-verifikasi-skill)
8. [Membuat `scope.yaml`](#7-membuat-scopeyaml)
9. [Pengujian di Telegram](#8-pengujian-di-telegram)
10. [Backup ke GitHub](#9-backup-ke-github-opsional)
11. [Troubleshooting](#10-troubleshooting)
12. [Tautan Penting](#11-tautan-penting)
13. [Catatan Legal](#12-catatan-legal)
14. [Ringkasan Perintah Cepat](#13-ringkasan-perintah-cepat)

---

## 0. Persiapan Awal

Buka terminal Linux (`Ctrl + Alt + T`).

```bash
# Buat folder kerja
mkdir -p ~/hermes-cyber-kit/{scripts,skills/{vuln,recon,report},config,workflows,docs}
cd ~/hermes-cyber-kit

# Update sistem & install dependency
sudo apt update
sudo apt install -y curl git python3 python3-pip nano

# Cek versi
uname -a
python3 --version
```

---

## 1. Install Hermes Agent

```bash
# Install Hermes (1 perintah)
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# Reload shell
source ~/.bashrc

# Verifikasi
hermes --version
```

Output yang diharapkan:

```bash
hermes 0.21.5+xxxxx
```

---

## 2. Pilih Model: Free atau Paid

Di sini kamu punya 2 pilihan. Pilih salah satu, atau dua-duanya (free dulu, paid sebagai fallback).

### 2.1 Pilihan Free (OpenRouter Free Tier)

```bash
# LANGKAH 1: Dapatkan API Key
# 1. Buka browser: https://openrouter.ai/keys
# 2. Sign up gratis (bisa pakai GitHub/Google/Email)
# 3. Klik "Create Key"
# 4. Kasih nama: "Hermes Agent"
# 5. Copy API Key (format: sk-or-v1-xxxxxxxxxxxx)
# 6. SIMPAN DI TEMPAT AMAN — cuma muncul sekali!

# LANGKAH 2: Setup via wizard
hermes model
# Pilih: OpenRouter
# Paste: API Key
# Pilih model: openrouter/free

# ATAU setup manual:
mkdir -p ~/.hermes
echo "OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxx" >> ~/.hermes/.env

# Buat config.yaml
cat > ~/.hermes/config.yaml << 'EOF'
model:
  provider: openrouter
  default: "openrouter/free"

fallback_providers:
  - provider: openrouter
    model: "openrouter/free"

tools:
  rate_limit:
    nuclei: 10
    subfinder: 10
    httpx: 50
    ffuf: 20
EOF

# LANGKAH 3: Test
hermes chat -q "Reply with exactly one word: pong"
```

Output yang diharapkan:

```text
pong
```

Catatan free tier:

- 50 request per hari (gratis)
- 20 request per menit
- Reset harian jam 00:00 UTC
- Model gratis bisa berubah kapan saja
- Kalau model `:free` hilang, ganti ke `openrouter/free`

### 2.2 Pilihan Paid (Top Up $10 untuk 1.000 request/hari)

Kalau kamu butuh lebih dari 50 request per hari:

1. Buka: https://openrouter.ai/settings/credits
2. Klik "Add Credits"
3. Masukkan $10
4. Bayar via kartu kredit / crypto / Alipay
5. Limit gratis naik permanen dari 50 → 1.000 request/hari

Keuntungan top-up:

- Limit gratis naik 20x lipat
- Kredit tidak hangus (kecuali tidak dipakai 1 tahun)
- Bisa dipakai untuk model berbayar murah
- Tidak ada drama "model unavailable for free"

```bash
cat > ~/.hermes/config.yaml << 'EOF'
model:
  provider: openrouter
  default: "meta-llama/llama-3.3-70b-instruct"

fallback_providers:
  - provider: openrouter
    model: "qwen/qwen3-coder"

tools:
  rate_limit:
    nuclei: 10
    subfinder: 10
    httpx: 50
    ffuf: 20
EOF
```

Biaya model berbayar murah:

- Claude Haiku: ~$0.25 per 1M token input
- GPT-4o mini: ~$0.15 per 1M token input
- Llama 3.3 70B: ~$0.23 per 1M token input

Dengan $10, bisa ~40 juta token — cukup berbulan-bulan.

Cek saldo:

```bash
curl -s https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $(grep OPENROUTER ~/.hermes/.env | cut -d= -f2)"
```

### 2.3 Pilihan Alternatif (Tanpa OpenRouter)

#### Opsi 1: Ollama (100% lokal, gratis selamanya)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen2.5:14b
```

```bash
cat > ~/.hermes/config.yaml << 'EOF'
model:
  provider: ollama
  default: "qwen2.5:14b"
  base_url: "http://localhost:11434"
EOF
```

#### Opsi 2: Groq (gratis, cepat)

```bash
# Daftar: https://console.groq.com/keys
# Tambah ke ~/.hermes/.env:
# GROQ_API_KEY=gsk_xxxxxxxxxxxx
```

#### Opsi 3: Google Gemini (gratis, limit besar)

```bash
# Daftar: https://aistudio.google.com/apikey
# Tambah ke ~/.hermes/.env:
# GOOGLE_API_KEY=AIzaxxxxxxxxxxxx
```

---

## 3. Setup Telegram Bot

### Langkah 1: Buat Bot di Telegram

1. Buka Telegram, cari `@BotFather`
2. Kirim: `/newbot`
3. Kasih nama bot (contoh: `My Hermes Bot`)
4. Kasih username, HARUS diakhiri `_bot`
5. BotFather kasih token: `1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ`
6. Copy token

### Langkah 2: Dapatkan User ID kamu

1. Cari `@userinfobot`
2. Kirim pesan apa saja
3. Bot balas: `Id: 123456789`
4. Copy angka User ID

### Langkah 3: Setup gateway

```bash
hermes gateway setup
# Pilih: Telegram
# Paste: Bot Token
# Paste: User ID kamu (bisa multiple, pisah koma)
```

Atau setup manual:

```bash
cat >> ~/.hermes/.env << 'EOF'
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ
TELEGRAM_ALLOWED_USERS=123456789
EOF
```

### Langkah 4: Start gateway

```bash
hermes gateway start
```

### Langkah 5: Cek status

```bash
hermes gateway status
```

Output yang diharapkan:

```text
active (running)
```

### Langkah 6: Test di Telegram

- Buka bot kamu
- Kirim: `/help`
- Bot harus membalas daftar perintah

---

## 4. Update Hermes (Opsional, Disarankan)

```bash
hermes update
```

Kalau ada warning `plugin solstice: No module named httpx`:

```bash
hermes pm install httpx
hermes gateway restart
```

---

## 5. Membuat 9 Skill Custom

Semua skill di bawah ini ditulis menggunakan perintah `cat >`. Salin perintah untuk skill yang diperlukan ke terminal; tidak perlu membuat file secara manual.

### Skill 1: `cve-lookup`

```bash
mkdir -p ~/.hermes/skills/cve-lookup
cat > ~/.hermes/skills/cve-lookup/SKILL.md << 'SKILL_EOF'
---
name: cve-lookup
description: Use when searching for CVE information on software versions. Queries NVD, MITRE, Exploit-DB, and GitHub for public PoCs.
version: 1.0.0
metadata:
  hermes:
    tags: [security, cve, vulnerability]
---

# CVE Lookup

## Overview
Search for CVE information for a given software and version. Returns CVE IDs, CVSS scores, exploitation status, and patch information.

## When to Use
- Authorized security assessment
- Vulnerability research for a specific software version
- Bug bounty planning

## Procedure

### Step 1: Identify Software Version
Determine exact version (e.g., nginx/1.18.0).

### Step 2: Query NVD
https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=software+version

### Step 3: Query MITRE
https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=software

### Step 4: Check Exploit-DB
https://www.exploit-db.com/search?q=software

### Step 5: Check GitHub for PoCs
Search GitHub for CVE-ID PoC.

### Step 6: Summarize
Output format:
- CVE ID
- CVSS score
- Description
- Exploitation status (PoC public? in wild?)
- Fixed version

## Common Pitfalls
1. Outdated data - always cross-check multiple sources
2. False positives - verify CVE applies to exact version
3. EOL software - note if software is EOL

## Verification Checklist
- [ ] CVE applies to exact version
- [ ] CVSS score verified
- [ ] Patch version identified
- [ ] PoC status checked
SKILL_EOF
```

### Skill 2: `subdomain-enum`

```bash
mkdir -p ~/.hermes/skills/recon/subdomain-enum
cat > ~/.hermes/skills/recon/subdomain-enum/SKILL.md << 'SKILL_EOF'
---
name: subdomain-enum
description: Use when performing passive subdomain discovery on an authorized target. Enumerates subdomains via subfinder and validates live hosts with httpx.
version: 1.0.0
metadata:
  hermes:
    tags: [recon, subdomain, enumeration]
---

# Subdomain Enumeration

## Overview
Passive subdomain discovery using subfinder, followed by live host validation with httpx.

## When to Use
- Authorized bug bounty program with subdomain scope
- Security assessment where passive recon is permitted
- Do not use for unauthorized scanning

## Procedure

### Step 1: Passive Enumeration
subfinder -d target -silent -rate-limit 10 -o recon/target/subdomains.txt

### Step 2: Live Host Validation
httpx -l recon/target/subdomains.txt -silent -rate-limit 50 -o recon/target/live.txt

### Step 3: Technology Fingerprinting
httpx -l recon/target/live.txt -tech-detect -silent -o recon/target/tech.json

### Step 4: Save Results
mkdir -p recon/target
mv subdomains.txt live.txt tech.json recon/target/

## Common Pitfalls
1. Exceeding rate limits - always start with 10 req/s
2. Scanning out-of-scope domains - verify against scope
3. Forgetting to save results

## Verification Checklist
- [ ] All subdomains verified against scope
- [ ] Rate limits respected
- [ ] Results saved to target-specific directory
SKILL_EOF
```

### Skill 3: `sql-injection`

```bash
mkdir -p ~/.hermes/skills/vuln/sql-injection
cat > ~/.hermes/skills/vuln/sql-injection/SKILL.md << 'SKILL_EOF'
---
name: sql-injection
description: Use when testing for SQL injection vulnerabilities on authorized targets. Covers error-based, boolean-blind, time-blind, and UNION-based techniques.
version: 1.0.0
metadata:
  hermes:
    tags: [sqli, injection, database, owasp-a03]
---

# SQL Injection Detection

## Overview
SQL Injection occurs when user input is concatenated into SQL queries without proper sanitization.

## When to Use
- Authorized penetration test with web application scope
- Bug bounty program that includes SQLi in scope
- Do not use for unauthorized testing

## Detection Points
- Login forms (username, password)
- Search params (q, search, filter)
- URL paths (/user/123, /product/abc)
- Cookies (session, user_id)
- HTTP headers (User-Agent, Referer)

## Procedure

### Step 1: Error-Based Detection
Inject single quote.
Look for SQL error messages:
- MySQL: You have an error in your SQL syntax
- PostgreSQL: syntax error at or near
- MSSQL: Unclosed quotation mark

### Step 2: Boolean-Blind Detection
True: `quote AND 1=1`
False: `quote AND 1=2`
Compare responses.

### Step 3: Time-Blind Detection
MySQL: `quote AND SLEEP(5)`
PostgreSQL: `quote; SELECT pg_sleep(5)`
MSSQL: `quote; WAITFOR DELAY 0:0:5`

### Step 4: UNION-Based Detection
Determine column count with `ORDER BY`.
Extract data with `UNION SELECT`.

### Step 5: Automated Confirmation
sqlmap -u "https://target.com/search?q=test" --batch --level=3 --risk=2 --delay=1
Note: `--delay=1` means 1 second between requests.

## Severity Assessment
- Data extraction (PII): High or Critical
- Authentication bypass: Critical
- RCE via xp_cmdshell: Critical
- Blind SQLi only: Medium or High

## Remediation
- Use parameterized queries / prepared statements
- Input validation + whitelist
- Least-privilege database accounts
- WAF as defense-in-depth

## References
- OWASP A03:2025 Injection
- HackerOne Top 10
SKILL_EOF
```

### Skill 4: `idor-check`

```bash
mkdir -p ~/.hermes/skills/vuln/idor-check
cat > ~/.hermes/skills/vuln/idor-check/SKILL.md << 'SKILL_EOF'
---
name: idor-check
description: Use when testing for Insecure Direct Object Reference on authorized targets. Covers horizontal and vertical privilege escalation via ID manipulation.
version: 1.0.0
metadata:
  hermes:
    tags: [idor, bola, access-control, owasp-a01]
---

# IDOR Detection

## Overview
IDOR occurs when an application exposes internal object references without proper authorization checks, allowing users to access resources belonging to others.

## When to Use
- Authorized bug bounty program with access control in scope
- Penetration test of multi-user web application
- Do not use for testing with other users accounts

## Testing Points
- API endpoints: /api/user/123, /api/order/456
- Query params: user_id, doc
- Path segments: /invoice/INV-001
- Request body: user_id field
- Cookies: user id

## Procedure

### Step 1: Create Two Test Accounts
- Account A (attacker): your primary account
- Account B (victim): your secondary account
- Both must be yours

### Step 2: Capture Legitimate Request
Login as A, access resource owned by A.

### Step 3: Replace ID with B's ID
Request using B's ID; if response returns B data, IDOR confirmed.

### Step 4: Test Vertical Escalation
Login as regular user and access admin resource.

### Step 5: Test UUID vs Sequential
UUIDs are not automatically safe.

### Step 6: Test HTTP Methods
GET, POST, PUT methods are tested.
DELETE is NOT tested.

## Severity Assessment
- Horizontal (user to user): High
- Vertical (user to admin): Critical
- PII exposure: High or Critical
- Read-only IDOR: Medium or High
- Write or Delete IDOR: Critical

## Remediation
- Implement server-side authorization checks
- Use indirect references (session-mapped IDs)
- Deny by default
- Log all access attempts

## References
- OWASP A01:2025 Broken Access Control
- HackerOne Top 10 #3
SKILL_EOF
```

### Skill 5: `xss-basic`

```bash
mkdir -p ~/.hermes/skills/vuln/xss-basic
cat > ~/.hermes/skills/vuln/xss-basic/SKILL.md << 'SKILL_EOF'
---
name: xss-basic
description: Use when testing for Cross-Site Scripting on authorized targets. Covers reflected, stored, and DOM-based XSS with non-destructive PoC.
version: 1.0.0
metadata:
  hermes:
    tags: [xss, injection, client-side, owasp-a03]
---

# XSS Detection

## Overview
XSS occurs when user input is rendered in HTML without proper encoding, allowing attackers to execute JavaScript in victims browsers.

## When to Use
- Authorized web application penetration test
- Bug bounty program that includes XSS in scope
- Do not use for unauthorized testing

## Types of XSS
- Reflected: payload in request, reflected in response
- Stored: payload saved in DB, rendered to others
- DOM-based: client-side JS writes input to DOM

## Procedure

### Step 1: Injection Marker
Inject: `xss test marker`
Observe response.

### Step 2: Test Common Payloads
- Basic: `script alert`
- IMG: `img src=x onerror=alert`
- SVG: `svg onload=alert`
- Attribute break-out: `quote script alert`
- JavaScript URI: `javascript alert`

### Step 3: Context Analysis
HTML context, attribute context, JavaScript context, URL context.

### Step 4: DOM-Based Detection
Check sinks: `innerHTML`, `outerHTML`, `document.write`, `eval`, `setTimeout`, `setInterval`, `Function`.

### Step 5: Non-Destructive PoC
Use only: `alert` or `console.log`.
NEVER use: cookie stealing, keylogging, malicious redirects.

## Severity Assessment
- Stored XSS (admin panel): Critical
- Stored XSS (user): High
- Reflected XSS: Medium
- DOM XSS: Medium or High
- Self-XSS only: Low

## Remediation
- Output encoding
- Input validation
- Content Security Policy (CSP)
- HttpOnly cookies
- Modern frameworks with proper escaping

## References
- OWASP A03:2025 Injection
- HackerOne Top 10 #1
SKILL_EOF
```

### Skill 6: `ssrf-detect`

```bash
mkdir -p ~/.hermes/skills/vuln/ssrf-detect
cat > ~/.hermes/skills/vuln/ssrf-detect/SKILL.md << 'SKILL_EOF'
---
name: ssrf-detect
description: Use when testing for Server-Side Request Forgery on authorized targets. Covers internal service access, cloud metadata reachability, and blind SSRF callbacks.
version: 1.0.0
metadata:
  hermes:
    tags: [ssrf, injection, access-control, owasp-a01]
---

# SSRF Detection

## Overview
SSRF occurs when an application fetches a URL without validating it, allowing attackers to reach internal services or cloud metadata.

## When to Use
- Authorized bug bounty with SSRF in scope
- Web app that fetches URLs (webhooks, PDF gen, image import)
- Do not use for unauthorized testing

## Testing Points
- URL params: url, path, dest, redirect
- Webhooks: POST webhook with url
- PDF generation: POST pdf with html_url
- Image import: POST import with image_url
- File upload via URL: POST upload with source
- API integrations: OAuth callbacks, SSO endpoints

## Procedure

### Step 1: Basic Internal Access
- Localhost: `http://127.0.0.1:80`
- Local network: `http://192.168.1.1`
- AWS metadata: `http://169.254.169.254/latest/meta-data/`
- GCP metadata: `http://metadata.google.internal/computeMetadata/v1/`
- Azure metadata: `http://169.254.169.254/metadata/instance`

### Step 2: Bypass Filters
- Decimal IP: `http://2130706433`
- Hex IP: `http://0x7f000001`
- Octal IP: `http://0177.0.0.1`
- DNS rebinding: `make-127-0-0-1-rebind.example.com`
- Redirect: `attacker.com/redirect?url=http://169.254.169.254/`
- IPv6: `http://[::1]`

### Step 3: Blind SSRF Detection
Use interactsh or Burp Collaborator.

### Step 4: Verify Impact
If cloud metadata accessible, check AWS credentials.

## Severity Assessment
- Cloud metadata access (IAM creds): Critical
- Internal service access: High
- Port scanning internal: Medium or High
- Blind SSRF with DNS callback: Low or Medium
- No callback: Informational

## Remediation
- Whitelist allowed domains
- Block private IP ranges
- Disable unused URL schemes
- Use IMDSv2 for AWS
- Network segmentation

## References
- OWASP A01:2025 Broken Access Control
- HackerOne SSRF reports
SKILL_EOF
```

### Skill 7: `jwt-attacks`

```bash
mkdir -p ~/.hermes/skills/vuln/jwt-attacks
cat > ~/.hermes/skills/vuln/jwt-attacks/SKILL.md << 'SKILL_EOF'
---
name: jwt-attacks
description: Use when testing JWT-based authentication on authorized targets. Covers algorithm confusion, none algorithm, weak secrets, and claim tampering.
version: 1.0.0
metadata:
  hermes:
    tags: [jwt, authentication, token, owasp-a07]
---

# JWT Attacks

## Overview
JWT implementation flaws can lead to authentication bypass, privilege escalation, or account takeover.

## When to Use
- Authorized penetration test of JWT-based auth
- Bug bounty program with authentication in scope
- Do not use for unauthorized testing

## Attack Vectors

### 1. Algorithm Confusion (RS256 to HS256)
Server uses RS256 but accepts HS256.

### 2. None Algorithm
Server accepts `alg: none` without signature.

### 3. Weak Secret Brute-Force
Use `hashcat`, `john`, or `jwt_tool`.

### 4. Claim Tampering
Modify claims without re-signing or test if server validates signature.

## Procedure

### Step 1: Decode JWT
Header: `echo JWT | cut -d. -f1 | base64 -d`
Payload: `echo JWT | cut -d. -f2 | base64 -d`
Check `alg`, `kid`, `jku`, `x5u`, `sub`, `role`, `user_id`.

### Step 2: Test None Algorithm
python3 jwt_tool.py JWT -X a

### Step 3: Test Algorithm Confusion
Only if server uses RS256.

### Step 4: Test Weak Secret
Common weak secrets: `secret`, `password`, `123456`, `jwt_secret`.

### Step 5: Test Claim Tampering
Modify `sub`, `role`, `user_id`.

### Step 6: Test KID Injection
`kid` may contain traversal or SQL injection payloads.

### Step 7: Test JKU or X5U Injection
`jku` or `x5u` can point to attacker-controlled sources.

## Severity Assessment
- Auth bypass: Critical
- Privilege escalation: Critical
- Account takeover: Critical
- Weak secret (crackable): High
- None algorithm: Critical

## Remediation
- Use strong random secrets (min 256-bit)
- Explicitly specify allowed algorithms
- Validate all claims
- Do not trust `kid`, `jku`, `x5u` blindly
- Use established libraries

## References
- OWASP A07:2025 Authentication Failures
- jwt_tool: https://github.com/ticarpi/jwt_tool
SKILL_EOF
```

### Skill 8: `deserialization`

```bash
mkdir -p ~/.hermes/skills/vuln/deserialization
cat > ~/.hermes/skills/vuln/deserialization/SKILL.md << 'SKILL_EOF'
---
name: deserialization
description: Use when testing for insecure deserialization on authorized targets. Covers gadget chains leading to RCE or data tampering.
version: 1.0.0
metadata:
  hermes:
    tags: [deserialization, rce, integrity, owasp-a08]
---

# Insecure Deserialization

## Overview
Deserialization flaws allow attackers to manipulate serialized objects, potentially achieving RCE or data tampering.

## When to Use
- Authorized penetration test
- Application using serialized data (cookies, API, cache)
- Do not use for unauthorized testing

## Detection Points
- Cookies: Base64-encoded binary data
- API params: data, token, state
- Cache keys: Redis, Memcached
- Message queues: RabbitMQ, Kafka
- File uploads: .ser, .pickle, .dat
- Session tokens: custom session format

## Common Formats
- Java: `ObjectInputStream`, magic `AC ED 00 05`
- PHP: `serialize`, magic `O:` or `a:`
- Python: `pickle`, magic `80 04`
- .NET: `BinaryFormatter`
- Ruby: `Marshal`

## Procedure

### Step 1: Identify Serialized Data
Decode base64 and inspect magic bytes.

### Step 2: Determine Format
Java, PHP, Python, .NET, Ruby.

### Step 3: Generate Payload (Non-Destructive Only)
Use tool like `ysoserial`, `PHPGGC`, or `pickle-payloads`.

### Step 4: Test Payload
Send serialized payload to the target endpoint.

### Step 5: Verify Impact
Check callback server or logs.
NEVER run destructive commands.

## Severity Assessment
- RCE: Critical
- Data tampering: High
- DoS: Medium
- Info disclosure: Medium

## Remediation
- Avoid deserializing untrusted data
- Use safe formats (JSON, Protobuf)
- Integrity checks (HMAC)
- Whitelist allowed classes
- Isolate in sandbox

## References
- OWASP A08:2025 Software/Data Integrity Failures
- ysoserial: https://github.com/frohoff/ysoserial
- PHPGGC: https://github.com/ambionics/phpggc
SKILL_EOF
```

### Skill 9: `vuln-report-format`

```bash
mkdir -p ~/.hermes/skills/report/vuln-report-format
cat > ~/.hermes/skills/report/vuln-report-format/SKILL.md << 'SKILL_EOF'
---
name: vuln-report-format
description: Use when generating vulnerability reports. Produces standardized Markdown reports with reproduction steps and impact assessment.
version: 1.0.0
metadata:
  hermes:
    tags: [reporting, documentation]
---

# Vulnerability Report Format

## Overview
Standardized format for vulnerability reports.

## When to Use
- After confirming a vulnerability
- Before submitting to bug bounty platform
- For internal security documentation

## Template

# Vulnerability Type in Endpoint

## Metadata
- Target: https://target.com
- Endpoint: /api/user/123
- Date: 2026-01-15
- Reporter: Your Name
- Program: Bug Bounty Program Name

## Severity
- CVSS Score: X.X
- CVSS Vector: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

## Summary
Brief description (2–3 sentences).

## Steps to Reproduce
1. Login as user A
2. Navigate to /api/user/123
3. Replace 123 with 456
4. Observe response contains user B data

## Impact
Attacker can access other users PII.

## Evidence
Request and response examples.

## Reproduction Rate
100% (tested 3 times)

## Remediation
Implement server-side authorization checks.

## References
- OWASP A01:2025 Broken Access Control
- CWE-639

## Output Location
reports/target/vuln-type-timestamp.md
reports/target/SUMMARY.md
reports/target/evidence/

## Checklist
- [ ] Severity justified with CVSS
- [ ] Steps reproducible (tested 3x)
- [ ] Impact clearly stated
- [ ] Evidence attached
- [ ] Remediation provided
- [ ] No destructive actions documented
- [ ] No other users data in evidence
- [ ] Program rules followed

## References
- CVSS Calculator: https://www.first.org/cvss/calculator/3.1
- CWE Database: https://cwe.mitre.org/
SKILL_EOF
```

---

## 6. Reload dan Verifikasi Skill

```bash
hermes skills reload
hermes gateway restart

# Verifikasi semua skill terdaftar
hermes skills list | grep -E "cve|idor|sql|xss|ssrf|jwt|deserial|subdomain|vuln-report"
```

Output yang diharapkan:

```text
cve-lookup           enabled
deserialization      enabled
idor-check           enabled
jwt-attacks          enabled
sql-injection        enabled
ssrf-detect          enabled
subdomain-enum       enabled
vuln-report-format   enabled
xss-basic            enabled
```

---

## 7. Membuat `scope.yaml`

```bash
mkdir -p ~/hermes-cyber-kit/config
cat > ~/hermes-cyber-kit/config/scope.yaml << 'SCOPE_EOF'
program: "Authorized Bug Bounty Program"
authorization: "AUTHORIZED"
authorized_by: "Program Owner Name"
date: "2026-01-01"

targets:
  - "https://api.mercadolibre.com"
  - "https://api.mercadopago.com"

out_of_scope:
  - "*.mercadolibre.cn"
  - "*.internal.example.com"

allowed:
  reconnaissance: true
  endpoint_discovery: true
  vulnerability_testing: true
  poc_generation: true
  automated_scanning: false

restricted:
  destructive_actions: true
  denial_of_service: true
  data_exfiltration: true
  social_engineering: true
  physical_access: true
  brute_force: true
  mass_account_creation: true

rate_limits:
  nuclei: 10
  subfinder: 10
  httpx: 50
  ffuf: 20
  katana: 10
SCOPE_EOF
```

---

## 8. Pengujian di Telegram

Buka Telegram, cari bot kamu:

1. Kirim: `/reset`
2. Kirim: `/skills`
3. Test skill satu-satu

Gunakan skill:

- `cve-lookup` untuk cari CVE Apache 2.4.50
- `idor-check` untuk analisis endpoint di `scope.yaml`
- `subdomain-enum` untuk recon `example.com`
- `sql-injection` untuk analisis endpoint login
- `xss-basic` untuk test parameter `q`
- `ssrf-detect` untuk test parameter `url`

---

## 9. Backup ke GitHub (Opsional)

```bash
cd ~/hermes-cyber-kit

mkdir -p skills/{vuln,recon,report,cve-lookup}
cp -r ~/.hermes/skills/vuln/* skills/vuln/
cp -r ~/.hermes/skills/recon/* skills/recon/
cp -r ~/.hermes/skills/report/* skills/report/
cp -r ~/.hermes/skills/cve-lookup skills/cve-lookup/ 2>/dev/null

git init
git add .
git commit -m "Initial commit: Hermes Cyber Kit"
git remote add origin https://github.com/USERNAME/hermes-cyber-kit.git
git branch -M main
git push -u origin main
```

---

## 10. Troubleshooting

### Error: `hermes: command not found`

```bash
source ~/.bashrc
```

### Error: No model configured

```bash
hermes model
```

### Error: 401 Unauthorized

```bash
cat ~/.hermes/.env | grep OPENROUTER
```

Copy ulang API key dari https://openrouter.ai/keys.

### Error: HTTP 404 model unavailable for free

```bash
nano ~/.hermes/config.yaml
```

Ganti default ke:

```yaml
openrouter/free
```

### Error: YAML frontmatter parse error

Hapus tanda titik dua di `description` pada file skill.

### Error: Gateway not running

```bash
hermes gateway restart
```

### Error: Telegram bot tidak balas

```bash
hermes gateway status
hermes logs -n 30
```

### Cek kuota OpenRouter

```bash
curl -s https://openrouter.ai/api/v1/key -H "Authorization: Bearer $(grep OPENROUTER ~/.hermes/.env | cut -d= -f2)"
```

Kalau kuota habis:

1. Tunggu reset harian (00:00 UTC)
2. Top up $10 di https://openrouter.ai/settings/credits
3. Atau ganti ke provider lain (Ollama, Groq, Gemini)

---

## 11. Tautan Penting

- OpenRouter Keys: https://openrouter.ai/keys
- OpenRouter Credits: https://openrouter.ai/settings/credits
- OpenRouter Models: https://openrouter.ai/models
- Hermes Docs: https://hermes-agent.nousresearch.com/docs/
- Hermes GitHub: https://github.com/NousResearch/hermes-agent
- PortSwigger Lab: https://portswigger.net/web-security
- OWASP Juice Shop: https://owasp.org/www-project-juice-shop/
- HackerOne: https://hackerone.com
- Bugcrowd: https://bugcrowd.com
- YesWeHack: https://yeswehack.com
- CVSS Calculator: https://www.first.org/cvss/calculator/3.1
- CWE Database: https://cwe.mitre.org/

---

## 12. Catatan Legal

### Hanya Uji Target yang Sah

- Bug bounty program resmi (HackerOne, Bugcrowd, YesWeHack)
- Kontrak penetration test tertulis
- Sistem milik sendiri
- Lab latihan (PortSwigger, Juice Shop, DVWA, HackTheBox)

### Aktivitas yang Dilarang

- Target tanpa izin (ILEGAL — UU ITE Pasal 30)
- DoS / serangan destruktif
- Ekstraksi data orang lain
- Automated scanning tanpa izin
- Social engineering
- Brute force testing
- Mass account creation

### Konsekuensi Pelanggaran

- Ban permanen dari platform bug bounty
- Tuntutan hukum pidana
- Denda miliaran rupiah
- Reputasi hancur

---

## 13. Ringkasan Perintah Cepat

```bash
# Install Hermes
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc

# Setup model
hermes model

# Setup Telegram
hermes gateway setup

# Start gateway
hermes gateway start

# Cek status
hermes gateway status

# Update
hermes update

# Cek skill
hermes skills list

# Restart
hermes gateway restart

# Test chat
hermes chat -q "Reply with exactly one word: pong"

# Reload skill
hermes skills reload

# Cek kuota OpenRouter
curl -s https://openrouter.ai/api/v1/key -H "Authorization: Bearer $(grep OPENROUTER ~/.hermes/.env | cut -d= -f2)"
```

---

## Ringkasan

- Total skill: 9 custom + 53 builtin = 62 skill
- Model: `openrouter/free` (gratis) atau paid setelah top-up $10
- Akses: Terminal Linux + Telegram Bot
- Waktu setup: 30–60 menit
- Status: PRODUCTION READY

---

## Catatan

Dokumen ini dibuat untuk keperluan edukasi, keamanan yang sah, dan penggunaan yang sesuai dengan peraturan serta program yang diizinkan. Gunakan secara bertanggung jawab dan hanya pada target yang Anda miliki izin resmi untuk diuji.

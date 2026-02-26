# FocusFire v2.1.0 — Alpha Release Burp Suite Extension

Advanced automated penetration testing extension for Burp Suite Professional/Community. 18 passive detectors, 8 active scanners, deserialization engine, BAC tester, visual spider map, AI-assisted analysis, and multi-format export.

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Navigation](#navigation)
- [Passive Detectors](#passive-detectors)
- [Active Scanners](#active-scanners)
- [Scan Tools](#scan-tools)
- [Crawler & Spider Map](#crawler--spider-map)
- [Export & Reporting](#export--reporting)
- [Configuration](#configuration)
- [Building from Source](#building-from-source)

---

## Requirements

- Java 17+
- Burp Suite Professional or Community Edition
- Maven 3.6+ (for building from source)

## Installation

1. Download `focusfire-1.0.0.jar` from Releases (or [build from source](#building-from-source))
2. Open Burp Suite
3. Go to **Extensions** → **Installed** → **Add**
4. Extension Type: **Java**
5. Select `focusfire-1.0.0.jar`
6. Click **Next** — FocusFire will appear as a new tab

## Quick Start

1. **Set your scope** — Go to the **Scope** section and add target domains (e.g. `example.com`, `*.example.com`). Only in-scope traffic is analyzed.
2. **Browse** — Use Burp's browser or proxy your traffic. Passive detectors run automatically on every request/response.
3. **Check the Dashboard** — The **Dashboard** panel shows live stats: JWTs found, API keys, cookies, parameters, and findings by severity.
4. **Review Findings** — The **Findings** panel shows all discovered vulnerabilities with full request/response data, filtering, and sorting.
5. **Run Active Scans** — Active scanners test parameters automatically when traffic flows through. Additional tools (SQLMap, Fuzzer, AuthZ, SSRF Scanner, Deserializer, BAC Tester) are available under **Scan Tools**.
6. **Crawl** — Use the **Crawler** to discover endpoints from a starting URL with an optional wordlist.
7. **Export** — Export findings as JSON, CSV, or a self-contained HTML report.

---

## Navigation

FocusFire uses a grouped sidebar layout:

| Section | Panels | Purpose |
|---------|--------|---------|
| **Overview** | Dashboard | Live stats, severity breakdown, recent findings |
| **Analysis** | Findings, AI Assistant | Review all findings with filtering; AI-powered analysis |
| **Scan Tools** | SQLMap, SSRF Scanner, AuthZ Test, Fuzzer, TLS Scanner, Deserializer, BAC Tester | Manual active testing tools |
| **Recon** | Crawler, Scope | Spider map, endpoint discovery, scope management |
| **Output** | Export | JSON, CSV, HTML report export with preview |
| **Info** | Changelog, Help | Version history, documentation |

---

## Passive Detectors

Passive detectors analyze traffic flowing through Burp without sending any additional requests. All respect scope filtering and report findings with severity and confidence levels.

### 1. SQL Injection (SQLi)

**Finding Type:** `SQLI` | **Severity:** HIGH | **Confidence:** Firm

Detects SQL error messages leaked in responses across 6 database engines:

| Database | Patterns Detected |
|----------|-------------------|
| MySQL | `You have an error in your SQL syntax`, `mysql_fetch`, `mysql_num_rows` |
| PostgreSQL | `PSQLException`, `ERROR: syntax error at or near` |
| Oracle | `ORA-`, `oracle.jdbc`, `quoted string not properly terminated` |
| MSSQL | `Unclosed quotation mark`, `Microsoft OLE DB`, `ODBC SQL Server Driver` |
| SQLite | `SQLite3::`, `SQLITE_ERROR`, `unrecognized token` |
| Java/PHP | `java.sql.SQLException`, `pg_query`, `PDOException` |

---

### 2. Cross-Site Scripting (XSS)

**Finding Types:** `XSS_REFLECTION`, `XSS_PATTERN` | **Severity:** HIGH | **Confidence:** Firm

Only runs on `text/html` responses to avoid false positives on JSON/XML APIs.

- **Reflection detection:** Extracts parameter values from request URL, checks if dangerous characters (`<`, `>`, `"`, `'`, `javascript:`) appear unencoded in the response
- **Sink pattern detection:** `javascript:` URIs, event handlers, `<img onerror>`, `<svg><script>`, `data:` URIs

---

### 3. Server-Side Template Injection (SSTI)

**Finding Type:** `SSTI` | **Severity:** HIGH | **Confidence:** Firm

Detects template engine exceptions: Freemarker, Thymeleaf, Spring SpEL, Velocity, Pebble, Jinja2, Twig, Smarty, Handlebars, Groovy.

---

### 4. Command Injection

**Finding Type:** `CMD_INJECTION` | **Severity:** HIGH | **Confidence:** Firm

Detects OS command output and error messages for Linux and Windows.

---

### 5. XML External Entity (XXE)

**Finding Type:** `XXE` | **Severity:** HIGH | **Confidence:** Firm

Detects XXE evidence: `/etc/passwd` content, `boot.ini`, DOCTYPE errors, SAX/XML parser exceptions.

---

### 6. Insecure Deserialization

**Finding Type:** `DESERIALIZATION` | **Severity:** HIGH | **Confidence:** Firm

Detects deserialization indicators: Java (`rO0AB`, `InvalidClassException`), PHP (`unserialize()`), Python (Pickle), .NET (`BinaryFormatter`), YAML (SnakeYAML).

---

### 7. NoSQL Injection

**Finding Type:** `NOSQL` | **Severity:** HIGH | **Confidence:** Firm

Detects MongoDB, CouchDB, Elasticsearch error patterns and `$where` injection indicators.

---

### 8. Path Traversal / LFI

**Finding Type:** `PATH_TRAVERSAL` | **Severity:** HIGH | **Confidence:** Firm

Detects file content from directory traversal: `/etc/passwd`, `boot.ini`, PHP include errors, Java `FileNotFoundException`.

---

### 9. Credentials & Secrets

**Finding Types:** `JWT`, `API_KEY`, `COOKIE` | **Severity:** HIGH

- **JWT detection:** Validated by Base64-decoding header and confirming `"alg"` key
- **API keys:** AWS, GitHub, OpenAI, Slack, Google
- **Cookie tracking:** Captures all cookies for reporting

---

### 10. WAF Detection

**Finding Type:** `WAF` | **Severity:** INFO

Identifies 25+ WAF products (Cloudflare, AWS WAF, Akamai, Imperva, F5, ModSecurity, etc.) via response headers. Provides bypass hints.

---

### 11–18. Additional Passive Detectors

| # | Finding Type | Description |
|---|---|---|
| 11 | `VIEWSTATE_*` | ASP.NET ViewState protection analysis |
| 12 | `VERBOSE_ERROR` / `MASS_ASSIGNMENT` / etc. | OWASP API Top 10 checks |
| 13 | `MISSING_HEADER` | Security headers audit (HSTS, CSP, X-Content-Type-Options, X-Frame-Options) |
| 14 | `CSP_*` | CSP weakness analysis (unsafe-inline, CDN bypass for 15 CDNs) |
| 15 | `CORS` | CORS misconfiguration detection |
| 16 | `HOST_HEADER` | Host header injection (5 vectors) |
| 17 | `XFS` | Clickjacking / Cross-Frame Scripting with HTML PoC |
| 18 | `BROKEN_ACCESS` | Broken access control (admin panels, IDOR, actuators, sensitive files) |

---

## Active Scanners

Active scanners send additional requests to test for vulnerabilities.

| # | Scanner | Techniques |
|---|---------|-----------|
| 1 | **SQL Injection** | Error-based, Union-based, Boolean-based, Time-based |
| 2 | **XSS** | Reflected (7 payloads), DOM-based, context analysis |
| 3 | **Command Injection** | Output-based & time-based (Linux + Windows) |
| 4 | **Path Traversal** | Linux & Windows targets with bypass variants |
| 5 | **SSRF** | Localhost, cloud metadata, internal IPs, IPv6 |
| 6 | **Open Redirect** | 10+ bypass payloads with DOM-based detection |
| 7 | **CRLF / Header Injection** | 7 payload variants with canary header |
| 8 | **Concurrent Login** | Race condition testing on auth endpoints |

---

## Scan Tools

### SQLMap Integration

Manual SQL injection testing with Hunt Mode, Ghauri-inspired XOR-based blind injection, 6 WAF bypass tampers, header-based injection, adaptive time-based detection with statistical baselines.

### SSRF Scanner

Dedicated SSRF testing with cloud metadata probing (AWS/GCP/Azure/DO/Alibaba/Oracle/K8s), protocol smuggling (file/gopher/dict/ftp/jar), localhost bypass (26 payloads), OOB callback support.

### Authorization Testing (AuthZ)

Broken access control testing: BOLA/IDOR parameter manipulation, role-based access testing, response comparison between privilege levels.

### Fuzzer

Lightweight Intruder-style fuzzer with Sniper & Battering Ram modes, multi-threaded, match filters, sortable results.

### TLS Scanner

Certificate validity, weak protocol detection (SSLv3, TLS 1.0/1.1), cipher suite analysis, self-signed certificate detection.

### Deserialization Engine

Universal insecure deserialization payload generator covering 6 languages:

| Language | Chains |
|----------|--------|
| **Java** | URLDNS (native), CommonsCollections1/5/6, CommonsBeanutils1, JNDIExploit, DNSCallback |
| **.NET** | 25 gadgets: TypeConfuseDelegate, TextFormattingRunProperties, JsonNet, JavaScriptSerializer, PSObject, ViewState, DataSet, NetDataContractSerializer, and more |
| **PHP** | LaravelPOP, MonologRCE, GuzzleFnStream, WordPressPHPObject, GenericDestruct, GenericPHPGGC |
| **Python** | Pickle (os.system, subprocess, eval, exec), PyYAML exploits |
| **Ruby** | ERBTemplate, GemRequirement, GemInstaller, UniversalRCE |
| **Node.js** | node-serialize IIFE, js-yaml, cryo RCE |

Auto-detection of serialized data signatures in HTTP traffic. Multiple encoding options (Raw, Base64, URL-encoded).

### BAC Tester

Automated Broken Access Control & Privilege Escalation testing:

1. **Map** — BFS crawl as high-privilege user
2. **Replay** — Re-send all endpoints with low-privilege credentials
3. **No-auth** — Optional test with all auth headers stripped
4. **Report** — Color-coded results matrix (BYPASS/PARTIAL/BLOCKED/INCONCLUSIVE)

Context menu integration: right-click in Proxy/Repeater to send requests to Deserializer or BAC Tester.

---

## Crawler & Spider Map

Discovers endpoints by following links and brute-forcing paths. Visual force-directed spider map with status-colored nodes, search, tree layout, minimap, drag & zoom, and PNG export.

---

## Export & Reporting

| Format | Description |
|--------|-------------|
| **JSON** | Full export with metadata, severity counts, all findings/JWTs/API keys/cookies |
| **CSV** | Tabular format for spreadsheet analysis |
| **HTML** | Self-contained report with severity cards, expandable findings, inline CSS |

---

## Configuration

### Scope

Add target domains in the **Scope** panel. Supports exact domain, wildcard subdomain (`*.example.com`), and path prefix matching.

### Findings Panel

Filter by text/severity/confidence, column sorting, split orientation toggle, syntax-highlighted request/response viewer, finding comparison, keyboard shortcuts (`Ctrl+D` delete, `Ctrl+R` send to Repeater).

---

## Building from Source

```bash
git clone https://github.com/AK-VAULTCODE/FocusFire.git
cd FocusFire

# Build (requires Java 17+ and Maven 3.6+)
mvn clean package

# Output JAR
# target/focusfire-1.0.0.jar
```

The build uses `maven-shade-plugin` to bundle all dependencies into a single JAR compatible with Burp Suite's classloader.

---

## Author

**VaultCode Pte Ltd**

Special thanks to [@worldtreeboy](https://github.com/worldtreeboy) for contributing!

## License

MIT License


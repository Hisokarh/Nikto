# Lab Report — Vulnerability Scanning with Nikto

Date: 2025-12-25
Author: Wongani Kasawala Nkosi (Hisorkah on Github)

## Objective
Use Nikto to scan a vulnerable web application and report findings, evidence (screenshots), and remediation recommendations.

## Lab environment
- Scanner: Kali Linux (fully updated)
- Target: Metasploitable2 (IP 192.168.56.101) — OR DVWA running in Docker (http://172.17.0.2)
- Network: Host-only / isolated lab network

## Setup steps
1. Start the vulnerable VM or container.
   - Example (Metasploitable2 in VirtualBox): start VM and note its IP.
   - Example (DVWA via Docker):
     - docker run --rm -d --name dvwa -p 80:80 vulnerables/web-dvwa

2. On Kali, install Nikto:
   - sudo apt update
   - sudo apt install -y nikto

3. Verify Nikto:
   - nikto -Version

## Nikto scans performed (commands + explanation)
- Basic scan (HTTP):
  - nikto -h http://192.168.56.101
  - Explanation: Default checks against host; verbose terminal output.

- Scan specific port:
  - nikto -h 192.168.56.101 -p 80

- Save output (text):
  - nikto -h 192.168.56.101 -o nikto-output.txt

- Save output (HTML):
  - nikto -h 192.168.56.101 -o nikto-output.html -Format htm

- Tuning (reduce checks or focus on particular classes):
  - nikto -h 192.168.56.101 -Tuning 9
  - (Tuning values: 0=all, 1=interesting, 2, ... 9=headers/plugins; see nikto docs)

- Use SSL if target is HTTPS:
  - nikto -h https://192.168.56.101 -ssl

- Example full command (verbose + save):
  - nikto -h http://192.168.56.101 -p 80 -output scans/nikto-192.168.56.101.txt -Format txt -Display V

## Example sample output (representative lines)
(Note: actual output will vary)
- + Server: Apache/2.2.8 (Ubuntu) DAV/2
- + Retrieved x-powered-by header: PHP/5.3.2-1ubuntu4.12
- + /admin/: admin directory found
- + /phpinfo.php: 'phpinfo()' found - displays sensitive information
- + Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, PUT
- + robots.txt found: /robots.txt disallows sensitive paths
- + OSVDB-3092: Multiple versions of server and scripts have known vulnerabilities

## Screenshots captured
Place screenshots in `screenshots/`:
- screenshots/Nikto.png — Nikto running in terminal
- screenshots/Nikto.png — Important output highlighted
- screenshots/vulnerable-page-phpinfo.png — evidence of phpinfo or admin page

## Findings and observations (examples)
1. Directory indexing or exposed admin directory
   - Evidence: /admin/ returned 200 and directory listing
   - Risk: Information disclosure, potential entry point
   - Remediation: Remove directory listings (Apache: `Options -Indexes`), restrict access via auth or IP

2. phpinfo() page present
   - Evidence: /phpinfo.php returned full PHP config including env variables
   - Risk: Credentials/environment leakage
   - Remediation: Remove phpinfo.php from production and restrict to dev machines

3. Missing security headers (X-Frame-Options, X-XSS-Protection)
   - Evidence: Nikto reported missing headers
   - Risk: Clickjacking, XSS mitigation missing
   - Remediation: Add recommended headers (e.g., `X-Frame-Options: DENY`)

4. Outdated server components
   - Evidence: Server banner shows old Apache/PHP versions
   - Risk: Known CVEs may apply
   - Remediation: Patch/upgrade server packages and remove version banners

## Remediation checklist (summary)
- Remove development/test pages (phpinfo, example scripts).
- Disable directory listing.
- Harden HTTP headers (CSP, X-Frame-Options, X-XSS-Protection, HSTS).
- Limit allowed HTTP methods; disable PUT/DELETE if unused.
- Upgrade server software; apply OS security patches.
- Use firewall / network segmentation to restrict access.

## Conclusion
Nikto quickly enumerated common web server misconfigurations and known issues. The output is noisy but valuable as a reconnaissance step; follow up with targeted testing (e.g., attempt exploit verification in a controlled environment) and patching.

## Artifacts
- Nikto output files (txt/html) placed in `scans/`
- Screenshots in `screenshots/`

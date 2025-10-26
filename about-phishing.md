**Email phishing analysis** is one of the areas of Security Operations I really enjoy performing. From securely grabbing the suspicious email to carefully confirming its legitimacy, every step is crucial. It’s about piecing together clues like headers, IP addresses, URLs, attachments, and the tiniest inconsistencies, to figure out if someone is trying to trick a user or compromise a system.  

Phishing is when attackers pretend to be someone you trust to steal sensitive information like passwords, credit card numbers, or personal data.  

These attacks often arrive through **emails, text messages, or phone calls** that appear legitimate but are crafted to deceive users into taking harmful actions.

### 💡 Common Types of Phishing
- **Vishing** – Voice phishing conducted over phone calls.  
- **Spear Phishing** – Targeted attacks aimed at specific individuals or organizations.  
- **Whaling** – High-profile phishing targeting executives or senior management.  

These attackers are clever 🎭 . They use psychological manipulation to make their messages believable by:

- Creating a **sense of urgency** (e.g., “Your account will be locked!”)  
- Pretend to be an **authority figure** (e.g., banks, IT admins, or CEOs)  
- Try to **scare you** or make you **panic**  
- Make you feel like you’ll **miss out on something** if you don’t act

The dangers of phishing for a company are real — data breaches, financial loss, and reputational damage. That’s why it’s crucial for SOC analysts to detect attacks quickly, investigate thoroughly, and take steps to protect users and the organization.

For organizations, detecting and analyzing phishing attempts early is crucial.  
That’s where **Security Operations Center (SOC) Analysts** play a vital role — investigating suspicious messages, identifying Indicators of Compromise (IOCs), and implementing defense strategies to prevent further attacks.

Let's dive a bit deeper by going over Phishing Analysis, which is the focus of this repository.

**Phishing Analysis** is the process of investigating suspicious emails to determine whether they are malicious.  
This involves examining:
- **Email headers** – to trace the origin and identify spoofing  
- **Links and domains** – to detect redirects or fake websites  
- **Attachments** – to check for malware or payloads  
- **Email content and HTML** – to find signs of deception, urgency, or brand impersonation

## 🛠️ Essential Tools for Phishing Analysis

> Use multiple tools — each adds a layer of confidence.

- **Email Header Analyzer (EML Analyzer / MXToolbox)** — parse headers, trace hops, view SPF/DKIM/DMARC.  
- **VirusTotal** — scan URLs, domains, IPs and file hashes against many engines.  
- **URLScan.io** — safely render and inspect web landing pages and redirect chains.  
- **Any.Run / Hybrid Analysis** — sandbox suspicious files or URL behavior dynamically.  
- **CyberChef / Base64 Decoder** — decode base64/hex/obfuscated strings to reveal hidden payloads.  
- **OLETools (oledump, olevba)** — analyze Office documents/macros for suspicious code.  
- **Whois / IP lookup (ARIN, RIPE, DomainTools)** — verify domain registration and IP ownership.  
- **AbuseIPDB / Cisco Talos / IPinfo** — reputation and ASN context for IPs.  
- **Isolated VM / Sandbox environment** — safe place to open .eml or attachments without risking production.

## 🧪 Phishing Analysis Process

1. **Collect sample** — export email as `.eml`/`.msg`; preserve chain of custody.  
2. **Initial triage** — isolate file, do not click links or open attachments outside a sandbox.  
3. **Analyze headers** — extract first `Received:` hop, sender IP, Return-Path, SPF/DKIM/DMARC results.  
4. **Validate domain/IP** — `whois`, ASN, PTR (reverse DNS); compare to claimed organization.  
5. **Inspect body & links** — view raw HTML, hover links (or paste into VirusTotal/URLScan), decode obfuscated content (base64).  
6. **Scan artifacts** — submit URLs and attachment hashes to VirusTotal; detonate samples in sandbox if needed.  
7. **Extract IOCs** — domains, IPs, URLs, message-id, return-path, attachment hashes, subject lines.  
8. **Contain & remediate** — block IoCs at mail gateway/firewall, remove similar emails from inboxes, reset compromised accounts.  
9. **Report & document** — log findings, notify stakeholders, and feed IoCs to threat intelligence/blacklists.

## 📎 Suspicious Attachment Indicators

- Executables disguised as documents (`Invoice.pdf.exe`, `Resume.doc.scr`).  
- Password-protected / encrypted ZIPs containing executables or scripts.  
- Macro-enabled Office files (`.docm`, `.xlsm`) that request enabling macros.  
- Base64 / hex encoded HTML or script blocks inside the email body.  
- Unexpected themes (invoice, CV, urgent account notice) used to prompt action.

## 🔍 Example Phishing Scenario

Imagine receiving the following email:

- **Sender:** Outlook Support Team  
- **Message:** Your account has been flagged for unusual activity and temporarily disabled.  
- **Call to Action:** Click the button below to "re-verify" your account.  
- **Threat:** Your account will be permanently suspended within 24 hours if no action is taken.  

### ⚠️ Red Flags
1. **Urgency / Fear Tactic:** The email pressures the user to act quickly to avoid a negative consequence.  
2. **Unverified Sender:** The sender address may look official but could be a spoofed or similar-looking domain.  
3. **Suspicious Links:** Buttons or links may appear legitimate but redirect to a malicious site.  

As a SOC Analyst, it’s important not to jump to conclusions. Each suspicious email is treated like a potential security incident: we carefully collect evidence, analyze headers and content, and validate links before confirming whether it’s a phishing attempt.  
This systematic approach helps protect users while avoiding unnecessary disruptions.

# Phishing Analysis: Investigating a Phishing Email

<img width="972" height="697" alt="image" src="https://github.com/user-attachments/assets/e9edbdaf-54c1-4399-8aad-53d344695053" />


## Objective
In this Lab, I analyzed a real phishing email sample using manual investigation techniques. I extracted and reviewed email headers, URLs, and attachments, validated the sender's identity and identified indicators of compromise (IOCs).

## Lab Set Up
- [Download Email Sample (.eml)](./samples/bradesco_phishing_sample.eml)
- Tools Used:
  - [EML Analyzer](https://eml-analyzer.herokuapp.com/#/)
  - [MX Toolbox Email Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx)
  - [VirusTotal](https://www.virustotal.com/gui/home/upload) - for IP Reputation Check, URL & Domain Analysis
  - [Whois lookup](https://whois.domaintools.com/)
 
### SOC Incident: 

A user received an email from **BANCO DO BRADESCO LIVELO** claiming that their card has **92,990 points expiring today**. The message was sent from `banco.bradesco@atendimento.com.br`. The user reported the email as suspicious and the SOC team was alerted.

The SOC team preserved the original .eml file and began a phishing analysis to verify the sender’s legitimacy, trace the email origin, and identify any potential Indicators of Compromise (IoCs).

---
## Intitial Triage
1. **Containment** - Ensured the .eml file is stored in the evidence folder on isolated analysis VM. Attachments should not be opened in a non-sandboxed environment.
2. **Evidence collection** - Saved the original .eml, mail headers, screenshots of message preview (redact user PII), and any attachments.
3. **Preliminary judgement** - Marked incident as Potential Phishing and assign severity Medium → High depending on whether links are active or attachment present.

---
## Investigation Steps & Findings

1.**Analyzed the email (.eml)** using **EML Analyzer** to extract detailed header information.  

<img width="1312" height="316" alt="image" src="https://github.com/user-attachments/assets/cd5a382e-7d54-4feb-b03f-a4d441a596c9" />

*Confirmed the sender's email and domain*

2. **Confirmed the sender’s IP address**

<img width="2734" height="578" alt="image" src="https://github.com/user-attachments/assets/a6173cb1-7a6c-46da-ac75-557407bd7449" />

3. **Checked the sender’s IP reputation** using **VirusTotal**

<img width="1417" height="514" alt="image" src="https://github.com/user-attachments/assets/d60554be-8cc4-450f-a87a-4f5fe4c35835" />

*Only 1 out of 94 antivirus software flagged this as a malicious IP address. Further investigation would need to be performed*.

I then used **https://whois.domaintools.com/** to see who the iP belonged to

<img width="785" height="684" alt="image" src="https://github.com/user-attachments/assets/c5dea747-975a-4c5b-9709-ec68dc692ce9" />


## Key Observations:
- The IP does not belong to BANCO DO BRADESCO LIVELO or any legitimate bank network.
- The IP belongs to DigitalOcean, a cloud hosting provider. Attackers often use cloud hosting IPs to send phishing emails because they are easy to provision and can bypass some filters.
- Reverse DNS would likely resolve to a DigitalOcean server, not a bank domain.

Any email claiming to be from a bank but originating from a DigitalOcean IP is highly suspicious.

> ⚠️ These findings indicate that the IP used to send the email is potentially compromised or part of a malicious infrastructure, reinforcing the phishing nature of the email.

5. **Validated email authentication results** (SPF, DKIM, DMARC) in **EML Analyzer**
   
   <img width="1292" height="108" alt="image" src="https://github.com/user-attachments/assets/c9077c29-31a6-4191-afaa-5f51011ef719" />

## 🧾 Email Authentication Results (SPF, DKIM, DMARC)

<img width="1259" height="112" alt="image" src="https://github.com/user-attachments/assets/9ef0ca47-2a11-4812-a9ab-fed1e149e878" />

The email failed standard authentication checks, indicating spoofing or misconfiguration.  
Below are the results extracted from **EML Analyzer**:

| Protocol | Status / Result | Details |
|-----------|-----------------|----------|
| **SPF**   | ❌ `temperror`   | Sender IP **137.184.34.4** could not be validated due to DNS timeout when checking `ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06`. |
| **DKIM**  | ❌ `none`        | The message was **not signed**; no DKIM signature present in headers. |
| **DMARC** | ❌ `temperror`   | Policy lookup for `atendimento.com.br` failed due to DNS issues; **action=none** applied. |
| **CompAuth** | ❌ `fail (reason=001)` | Microsoft’s Composite Authentication failed, indicating the sender’s domain failed alignment checks. |

> ⚠️ **Conclusion:**  
> The message did not pass any authentication checks (SPF, DKIM, DMARC). Combined with the suspicious sending IP and content, this strongly supports classification as a **spoofed phishing email**.

6. **Inspected embedded URLs** from the email body using **VirusTotal** to detect potential redirects or phishing sites.

   <img width="1279" height="751" alt="image" src="https://github.com/user-attachments/assets/78e95820-198d-4399-9d77-8bc263c1998c" />
---
<img width="1404" height="304" alt="image" src="https://github.com/user-attachments/assets/d60b5643-ca6b-4508-aaa4-8a329b9d20fb" />

*This URL has not been flagged as maliciosu by any of the vendors, but this does not mean it is not malicious.

   > ⚠️ *Note:* Newly registered domains or recently created phishing URLs may not immediately appear as malicious in antivirus databases.  

7. **Used MXToolbox Email Header Analyzer** to cross-verify authentication results and trace the email’s delivery path.  

<img width="1405" height="696" alt="image" src="https://github.com/user-attachments/assets/afd220a2-69c7-4466-8bc9-a6ad27515b97" />

---
<img width="1367" height="299" alt="image" src="https://github.com/user-attachments/assets/27139927-ab15-4de5-ac7b-a541e26fe538" />

### 🔍 Observations:

1. **Suspicious Origin**: The first hop shows the email coming from `ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06` (IP 137.184.34.4), which is **not a legitimate Bradesco mail server**.
2. **Blacklist Hits**: Hops 2 and 5 are listed on a blacklist, signaling prior abuse associated with these nodes.
3. **Microsoft Relays**: The middle hops pass through Microsoft Office 365 servers with valid TLS encryption, which is normal, but does **not validate the original sender**.
4. **Rapid Hops**: The email progresses through hops quickly (seconds), which is typical for automated delivery but also means the email was likely injected into a legitimate SMTP relay after originating from a suspicious IP.

> ⚠️ **Conclusion**: The combination of a non-bank origin IP, blacklist entries, and rapid relay through Microsoft servers strongly indicates this email was **forged and potentially part of a phishing attack**.

---

## 📧 Email Content Analysis: Urgency, Branding, and Spoofing

### 1. Urgency / Social Engineering
- The email text creates a sense of urgency by stating that the user’s **“92,990 points [are] expiring today.”**
- This is a classic phishing tactic designed to **pressure the recipient into clicking a link or taking immediate action** without careful consideration.
- Urgency is often combined with fear of loss or limited-time offers to override normal caution.

### 2. Branding / Impersonation
- The email uses the legitimate **Bradesco Livelo logo, formatting, and brand colors**, making it appear authentic at first glance.
- Attackers replicate corporate branding to **gain trust and reduce suspicion**.

### 3. Spoofing / Visual Deception
- The sender address (`banco.bradesco@atendimento.com.br`) looks similar to the official bank domain but is **not legitimate**.
- Hyperlinks in the HTML body may display familiar URLs (e.g., “Bradesco Livelo portal”) but **redirect to a malicious site** (`http://alphaMountain.ai/...`).
- Base64 encoding in the HTML content is sometimes used to **hide malicious scripts or tracking pixels**.

### 4. Combined Effect
- The combination of urgency, realistic branding, and deceptive links increases the likelihood that the recipient will **click without verifying the source**.
- When cross-referenced with **header anomalies, failed SPF/DKIM/DMARC checks, and external IP injection**, these elements confirm the email is **malicious** and intended for phishing.
---

---

## Summary
- **Trace Email Origin:** Followed the delivery path of an email using headers, hop timelines, and IP analysis to determine its source.
- **Validate Domains and IPs:** Used tools like WHOIS, MXToolbox, and VirusTotal to check domain registration, IP ownership, and blacklist status.
- **Understand Spoofing Techniques:** Identified visual and technical signs of spoofing, including display name tricks, lookalike domains, and injected emails.
- **Detect Phishing Attempts:** Analyze email headers, body content, HTML formatting, URLs, and attachments to extract indicators of compromise (IOCs) and confirm malicious intent.

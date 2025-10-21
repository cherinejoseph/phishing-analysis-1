# Phishing Analysis: Investigating a Phishing Email

## Objective
In this Lab, I analyzed a real phishing email sample using manual investigation techniques. I extracted and reviewed email headers, URLs, and attachments, validated the sender's identity and identified indicators of compromise (IOCs).

## Lab Set Up
- [Download Email Sample (.eml)](./samples/bradesco_phishing_sample.eml)
- Tools Used:
  - [MX Toolbox Email Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx)
  - [EML Analyzer](https://eml-analyzer.herokuapp.com/#/)
  - [VirusTotal](https://www.virustotal.com/gui/home/upload) - for IP Reputation Check, URL & Domain Analysis
  - [Whois lookup](https://whois.domaintools.com/)
 
### SOC Incident: 

A user received an email from **BANCO DO BRADESCO LIVELO** claiming that their card has **92,990 points expiring today**. The message was sent from `banco.bradesco@atendimento.com.br`. The user reported the email as suspicious and the SOC team was alerted.

The SOC team preserved the original .eml file and began a phishing analysis to verify the sender’s legitimacy, trace the email origin, and identify any potential Indicators of Compromise (IoCs).


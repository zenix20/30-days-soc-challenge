# Day 11: Introduction to Phishing Analysis and Email Security

**Topic:** Phishing Analysis: email header forensics and authentication mechanisms
**Tools:** Email header analyzer (header pasted from a provided .eml file)

## Scenario

Given a ".eml" file containing a phishing email impersonating a company CEO ("Rajiv Mehra") requesting an urgent wire transfer, the task was to analyze the raw email headers to identify the true origin, the phishing infrastructure involved, the spam confidence scoring, and which authentication mechanism failed to protect against the spoof.

**Approach to opening the file:** opened the ".eml" file in Notepad to view the raw text, then copied the full header + body content into an email header analyzer tool for structured parsing (SPF/DKIM/DMARC breakdown, hop-by-hop relay info, and flagged header values) rather than trying to manually parse everything by eye.

## Questions & Approach

### 1. Which IP address originated the phishing email?

Found in multiple corroborating header fields:
```
X-Originating-IP: [98.177.68.12]
X-Sender-IP: 98.177.68.12
Received-SPF: ...client-ip=98.177.68.12...
```

**Answer: 98.177.68.12**

### 2. Which domain hosted the embedded phishing page?

The email body's "Review Payment Instructions" link pointed to a domain entirely unrelated to the spoofed sender domain, a strong phishing indicator on its own, since a legitimate CEO email would link back to the company's own domain, not a third-party site:
```
http://adventure-nicaragua.net/index.php?option=com_mailto&tmpl=component&link=...
```

**Answer: adventure-nicaragua.net**

### 3. What Spam Confidence Level (SCL) was assigned to the email?

```
X-MS-Exchange-Organization-SCL: 9
```

**Answer: 9** ,Microsoft's SCL scale tops out at 9, meaning this message was scored as maximum-confidence spam/malicious by the receiving mail system's filtering, despite technically passing SPF.

### 4. What suspicious sender domain was used in the impersonation attempt?

The `From`, `Reply-To`, and `Return-Path` headers all used the same domain, deliberately crafted to look vaguely legitimate/financial at a glance but not matching the real company ("Northstar Holdings") the sender claimed to represent:
```
From: Rajiv Mehra, CEO <rajiv.mehra@citiprepaid-salarysea-at.tk>
```

**Answer: citiprepaid-salarysea-at.tk**

### 5. Which email authentication mechanism was absent from the message?

Breaking down the three standard mechanisms:
- **SPF:** Passed (`spf=pass`): the sending IP was authorized for that domain, since the attacker fully controlled the spoofed domain's DNS.
- **DKIM:** **Missing entirely**:dkim=none (message not signed), No DKIM-Signature header found.
- **DMARC:** No record published for the domain at all (No DMARC Record found), so it couldn't provide protection either.

Since DKIM was the mechanism with a direct "absent" signal (no signature ever existed, vs. DMARC which "failed" only because no policy was ever configured), it's the most precise answer for what was absent from the message itself.

**Answer: DKIM**

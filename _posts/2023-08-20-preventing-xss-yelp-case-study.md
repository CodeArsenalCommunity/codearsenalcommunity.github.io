---
layout: post
title: "Fixing a $6,000 XSS, A Case Study of Yelp.com"
author: rohitcoder
categories: [XSS]
tags: [security, xss, javascript]
image: https://i.imgur.com/IXVjp83.jpg
featured: false
hidden: false
---

## Disclaimer
The following report is a simulation of [HackerOne Report #2010530](https://hackerone.com/reports/2010530) intended to illustrate potential vulnerabilities, attack scenarios, and mitigation strategies related to a hypothetical Cross-Site Scripting (XSS) vulnerability in a web application. The specific code examples provided in this report are not taken directly from Yelp's backend systems; they are designed to represent a simplified and generic scenario for explanatory purposes.

While the described vulnerability and solutions are based on best practices and industry knowledge, they may not directly reflect the actual code, architecture, or security measures implemented by Yelp. The content of this report aims to educate and raise awareness about web security issues, rather than describing the precise details of Yelp's internal systems.

It's important to note that real-world security assessments and remediation efforts should be conducted in collaboration with experienced security professionals and with access to the actual code and systems in question. The steps and recommendations outlined in this report are general guidelines that should be adapted and applied appropriately to the specific context of each application.

## Introduction

In the realm of web application security, Cross-Site Scripting (XSS) vulnerabilities continue to pose a significant threat. In this report, we delve into a hypothetical case study involving Yelp, a popular online review platform, to explore the potential risks associated with a persistent XSS vulnerability present in JavaScript code. We will analyze the vulnerability, discuss its implications, and outline effective mitigation strategies.

### Vulnerability Description

The vulnerability centers around the unescaped reflection of the `window.yelp.guv` property on Yelp's homepage. Although intended for legitimate purposes, inadequate handling of user input enabled attackers to manipulate the property's value and execute arbitrary JavaScript code.

### Attack Scenario

The attacker executed the following steps to exploit the vulnerability:

1. **Identify Vulnerability:** The attacker identified the unescaped reflection of `window.yelp.guv` and devised a payload to exploit it.
2. **Payload Injection:** Crafting a payload to execute malicious JavaScript, the attacker stored it in the `yelpmainpaastacanary` cookie through a specially-crafted request.
3. **Payload Execution:** The attacker visited a page where unescaped cookie reflection occurred, triggering the malicious payload's execution.
4. **Payload Effect:** The payload, including an `alert` function, generated an alert dialog displaying the message "1."
5. **Ongoing Impact:** The malicious script executed whenever users accessed pages with unescaped reflection of the `guvo` payload.

### Potential Impacts

The Persistent XSS vulnerability exposes users to grave consequences:

- **Sensitive Data Theft:** Attackers can steal user login credentials and sensitive data through payload manipulation.
- **User Account Compromise:** Exploiting the vulnerability grants attackers unauthorized access, enabling actions on behalf of users.
- **Malware Propagation:** Injected malicious scripts spread malware to users' devices, potentially compromising their systems.

## Technical Details and Exploitation

The attacker exploited the vulnerability through the following steps:

1. **Cookie Manipulation:** Insecure cookie manipulation allowed the attacker to smuggle the `guvo` payload.
2. **Crafted Request:** The attacker's request contained the payload `?canary=asdf%20guvo%3D%3C%2Fscript%3E%3Cscript%3Ealert(1)%3C%2Fscript%3E`.
3. **Cookie Injection:** The server stored the `yelpmainpaastacanary` cookie with the malicious payload.
4. **Payload Execution:** The XSS payload triggered when unescaped cookie reflection took place, executing arbitrary code.

## Mitigation and Fix: Comprehensive Approach

Addressing this vulnerability necessitates a holistic approach. The following steps detail a comprehensive fix, preventing XSS attacks:

### Secure Initialization

Eliminate insecure values from `window.yelp.guv`, ensuring no malicious code execution due to improper values.

### Input Validation and Sanitization

Thoroughly validate and sanitize user-generated content before rendering to prevent malicious script execution via injected inputs.

### Output Encoding

Encode user-generated content before inclusion in HTML, JavaScript, or other output contexts to avert browser interpretation of malicious code.

### Strengthen Cookie Handling

Enhance cookie handling security with proper configuration of attributes such as Secure, HttpOnly, and SameSite.

### Content Security Policy (CSP)

Implement a robust CSP to confine script execution to trusted domains, limiting the impact of XSS attacks.

### Accurate Cookie Parsing

Ensure precise cookie parsing to prevent unintended behavior and data exposure risks.

### Utilize Frameworks and Libraries

Leverage reputable web frameworks and libraries equipped to prevent XSS vulnerabilities, offering secure methods for handling user-generated content.

## Fix in JavaScript

To illustrate the mitigation process, we've outlined the necessary steps along with code examples:

### Step 1: Secure Initialization

Remove the insecure value from `window.yelp.guv`:
```javascript
// Before Fix (Vulnerable Code)
window.yelp = window.yelp || {};
window.yelp.guv = 'F7A3C854B7CC0230'; // Insecure value

// After Fix
window.yelp = window.yelp || {};
window.yelp.guv = ''; // Remove the insecure value
```

### Step 2: Input Validation and Sanitization

Properly escape and validate the user's input:
```javascript
const payload = '<script>alert(1)</script>';
const escapedPayload = payload.replace(/</g, '&lt;').replace(/>/g, '&gt;');
```

### Step 3: Cookie Handling

Set the secure and HttpOnly flag for cookies:
```javascript
document.cookie = `yelpmainpaastacanary=${escapedPayload}; Domain=.yelp.com; Path=/; Secure; SameSite=Lax; HttpOnly`;
```

### Step 4: Accurate Cookie Parsing

Refactor the code to ensure proper cookie parsing:
```javascript
// Refactor the code to ensure proper cookie parsing
const cookies = document.cookie.split(';');
const cookieMap = {};
for (const cookie of cookies) {
    const [name, value] = cookie.split('=');
    cookieMap[name.trim()] = decodeURIComponent(value);
}
const yelpmainpaastacanaryValue = cookieMap['yelpmainpaastacanary'];
```

### Step 5: Output Encoding

Output escape the cookie values while reflecting:
```javascript
const yelpmainpaastacanaryValue = getCookie('yelpmainpaastacanary');

const escapedYelpmainpaastacanaryValue = yelpmainpaastacanaryValue.replace(/</g, '&lt;').replace(/>/g, '&gt;');

// Display the escaped cookie value in the HTML
document.getElementById('cookieValue').innerHTML = escapedYelpmainpaastacanaryValue;
```
## Conclusion
The hypothetical scenario involving Yelp's JavaScript code highlights the critical nature of Cross-Site Scripting vulnerabilities. By following best practices, implementing input validation, output encoding, and adopting a comprehensive security strategy, organizations can effectively mitigate the risks associated with XSS attacks. While the specifics of Yelp's systems may differ, the principles outlined in this report offer valuable insights into safeguarding web applications against such threats.

By consistently educating developers, employing secure coding practices, and prioritizing security assessments, businesses can protect their users and maintain the integrity of their online platforms.
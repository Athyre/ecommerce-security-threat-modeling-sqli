# E-Commerce Security Threat Modeling & SQL Injection Assessment

![Framework](https://img.shields.io/badge/Framework-STRIDE-blue)
![Attack Demo](https://img.shields.io/badge/Attack%20Demo-SQL%20Injection-red)
![Target](https://img.shields.io/badge/Lab%20Target-OWASP%20Juice%20Shop-orange)
![Type](https://img.shields.io/badge/Type-Academic%20Project-lightgrey)

A security assessment of a large-scale e-commerce marketplace (case study: **Shopee**), combining **STRIDE threat modeling**, a **hands-on authentication bypass demonstration via SQL Injection**, and a **prioritized risk mitigation roadmap**.

This project was completed as the final assignment (AOL) for **COMP6549001 – Software Security** at Bina Nusantara University.

📄 **Full report:** [`Software_Security_-_AOL-Kelompok3.pdf`](./Software_Security_-_AOL-Kelompok3.pdf) *(written in Bahasa Indonesia)*

---

## Disclaimer

- The threat model of Shopee is based **entirely on public information** (desk research, public financial filings, OSINT, and observation of the app as a normal user). **No unauthorized access or testing was performed against Shopee's real systems.**
- The SQL Injection attack was demonstrated **only in an isolated local lab** against **[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)**, an intentionally vulnerable application built for security training.
- Findings about Shopee are **hypothetical threat scenarios** for educational purposes, not confirmed vulnerabilities in the production platform.
- This repository is for **educational use only**. Do not use the techniques described here against systems you do not own or have explicit permission to test.

---

## Overview

| | |
|---|---|
| **Case study** | Shopee E-Commerce (marketplace, ShopeePay, SPayLater, COD) |
| **Features analyzed** | Payment, Search Bar, Promo/Voucher, Login/Register, Cash on Delivery (COD) |
| **Threat modeling framework** | STRIDE (with Data Flow Diagrams, level 0 and 1) |
| **Attack demonstrated** | Authentication Bypass via SQL Injection on the login endpoint |
| **Lab target** | OWASP Juice Shop (local environment) |
| **Tools** | Burp Suite, web browser |
| **Mitigation approach** | Risk-based trade-off: Address / Transfer / Accept |

---

## Methodology

1. **Data gathering**: Public document study (Sea Limited filings, APJII, Compas, SimilarWeb), direct app observation, and passive OSINT (job postings, engineering blogs, BuiltWith/Wappalyzer).
2. **Asset identification**: 12 critical assets mapped, from credential databases and payment gateway to CDN/cloud infrastructure and mobile apps.
3. **Threat modeling**: DFD level 0/1 and STRIDE classification with risk ratings.
4. **Attack demonstration**: Reconnaissance → vulnerability identification → exploitation → impact demonstration.
5. **Mitigation planning**: Defense-in-depth recommendations with a phased implementation roadmap.

---

## Key STRIDE Findings

| Category | Threat | Affected Component | Risk |
|---|---|---|---|
| **Spoofing** | Credential stuffing / SQLi account takeover | Authentication system, credential DB | High |
| **Tampering** | Client-side bypass of voucher/discount logic | Promo system, business logic | High |
| **Repudiation** | Buyers denying COD receipt due to one-sided courier confirmation | COD system, transaction DB | High |
| **Information Disclosure** | SQL Injection via unvalidated search input | Search engine, API endpoints | High |
| **Denial of Service** | Request flooding / oversized queries on search | Cloud infrastructure, API endpoints | High |
| **Elevation of Privilege** | Stored XSS leading to admin session hijacking | Session store, admin panel | High |

---

## Attack Demonstration: Authentication Bypass via SQL Injection

**Target:** OWASP Juice Shop login (`POST /rest/user/login`) · **Environment:** local lab

| Step | Description | Outcome |
|---|---|---|
| **1. Reconnaissance** | Captured a normal login request in Burp Suite to learn the endpoint and JSON body structure. | Identified `email` and `password` parameters sent as JSON. |
| **2. Vulnerability identification** | Submitted a single quote (`'`) in the email field. | Server returned HTTP 500 with a raw `SQLITE_ERROR`, leaking the DB engine and the query structure. |
| **3. Exploitation** | Submitted a boolean-based payload in the email field. | Query logic was altered; server returned HTTP 200 with a valid authentication token. |
| **4. Impact** | Logged in without knowing any password. | Authenticated as the first user in the database: the **administrator** account. |

**Security impact:** Spoofing + Elevation of Privilege, with full compromise of Confidentiality and Integrity.

### Root Cause

1. **Insufficient input validation and sanitization** of the login form.
2. **Dynamic SQL via string concatenation** instead of parameterized queries.
3. **Insecure error handling** that exposed database errors to the client.

---

## Mitigation Strategy

Designed around **defense-in-depth** and a **trade-off approach**:

- **Address**: Parameterized queries / ORM, server-side validation ("never trust the client"), MFA, rate limiting and account lockout, database locking and idempotency keys, two-way OTP confirmation for COD, immutable audit logging, RBAC, secure session management (HttpOnly / Secure / SameSite cookies, CSP), WAF with OWASP Core Rule Set, generic error pages.
- **Transfer**: Phishing risk in promo chat is shifted toward users through recurring cybersecurity awareness programs.
- **Accept**: Bot-driven request spamming on promos is accepted as a business trade-off (scaling cost outweighs risk value), with active monitoring (Prometheus and Grafana) as compensation.

### Implementation Roadmap

| Phase | Timeline | Focus |
|---|---|---|
| **Phase 1: Critical** | 0–3 months | Parameterized queries, secure cookie flags, custom error pages, WAF, login rate limiting, server-side checkout validation |
| **Phase 2: High** | 3–6 months | MFA, COD OTP confirmation, DB locking and idempotency, RBAC, audit logging, output encoding |
| **Phase 3: Medium** | 6–12 months | Lightweight KYC and device fingerprinting, device binding, admin panel hardening, user education, traffic monitoring, periodic pentesting |

---

## Repository Structure

```
.
├── README.md
└── Software_Security_-_AOL-Kelompok3.pdf   # Full report (Bahasa Indonesia)
```

---

## What the Full Report Contains

1. Executive summary
2. Company profile, business flow, and tech stack
3. Data gathering methods and findings, plus critical asset identification
4. STRIDE threat modeling (DFD level 0 and 1, threat table)
5. Attack demonstration (step by step, with Burp Suite evidence) and impact analysis
6. Mitigation and remediation strategy, with phased roadmap
7. Conclusion and recommendations
8. References

---

## Team (Group 3, Class LC07)

- Andhika Permana Sagiyono
- Darren Gavriel Suntara
- Michael Ernst Jeremy Octama
- Nathasa Bintang Kayesa
- Sebastian Chang

**Lecturer:** Ayu Mauliana, S.Kom., M.Kom
**Institution:** Bina Nusantara University, School of Computer Science
**Academic year:** 2025–2026, Semester 4

---

## 🔗 References & Further Reading

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [CAPEC: Common Attack Pattern Enumeration and Classification](https://capec.mitre.org/)
- Shostack, A. (2014). *Threat Modeling: Designing for Security*. John Wiley & Sons.

The complete reference list is available in the full report.

---

## License & Usage

This work is shared for educational and portfolio purposes. Shopee and all related names and trademarks belong to their respective owners; this project is **not affiliated with or endorsed by Shopee or Sea Limited**.

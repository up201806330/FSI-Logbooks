# Trabalho realizado na Semana #3

## Identification - CVE-2021-26855

- Also known as ProxyLogon, the CVE-2021-26855 is a  Microsoft Exchange Server Remote Code Execution Vulnerability.
- Vulnerability where an attacker can bypass authentication and impersonate as the admin or read emails on a physical server instance.
- Allows chaining of further vulnerabilities (e.g. CVE-2021-27065 for code execution), and as such poses a critical security risk.
- Microsoft Exchange Server versions affected: 2013 below 15.00.1497.012, 2016 CU18 below 15.01.2106.013, 2016 CU19 below 15.01.2176.009, 2019 CU7 below 15.02.0721.013 and 2019 CU8 below 15.02.0792.010.

## Listing

- Discovered by DEVCORE on December 10th, 2020 and reported afterwards to Microsoft via the Microsoft Security Response Center (MSRC) Portal.
- MSRC published a patch and advisory and acknowledged DEVCORE officially on March 3rd, 2021
- CVSS v3.0: Base Score Metrics 9.1 / 10, scored by Microsoft.

## Exploit

- Initial knowledge of the external IP address or domain name of a publicly server and target email are required.
- An attacker can generate and send custom HTTP POST request with an XML payload that performs any operation on mailboxes.
- There is a Metasploit module that automates RCE (Remote Code EXecution) https://www.rapid7.com/db/modules/exploit/windows/http/exchange_proxylogon_rce/
- Can be tested via a python script that can be found at https://github.com/RickGeex/ProxyLogon

## Attacks

- With write access, scripts can allow attackers remote execution privilege and as such poses critical threat.
- Apart from the dangers of RCE, the simple access to emails can result in blackmail through encryption or leaking.
- As of March 12th, there were between 7000 and 8000 UK vulnerable servers, of which approximately half had been patched 
- In January 2021, Volexity (security firm) detected an attack exploiting a zero-day SSRF vulnerability (CVE-2021-26855), allowing the attacker to steal content of user mailboxes.

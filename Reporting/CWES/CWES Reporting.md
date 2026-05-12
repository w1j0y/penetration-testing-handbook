# CWES Reporting

Note: These notes also apply well to CBBH-style web reporting. CWES is the current name for what was previously the CBBH certification track.

---

## Why web reporting is different

Web application reports (CWES/CBBH) are scoped differently from infrastructure reports (CPTS):

- No network enumeration appendices — no host discovery, no exploited hosts, no compromised users.
- No internal network compromise walkthrough — findings speak for themselves.
- Appendix is flags only.
- Executive summary uses grouped categories, not a finding-by-finding list.
- Remediation is per-finding only — no separate Short/Medium/Long term summary section.

The result is a shorter, more focused document, but the finding quality standards are identical.

---

## Report mindset

Each finding must be independently reproducible from the Finding Evidence section alone.

Think of yourself as writing a bug bounty report for a sophisticated program — clear, complete, no assumptions, reproduction from a clean state.

Write findings while you still have the application open and the request history in Burp. Reconstructing steps from memory hours later is how important details get lost.

---

## Finding-first workflow

CWES reporting is finding-first. There is no walkthrough to fill. The moment you confirm a finding is exploitable:

1. Open a new finding entry.
2. Fill the title, affected component, CWE, and a one-line root cause while the session is fresh.
3. Export the relevant Burp requests and screenshots.
4. Write the Finding Evidence section before moving to the next target.

---

## When to start writing

Start the report file before you start testing. Fill in:

- Cover page
- Engagement contacts
- Scope table (in-scope URLs/ports from the exam instructions)
- In-scope vulnerability type list
- Out-of-scope activity list

These sections are static and take under 10 minutes. Having them ready removes friction when you need to add a finding quickly.

---

## Web finding structure

### Title

Include the full attack chain — not just the vulnerability class.

Good patterns:
```
Remote Code Execution via Session Poisoning Exploited Through LFI in Language Parameter
Authentication Bypass via SQL Injection in Login Form
Stored XSS via <PARAMETER> in <COMPONENT>
IDOR via Direct Object Reference in <ENDPOINT>
Unrestricted File Upload on <APPLICATION>
```

Bad: `LFI`, `XSS`, `File Upload`, `SQL Injection`

The title should tell a grader what the impact is, not just the category.

### Severity / CVSS

CVSS 3.1 score and full vector string. Calculate it — do not estimate.

Web findings that achieve RCE or auth bypass on real functionality tend to score Critical or High (8.0+). Score honestly.

### CWE

Single CWE number and name from MITRE.

### Overview

2–4 sentences. Explain why the vulnerability exists, then how it appears in this specific application. This is Root Cause — do not mix in impact or remediation here.

Example:
> The application passes a user-supplied file path parameter directly to an include statement without sanitisation, allowing an attacker to include arbitrary files from the server filesystem. The `lang` parameter on `<URL>` is not restricted to an allowlist of valid values.

### Impact

Bullet list, worst consequence first. Tie the impact to this specific application's context, not generic boilerplate.

```
- Read arbitrary files from the server filesystem, including application source code and credentials
- Chain with session file inclusion to achieve remote code execution as <SERVICE_USER>
- Access internal services reachable from the web server
```

### Affected Components

Most specific label possible.

Format: `<SUBDOMAIN>:<PORT><PATH>` or `<DOMAIN><PATH>` for web findings.

Example: `<APPLICATION>:<PORT>/upload/profile-picture`

### Reproduction Steps

Step-by-step. Past tense, third person ("the tester"). Each step: one sentence → request/response or screenshot.

A grader must be able to reproduce from a clean state using only this section.

See the *Reproduction step quality* section below for details.

### Evidence

Screenshots and request/response pairs that prove the finding is real and exploitable.

See the *Screenshots and request/response evidence* section below.

### Recommendations

Bullet list of actionable steps. No product names. No code.

```
- Validate the language parameter against a server-side allowlist
- Disable dynamic file includes; use static mappings instead
- Restrict the web server process to the minimum required filesystem access
```

### References

- MITRE CWE page
- OWASP prevention cheat sheet (if applicable)
- PoC tool or public write-up (if relevant)

---

## Reproduction step quality

A reproduction that skips steps will fail grader review. Rules:

- Start from a clean browser session / clean Burp history.
- Document every click, navigation, and request — even setup steps ("the tester logged in as `<USERNAME>` at `<URL>`").
- Show exact tool commands with all flags.

```
ffuf -u http://<TARGET_IP>/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
```

- Show the full request and response for the exploitable step, not just a screenshot of the result.
- If URL encoding is needed, show both the decoded payload and the encoded form.
- End with the impact confirmed: the shell output, the extracted data, the bypassed page.

---

## Screenshots and request/response evidence

For web findings, evidence commonly includes:

- Burp Suite intercept with the manipulated request
- The server response showing the exploit result
- Burp Decoder screenshot if payload encoding is key to the technique
- Browser screenshot of the resulting page (for XSS, IDOR, auth bypass)
- Command output if RCE is achieved

Screenshot rules:
- Crop to the relevant portion of the screen.
- Do not include full Burp windows unless the full context is necessary.
- Place the screenshot immediately after the step that produced it.
- The surrounding sentence explains what the screenshot shows — no separate caption block.

**What to redact:**
- Session tokens and cookies in screenshots (replace with `<SESSION_TOKEN>`)
- Passwords in request bodies
- Flags in screenshots used in the public version (use `<FLAG_VALUE>`)

---

## Writing impact clearly

Impact explains what the vulnerability allows an attacker to do in terms of the application's security model and business context.

Bad:
> This vulnerability may allow an attacker to perform actions.

Good:
> The tester was able to read the application's database configuration file, which contained cleartext credentials for the backend database. These credentials could be used to access and modify all application data.

Rules:
- Start with the worst consequence.
- Name the specific data or capability at risk.
- Avoid generic impact statements copy-pasted across findings.
- Use conditional present: "allows an attacker to...", "can result in...".
- Do not overclaim — only claim impact that you demonstrated.

---

## Reportable vs interesting

Not everything interesting is reportable. Before writing a finding:

- Can you demonstrate actual exploitability, not just the presence of a vulnerable pattern?
- Is the impact meaningful in the application's context, or is it theoretical?
- Is this within the explicitly stated in-scope vulnerability types?

**Out-of-scope items to skip:**
- Self-XSS (requires attacker to use their own session)
- DoS (explicitly out of scope in most CWES/CBBH programs)
- Findings requiring MitM position
- Scanner output with no manual confirmation
- Theoretical vulnerabilities with no demonstrated impact

Write only findings you can fully demonstrate.

---

## False-positive discipline

Before submitting a finding:

- Reproduce it from a clean browser session.
- Confirm the behavior is consistent, not a one-time anomaly.
- Verify the impact you claim is actually achievable — not just inferred from the vulnerability class.
- For injections: show extracted data, not just an error message change.
- For auth bypass: show that protected content is actually accessible.

---

## Common web finding examples

Generic examples showing finding title format and finding evidence pattern:

**File upload leading to RCE:**
```
Title: Remote Code Execution via Unrestricted File Upload on <APPLICATION>
Evidence: tester uploaded shell.php → curl to <UPLOAD_PATH>/shell.php?cmd=id → uid=33(www-data)
```

**SQL injection:**
```
Title: Authentication Bypass via SQL Injection in Login Form
Evidence: tester submitted ' OR 1=1-- - as username → application logged tester in as <USER>
```

**IDOR:**
```
Title: Insecure Direct Object Reference Exposing <DATA_TYPE> in <ENDPOINT>
Evidence: tester changed user ID parameter from <OWN_ID> to <OTHER_ID> → response contained <OTHER_USER>'s <DATA_TYPE>
```

**LFI to RCE chain:**
```
Title: Remote Code Execution via Session Poisoning Exploited Through LFI in <PARAMETER>
Evidence (two separate findings, independently documented):
  Finding 1 — LFI: tester passed ../../../../etc/passwd → file contents returned
  Finding 2 — RCE: tester poisoned PHP session via log injection → included session file → code executed
```

---

## Common mistakes

- Grouping related vulnerabilities into one finding — LFI and the follow-on RCE are two separate findings, each independently documentable.
- Writing "I" instead of "the tester" in technical sections.
- Skipping the out-of-scope list in the executive summary — graders expect it.
- Submitting scanner output as a finding without manual exploitation confirmation.
- Missing the encoded form of a payload — show both decoded and encoded when encoding is relevant.
- Cropping screenshots too aggressively — the URL and response code should still be visible in most cases.
- Leaving session tokens unredacted in screenshots.
- Writing generic impact statements — tie impact to this specific application.
- Adding a Remediation Summary section — CWES/CBBH reports do not have one; remediation is per-finding only.
- Inflating CVSS scores — an information disclosure with no sensitive data in scope is not a 9.8.

---

## Final report review checklist

Before submitting:

- [ ] Cover page complete (exam name, date, version)
- [ ] Scope table lists all in-scope URLs and ports from the exam instructions
- [ ] In-scope vulnerability types explicitly listed in the executive summary Approach section
- [ ] Out-of-scope activities explicitly listed
- [ ] Executive summary Assessment Overview groups findings by theme (not a finding-by-finding list)
- [ ] Executive summary Impact paragraph uses business/risk language — no acronyms, no tool names
- [ ] Executive summary Recommended Remediations is a short bullet list
- [ ] Each finding has: Title, CVSS, CWE, Overview (Root Cause), Impact, Affected Component, Reproduction Steps, Evidence, Recommendations, References
- [ ] Each Finding Evidence section is reproducible from scratch, without reading any other section
- [ ] All screenshots placed immediately after the action that produced them
- [ ] Sensitive values (tokens, cookies, passwords) redacted in evidence
- [ ] All findings confirmed as reproducible from a clean session
- [ ] No out-of-scope findings submitted
- [ ] No scanner-only output submitted without manual confirmation
- [ ] Flags appendix complete (Flag #, Application URL, Flag Value, Flag Location, Method Used)
- [ ] No remediation summary section — remediation is per-finding only
- [ ] Third person ("the tester") used throughout technical sections

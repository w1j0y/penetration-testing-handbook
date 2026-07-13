# CPTS Reporting

Credit: My CPTS reporting workflow was heavily guided by Bruno Rocha Moura's "HTB CPTS Reporting: The Easy Way":
https://www.brunorochamoura.com/posts/cpts-report/

---

## Why reporting matters

The exam is only passed when the report passes review. A technically correct attack chain with weak documentation will fail. Treat the report as a deliverable to a real client, because the grader reads it exactly that way.

---

## Report mindset

Fill the report **while you test**, not after.

The structure PDF (see companion file in this directory) maps each section to the moment during the exam when you should fill it. Follow that trigger-based approach:

- Found a host → add it to Host & Service Discovery.
- Got a shell → add the host to Exploited Hosts.
- Grabbed a flag → add it to Flags Discovered.
- Uploaded a file → add it to Changes/Cleanup.

If you try to reconstruct everything from memory at the end, you will miss details, forget command flags, and misremember the order of steps.

---

## When the exam starts

Before touching anything:

1. Create your report file in SysReptor (or your preferred tool).
2. Fill in the cover page, contacts table, and scope table from the exam instructions.
3. Paste the in-scope IP ranges / domains into the Scope section.
4. Set the date range: start date is today.
5. Open the appendix tables (Host Discovery, Subdomain Discovery, Exploited Hosts, Compromised Users, Changes/Cleanup, Flags) so they are ready to receive data immediately.

---

## When you discover a host or service

Add it to **Appendix A.2: Host & Service Discovery** as soon as the port scan finishes.

| IP Address | Port | Service | Notes |
|---|---|---|---|
| `<TARGET_IP>` | 80 | http (nginx) | Hosts `<APPLICATION>` web app |
| `<TARGET_IP>` | 443 | https | |
| `<INTERNAL_IP>` | 445 | microsoft-ds | Domain-joined Windows host |
| `<INTERNAL_IP>` | 88 | kerberos-sec | Domain Controller for `<DOMAIN>` |

Notes column: use it to record context you will need later ("Domain Controller for `<DOMAIN>`", "Runs `<SERVICE_NAME>` version `<VERSION>`").

---

## When you find a subdomain or virtual host

Add it to **Appendix A.3: Subdomain Discovery** immediately.

| URL | Description | Discovery Method |
|---|---|---|
| `<SUBDOMAIN>.<DOMAIN>` | Internal HR portal | DNS zone transfer |
| `<VHOST>.<DOMAIN>` | Dev environment | Virtual host fuzzing (ffuf) |

Record the discovery method: it matters for reproducibility.

---

## When you get foothold

- Add the host to **Appendix A.4: Exploited Hosts**.
- Screenshot the shell prompt showing `id` / `whoami` with hostname visible.
- Note the vulnerability that gave access: use the finding title.

| Host | Scope | Method | Notes |
|---|---|---|---|
| `<HOSTNAME>` (`<TARGET_IP>`) | External | Unrestricted File Upload | Initial foothold |
| `<HOSTNAME>` (`<INTERNAL_IP>`) | Internal | `<PRIVILEGE_ESCALATION_METHOD>` | Domain Controller |

---

## When you root a host

- Screenshot `id` / `whoami` output, with hostname visible in the prompt.
- Capture the flag: screenshot with the flag value visible (or redact for public).
- Add the flag to **Appendix A.7: Flags Discovered**.
- Document any files you left behind in **Appendix A.6: Changes/Cleanup**.

---

## When you compromise a user

Add to **Appendix A.5: Compromised Users**.

| Username | Type | Method | Notes |
|---|---|---|---|
| `<USERNAME>` | Local admin | Credential in config file | On `<HOSTNAME>` |
| `<USERNAME>` | Domain user | Password spray | `<DOMAIN>` domain |
| `<USERNAME>` | Domain admin | `<LATERAL_MOVEMENT_METHOD>` | `<DOMAIN>` domain |

Type options: local user, local admin, domain user, domain admin, service account.

---

## When you discover a finding

Write the finding entry while the exploitation is fresh. Minimum required immediately:

- Title
- Affected Component
- CWE number
- One-sentence root cause

Come back and fill Impact, Recommendations, References, and Finding Evidence after the session, but capture the core while you remember it.

---

## Finding structure

SysReptor field names are used below since most candidates use SysReptor.

### Title

Clear, action-oriented. Include the location and mechanism, not just the vulnerability class.

Good patterns:
```
Unrestricted File Upload on <APPLICATION>
Remote Code Execution via <VECTOR> in <SERVICE>
Authentication Bypass via SQL Injection in <COMPONENT>
Privilege Escalation via <METHOD>
Excessive Active Directory <PERMISSION_TYPE> Privileges
Cleartext Credentials Found in <FILE_TYPE>
```

Bad: `SQL Injection`, `Weak Permissions`, `RCE`

### CWE

Single CWE number and name. Look it up on MITRE. Do not guess.

Example: `CWE-434: Unrestricted Upload of File with Dangerous Type`

### CVSS

CVSS 3.1 score and full vector string. Calculate it. Do not estimate.

Example: `8.8 / CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`

Use SysReptor's built-in calculator or an online CVSS calculator.

### Root Cause

2–4 sentences. Explain *why* the vulnerability exists, then how it appears in this instance. Do not describe impact or remediation here.

Example:
> The application does not validate or restrict the type of files users may upload, allowing an attacker to upload executable files. The file upload endpoint on `<APPLICATION>` stores submitted files directly within the web root without content or extension checks.

### Impact

Bullet list. Worst consequence first. Stay specific to this finding: no boilerplate.

```
- Execute arbitrary commands on the server as <SERVICE_USER>
- Gain persistent access to <HOSTNAME>
- Access internal systems accessible from <HOSTNAME>
- Read sensitive application data stored on the host
```

### Affected Components

Most specific label possible.

Formats:
- `<HOSTNAME> (<APPLICATION> / port <PORT> TCP)`
- `<SUBDOMAIN>:<PORT><PATH>`
- `<DOMAIN> Active Directory domain` (for AD-wide issues)

### Recommendations

Bullet list of actionable steps. No code, no product names.

```
- Implement a whitelist of allowed file extensions
- Store uploaded files outside the web root
- Validate file content (MIME type and file header), not just extension
- Run the web server process under a dedicated low-privilege account
```

### References

Include relevant links:
- NIST CVE page (if a CVE applies)
- MITRE CWE page
- OWASP prevention cheat sheet (if applicable)
- PoC tool or script link (if the tool is public and not yours)

### Finding Evidence

Step-by-step reproduction narrative. Past tense, third person ("the tester").

One action per step: one sentence of explanation → code block or screenshot.

```
The tester navigated to the file upload functionality at <URL> and observed
that the application accepted all file types without restriction.

The tester uploaded a file named `shell.php` containing the following payload:

    <?php system($_REQUEST["cmd"]); ?>

The tester confirmed remote code execution by sending the following request:

    curl http://<TARGET_IP>/<UPLOAD_PATH>/shell.php -d 'cmd=id'

The output confirmed execution as <SERVICE_USER>:

    uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Screenshots: place them immediately after the action that produced them. Crop to the relevant area. Do not add a separate caption block: the surrounding sentence explains what is shown.

A grader must be able to reproduce the finding from this section alone.

---

## Walkthrough structure

The walkthrough covers the **shortest path to domain compromise only**. Off-path findings go in Technical Findings, not here.

State this at the top: "The following walkthrough documents the shortest path to full compromise of the `<DOMAIN>` domain. Findings not on this path are documented in the Technical Findings section."

### Short attack chain summary

A numbered bullet list: one sentence per step, no images, no code blocks.

```
<CANDIDATE_NAME> ("the tester" herein) performed the following to fully compromise the <DOMAIN> domain:

1. Gained initial foothold via <VULNERABILITY> on external-facing <SERVICE>
2. Discovered <INTERNAL_SERVICE> through post-exploitation enumeration
3. Exploited <SECOND_VULNERABILITY> to obtain credentials for <USER_TYPE>
4. Moved laterally to <INTERNAL_HOSTNAME> using discovered credentials
5. Escalated privileges to <PRIVILEGED_USER> via <PRIVILEGE_ESCALATION_METHOD>
6. Dumped domain credentials via <CREDENTIAL_DUMP_METHOD>
7. Achieved full domain compromise of <DOMAIN>
```

### Detailed walkthrough

One section per host or logical phase. Structure per phase:

1. Short intro paragraph: what the tester was trying to do.
2. Explain WHY each action was taken, not just what.
3. Code block for every command.
4. Screenshot after each key action.
5. End the phase with the result: shell gained, credential found, flag captured.

Screenshot triggers in the walkthrough:
- Initial foothold (shell prompt with `id`/`whoami` visible)
- Flag capture (flag value visible)
- Key discoveries (credential file found, service version identified)
- Privilege escalation (pre- and post-escalation user output)
- Each lateral movement step (new shell on a new host)

---

## Evidence checklist

For each finding and walkthrough step:

- [ ] Exact command shown (not paraphrased)
- [ ] Relevant output shown immediately below the command
- [ ] Screenshot placed after the action that produced it
- [ ] Screenshot cropped to the relevant area only
- [ ] Hostname or path visible in the shell prompt (confirms which host)
- [ ] Proof of exploitability: not just "command ran" but what was accessible
- [ ] Sensitive values use `<PLACEHOLDER>` in the public version

---

## Cleanup / changes made

**Appendix A.6: Changes/Host Cleanup**

Document every artifact left on target systems, even if reverted.

| Host | Scope | Change / Cleanup |
|---|---|---|
| `<HOSTNAME>` | External | Uploaded `shell.php` to `<UPLOAD_PATH>`. File removed after exploitation. MD5: `<HASH>` |
| `<HOSTNAME>` | Internal | Created local user `<USERNAME>` for testing. Account deleted after session. |
| `<HOSTNAME>` | Internal | Dropped `<TOOLNAME>` to `C:\Windows\Temp\`. File removed. MD5: `<HASH>` |

Include MD5 for all uploaded files: the grader may check.

---

## Flags appendix

**Appendix A.7: Flags Discovered**

Sort ascending by flag number. Fill each entry as you capture the flag. Do not reconstruct from memory.

| Flag # | Host | Flag Value | Flag Location | Method Used |
|---|---|---|---|---|
| 1 | `<HOSTNAME>` | `<FLAG_VALUE>` | `/home/<USER>/user.txt` | `<EXPLOITATION_METHOD>` |
| 2 | `<HOSTNAME>` | `<FLAG_VALUE>` | `/root/root.txt` | `<PRIVILEGE_ESCALATION_METHOD>` |

---

## Domain password review

Not included in the default SysReptor CPTS template. Add it manually as an additional appendix after domain compromise.

Include after obtaining NTDS.dit and cracking hashes. Typical statistics:

- Total hashes obtained
- Number of unique hashes
- Percentage cracked
- Most commonly reused passwords
- Password length distribution

Keep this section factual, no personal details. Generalise any sensitive patterns.

---

## Executive summary

Written for a non-technical reader: a senior manager who knows the business, not security.

Rules:
- No tool names
- No CVE numbers
- No unexplained acronyms
- No jargon
- Short paragraphs (3–5 sentences max)
- Use plain English to describe risk: "an attacker could access all customer records" not "IDOR via BOLA"

**Approach** sub-section: state the testing methodology (Grey Box for CPTS: network ranges provided, no credentials, starting as an unauthenticated external user), date range, and non-evasive stance.

**Assessment Overview**: grouped plain-language summary of findings. No finding names: describe categories and business impact. End with a recommendation to perform periodic assessments.

---

## Remediation summary

Three buckets for each finding:

| Finding | Short Term | Medium Term | Long Term |
|---|---|---|---|
| `<FINDING_NAME>` | Quick config change that removes immediate risk | Architectural fix | Process or program improvement |

**Short term**: config changes, disabling a service, revoking a credential (something done in days).

**Medium term**: code changes, policy updates, patching (something done in weeks).

**Long term**: hardening programs, security training, monitoring maturity (something done over months).

Do not put everything in Short Term. Distribute honestly.

---

## Common mistakes

- Filling appendices after the exam ends instead of during: leads to gaps and wrong order.
- Putting off-path findings in the walkthrough. The walkthrough is for the attack chain only.
- Skipping the bullet list summary: both layers of the walkthrough are required.
- Writing the executive summary for a technical reader. It is for management, not the grader.
- Inflating CVSS scores: calculate them, do not guess.
- Grouping related vulnerabilities into one finding. Each finding must be independently reproducible.
- Writing the Root Cause section as if it is the Impact section: they are different things.
- Forgetting the Domain Password Review appendix. It is not in the default template.
- Leaving uploaded files undocumented in Changes/Cleanup: even deleted files must appear.
- Writing "I" instead of "the tester" in technical sections.

---

## Final report review checklist

Before submitting:

- [ ] Cover page complete (exam name, date, version)
- [ ] Scope table matches exam instructions exactly
- [ ] Executive summary uses plain English, no jargon
- [ ] Assessment Overview groups findings by theme, not by finding name
- [ ] Each finding has Title, CWE, CVSS, Root Cause, Impact, Affected Component, Recommendations, References, Finding Evidence
- [ ] Finding Evidence is reproducible from the section alone
- [ ] Walkthrough has both the bullet list summary and the detailed narrative
- [ ] Walkthrough covers the attack chain only (off-path findings in Technical Findings)
- [ ] All appendix tables are complete and consistent with the walkthrough
- [ ] Changes/Cleanup table lists every artifact (even reverted ones) with MD5 for files
- [ ] Flags table sorted ascending, values filled
- [ ] Domain Password Review appendix added (if domain was compromised)
- [ ] Remediation Summary has Short/Medium/Long term for each finding
- [ ] Third person ("the tester") used throughout technical sections
- [ ] No placeholder left unfilled in the final submission copy

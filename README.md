# Penetration Testing Handbook

This repository contains my personal penetration testing notes, cheat sheets, and practical references compiled during my security studies and research.

It started as a study resource for CPTS and CBBH, and has grown into a broader offensive security handbook covering web, Active Directory, enumeration, exploitation methodology, privilege escalation, reporting, and practical testing workflows.

The notes are updated regularly and are meant to serve as a quick reference and collection of actionable techniques for authorized penetration testing, bug bounty, labs, and security research.

# Take it further with xLimit

This handbook is the public foundation of a larger workflow I am building with **xLimit**.

xLimit is an **authorized-use AI security research assistant** built for pentesters, bug bounty hunters, security researchers, and advanced students. It helps with methodology, triage, validation planning, and report writing, not blind automation.

Its guidance combines real-world methodology with patterns and failure-mode experience gathered from authorized engagements, so responses reflect what tends to work in practice rather than acting as a simple knowledge-base lookup.

## Ways to use xLimit

### 1. Web Assistant

Access the hosted xLimit assistant at:

**app.xlimit.org**

### 2. Automatic In-Agent Retrieval *(Recommended)*

The `xlimit-client` repository includes an **MCP server**.
Register it once with tools such as **Claude Code** or **Codex**, and they can automatically call xLimit during active sessions to retrieve synthesized guidance, no copy-pasting required.

For full setup instructions, see the [step-by-step deployment guide](https://blog.xlimit.org/how-to-deploy-and-use-xlimit-client.html).

<img width="1157" height="132" alt="addxlimitmcp" src="https://github.com/user-attachments/assets/e2a89846-108a-4bc7-8ffe-274655e76402" />
<img width="2548" height="937" alt="claude-prompt-01" src="https://github.com/user-attachments/assets/ac1e50ab-407a-4deb-abdd-b57b406d28c8" />
<img width="2544" height="1140" alt="claude-prompt-02" src="https://github.com/user-attachments/assets/12e378fd-304a-410c-bff1-c6d3e802fcef" />
<img width="2535" height="1013" alt="claude-prompt-03" src="https://github.com/user-attachments/assets/c5636bf2-8a99-4da2-9453-e780c6e7924a" />

### 3. Manual Terminal Workflow

The same repository includes shell scripts such as `xlimit_context.sh` that let you pull raw reference snippets into any terminal-based assistant, whether MCP-compatible or not.

---

## Goal

The goal is simple: make these notes more usable during real authorized security work, whether you are:

* Studying security methodology
* Testing a lab environment
* Working on an internal assessment
* Triaging an approved bug bounty target

Signup now **app.xlimit.org**

Learn more at **xlimit.org**


> Built for guided security research, not blind automation.

---

## Disclaimer

xLimit and this handbook are intended **only for lawful, authorized, and in-scope security testing, research, and education**.



---

## Active Directory

- [Active Directory Mindmap (PDF)](./AD%20notes/Active%20Directory.pdf): A comprehensive mindmap covering enumeration, attacks, and common tools.  
  **Highly recommended for anyone studying or working with AD.**
- [Active Directory Written Notes](./Active%20Directory/): The mindmap is now fully expanded into organized, in-depth written notes and practical walkthroughs.  
  **Browse individual topics and techniques in detail.**

---

> **Disclaimer:** For educational and authorized testing purposes only.


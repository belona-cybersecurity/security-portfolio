# Security Awareness Training — Client A

> **Anonymization note.** This report documents real work performed under authorization
> from the owner. The client's name and infrastructure identifiers were replaced with
> placeholders (`Client A`, `cliente-a`) before publication. Belona Cybersecurity does not
> publish identifiable client data without written authorization.
### Audience: panel users (e.g., lawyers) · Law firm

**Framework:** CIS Controls v8 — **Control 14** (Security Awareness and Skills
Training) and **NIST CSF — PR.AT** (Awareness and Training). Each module notes the
control that backs it.

> Goal: everyone using the system understands the risks and acts securely.
> **The weakest link is usually the human, not the technology.**

---

## Module 1 — Passwords and credential management *(CIS 14.x · NIST PR.AC)*
- Use **long, unique** passwords (12+ character passphrase). Never reuse your email password.
- Use a **password manager**.
- **Never share your account or password** — each person has their own (so the Logs know who did what).
- If you suspect it was seen, **change it immediately**.

## Module 2 — Multi-factor authentication (MFA) *(NIST PR.AC-7)*
- The panel requires **MFA (Google Authenticator)** — your second barrier if your password is stolen.
- **Do not approve** MFA prompts you did not initiate (could be an attacker trying to log in).
- Protect access to your Authenticator app.

## Module 3 — Phishing and social engineering *(CIS 14.2 · law firms are frequent targets)*
- **Be suspicious of urgent emails/messages** asking for credentials, clicks, or transfers.
- Verify the **real sender** and **don't click** suspicious links (hover to see the real URL).
- For a "bank/Google/boss" email asking for data → **confirm through another channel** first.
- Remember: no legitimate party will ask for your password by email.

## Module 4 — Handling confidential client data (Law 1581 — Colombian data protection) *(CIS 14.4)*
- Consultation and case data are **confidential and sensitive**. Handle with maximum care.
- **Do not download** client documents to personal devices unnecessarily; do not forward via insecure channels.
- Access only what you need for your work (**least privilege**).
- For a client request about their data (access, correction, deletion) → follow the privacy policy procedure.

## Module 5 — Secure use of the admin panel *(NIST PR.AC)*
- **Log out** when done, especially on shared computers.
- **Lock the screen** (Win+L) if you step away.
- Do not leave the panel visible to third parties.
- Use your assigned role; if you need more permissions, request them (don't seek workarounds).

## Module 6 — Devices and networks *(CIS 14.x)*
- Keep the OS and browser **updated**.
- Keep **antivirus** active.
- Be careful with **public Wi-Fi**: avoid accessing the panel from untrusted networks.
- Do not install extensions/programs of dubious origin.

## Module 7 — Incident reporting *(CIS 14.6 · NIST RS)*
- If you see **something odd** (a suspicious email, an alert, unrecognized access, a changed file):
  1. **Do not ignore it.**
  2. **Report immediately** to the technical lead and/or legal@cliente-a.example.
  3. Don't try to "fix it" yourself if unsure.
- Reporting on time can prevent a major incident. **Reporting isn't bothering — it's protecting.**

---

## Quick self-check (mark what you'd do right)
- [ ] I use a unique password and MFA
- [ ] I don't share my account
- [ ] I verify senders before clicking
- [ ] I log out and lock the screen
- [ ] I know who to notify about an incident

## Recommended frequency
- Training **at onboarding** + **annual refresh** (CIS 14.1).
- Update the material when threats or the system change.

*Based on CIS Controls v8 (Control 14) and NIST CSF (PR.AT). See also the incident response playbooks.*

<!-- atribucion -->
---

*From the technical portfolio of **Belona Cybersecurity LLC** — Metro Atlanta,
Georgia · [belonacyber.com](https://belonacyber.com) · contact@belonacyber.com*

*© 2026 Belona Cybersecurity LLC. All rights reserved. Published for evaluation
by prospective clients and employers; see [`LICENSE`](LICENSE) for terms.*

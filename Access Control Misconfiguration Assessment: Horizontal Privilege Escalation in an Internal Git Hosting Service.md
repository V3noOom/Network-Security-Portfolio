# Access Control Misconfiguration Assessment: Horizontal Privilege Escalation in an Internal Git Hosting Service

**Course:** Network Security, Stockholm University — Spring 2026
**Context:** Authorized in-course security assessment of internal university lab infrastructure

> **Note on this write-up:** the original assessment identified a specific internal hostname and a specific other user's account. Both have been generalized below — the hostname is referred to only as "an internal Git hosting instance," and the affected account only as "another user's repository." The technical substance (methodology, root cause, classification, and remediation) is unchanged. Handling findings like this responsibly, even from coursework, is itself a security-practice signal worth demonstrating.

## Executive Summary

During an authorized security assessment of an internal Gitea instance used within our security lab environment, I identified a horizontal privilege escalation vulnerability in the repository access-control configuration: any authenticated user — regardless of their relationship to a given repository — could push commits to repositories they did not own. I confirmed this by pushing a clearly-labeled proof-of-access commit to a classmate's repository using my own account, with no special privileges, credential theft, or exploit required beyond having a valid account on the instance.

## Methodology

1. **Enumeration** — Browsed the target repository (publicly readable without authentication) and its commit history. Noticed an existing unauthorized commit from another student, suggesting the misconfiguration was already known to be exploitable.
2. **Verification** — Cloned the repository, reconfigured the git remote to authenticate with my own credentials, and attempted a push.
3. **Confirmation** — The push was accepted without error. The commit and proof file appeared immediately in the repository's web interface, confirming write access had been granted with no ownership or authorization check.

```bash
git clone https://[internal-git-host]/[other-user]/[project-repo].git
cd [project-repo]
git remote set-url origin https://[my-username]:<password>@[internal-git-host]/[other-user]/[project-repo].git
echo "Proof of unauthorized write access - horizontal privilege escalation" > proof.txt
git add proof.txt
git commit -m "Proof of unauthorized write access - horizontal privilege escalation"
git push
```

## Root Cause

The Git hosting instance was configured with no repository-level access controls: by default, any authenticated user was granted write access to every repository on the instance. No branch protection, no collaborator restrictions, and no ownership verification were enforced before accepting a push.

## Vulnerability Classification

- **OWASP A01:2021** — Broken Access Control
- **CWE-862** — Missing Authorization
- **Type** — Horizontal privilege escalation (a user accessing another user's resources at the same privilege level)

This is a misconfiguration, not a software defect — the platform supports proper access controls, but they weren't enabled.

## Impact Assessment

| Impact Area | Description |
|---|---|
| Code integrity | Any user could inject changes into any repository |
| Supply chain risk | Modified code could propagate to anything built from that repository |
| Data tampering | History and files could be altered or deleted by any account holder |
| Accountability | Difficult to distinguish legitimate from illegitimate commits after the fact |

## Mitigation Recommendations

**Immediate**
- Restrict push access to repository owners and explicitly-added collaborators
- Audit all repositories for unauthorized commits
- Default new repositories to private unless public access is intentional

**Hardening**
- Enable branch protection on main/default branches
- Require review before merging
- Enforce signed commits to verify author identity
- Tie accounts to verified identities (e.g., via institutional SSO)

## Ethical Reflection

The technical steps here are identical to what a malicious actor would do — the only thing that differs is authorization and intent. Operating within an explicitly scoped assessment meant this could be documented and reported rather than silently exploited (or discovered later with no way to distinguish it from an actual compromise). That an unrelated unauthorized commit was already present in the repository's history before I started is itself a finding: without proper access controls and audit logging, this kind of access is genuinely hard to detect after the fact.

It's also a reminder that misconfigurations are just as dangerous as unpatched software — the platform here was fully capable of enforcing least privilege; it just wasn't configured to.

## Skills Demonstrated

`access control testing` `OWASP Top 10 mapping` `CWE classification` `Git` `responsible vulnerability documentation` `impact assessment`

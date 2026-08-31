---
title: "Standing Administrator Account Excluded from Conditional Access MFA & Risk-Based Policies"
date: "2026-08-25"
lab: "Lab 02 — Conditional Access, Module 02: MFA Enforcement (CA01)"
environment: "Microsoft Entra ID — Conditional Access"
severity: "High"
finding_type: "Policy Gap"
author: "Mike Tully"
---

# Standing Administrator Account Excluded from Conditional Access MFA & Risk-Based Policies

## 1. Executive Summary

While building a tenant-wide MFA enforcement policy in Microsoft Entra ID, the organization's primary Global Administrator account was found sitting in the exclusion list of two Conditional Access policies — meaning that account could sign in without ever being subject to MFA enforcement or risk-based sign-in blocking. This is the single highest-privilege identity in the tenant, and it was the one account left unprotected by the very controls meant to protect it. The gap was caught during the build, not after, and was corrected the same day: the standing exclusion was removed from both policies, leaving only two dedicated emergency-access accounts excluded, and the fix was verified against live sign-in activity before the policy went into enforcement.

---

## 2. Finding Details

| Field | Value |
|---|---|
| **Finding Title** | Standing Administrator Account Excluded from CA MFA Enforcement & Risk-Based Sign-In Policies |
| **Finding Type** | Policy Gap / Access Control Violation |
| **Date Identified** | 2026-08-25 |
| **Environment / Tool** | Microsoft Entra ID, Conditional Access (tenant: iampamprointheknow42gmail.onmicrosoft.com) |
| **Risk Rating** | High |
| **Affected System(s)** | CA01 - Require MFA All Users; CA02 - Risk_Based Sign-In Block |
| **Affected Account(s)** | Primary Global Administrator (Mike Tully) |

---

## 3. Technical Description

**Lab context:** This finding surfaced during the build of Module 02 of Lab 02 (Conditional Access) — a tenant-wide MFA enforcement policy, `CA01 - Require MFA All Users`, targeting all users and all cloud apps with a grant control of Require Multifactor Authentication.

**What was observed:**

When each policy (`CA01` and, earlier, `CA02 - Risk_Based Sign-In Block`) was reviewed under **Policy details → Excluded identities**, the count read **3 users, 0 groups, 0 roles** — one more than expected. The two expected exclusions were the tenant's dedicated break-glass emergency access accounts (`BreakGlass-Emergency_Access-01`, `BreakGlass-Emergency_Access-02`), which by design are permanently excluded from every Conditional Access policy in this lab so a policy misconfiguration can never lock out all administrative access. Expanding the exclusion list showed the third excluded identity was the tenant's primary Global Administrator account, not a break-glass account.

**How it was discovered:** Manual review of each policy's exclusion list before finalizing the build — checking "who is excluded and why" is treated as a standard step before any Conditional Access policy is considered complete, not an afterthought.

**Why this is a problem:** The Microsoft Entra portal defaults to suggesting exclusion of the currently signed-in administrator when saving a new-or-edited Conditional Access policy ("Don't lock yourself out" warning), as a safety measure during initial build and testing. That default is reasonable *while a policy is still being tuned* — it prevents an admin from being locked out of the tenant by their own in-progress, unstable policy. It is not an acceptable end state for a finished, enforced policy. Left in place, it means:

- The tenant's most privileged identity is exempt from MFA enforcement (CA01) and from risk-based sign-in blocking (CA02).
- A compromised or misused admin credential would face none of the controls the tenant otherwise relies on.
- The exclusion has no legitimate ongoing justification once the policy is stable — unlike the break-glass accounts, which have a permanent, documented reason to be excluded (they exist solely for emergency recovery and are separately monitored).

**Resolution:** For both CA01 and CA02, the Global Administrator account was removed from the exclusion list. The Entra portal's standard confirmation prompt ("I understand that my account will be impacted by this policy. Proceed anyway.") was acknowledged and the change saved. Break-glass accounts remain the only permanent exclusions on both policies.

---

## 4. Risk Rating & Justification

**Rating:** High

**Justification:** Per this lab's risk rating guide, "MFA not enforced on privileged accounts" is rated High–Critical, since it represents a key control failure on the tenant's highest-value identity. This finding is rated High rather than Critical because: (1) this is a lab/development tenant with no production data or live users at risk, (2) the gap was self-identified and remediated within the same build session, before the policy reached a finished/reviewed state, and (3) compensating controls existed throughout — the two break-glass accounts provided an independent recovery path, and no external actor had an opportunity to exploit the window. In a production tenant, the same finding on a live Global Admin account, left unaddressed past initial policy testing, would justify a Critical rating.

**Likelihood:** Medium (requires compromise or misuse of the specific excluded admin credential)
**Impact:** High (full standing tenant privilege, exempt from the two controls meant to catch exactly this kind of misuse)

---

## 5. Business Impact

- **Operational risk:** The account with the broadest possible access in the tenant would be the *least* monitored/controlled account from a Conditional Access standpoint — the opposite of the intended security posture.
- **Compliance risk:** Directly implicates NIST 800-53 IA-2 (MFA for privileged accounts) and SOX ITGC logical access controls, both of which expect privileged accounts to be *more* tightly controlled, not exempted.
- **Reputational/audit risk:** In a real audit or interview review of this policy's configuration/JSON export, an unexplained standing exclusion of the Global Admin account is the first thing a reviewer would flag — there is no defensible permanent justification for it, only a defensible *temporary* one during build/test.

---

## 6. Compliance Mapping

| Framework | Control | Description |
|---|---|---|
| NIST 800-53 | IA-2 | Identification and Authentication (MFA enforcement on privileged accounts) |
| NIST 800-53 | AC-6 | Least Privilege |
| CIS Controls v8 | CIS 6 | Access Control Management |
| SOX ITGC | Logical Access Controls | MFA not enforced, orphaned/standing privileged exclusions |

---

## 7. Remediation Steps

1. Remove the standing Global Administrator exclusion from `CA01 - Require MFA All Users` and `CA02 - Risk_Based Sign-In Block`, leaving only the two break-glass accounts excluded.
2. Acknowledge the Entra portal's "your account will be impacted" confirmation and save the change — this is expected and correct, not an error state.
3. Validate the fix: confirm via Sign-in logs that the admin account's own sign-ins are now evaluated against both policies (visible under the sign-in's Conditional Access tab).
4. Re-test the policy end to end with a non-privileged test account to confirm scope is otherwise correct (see Section 8 — this was done for CA01 using a test user account, `alyssa.morgan@...`, which correctly triggered a Report-only "User action required" result and, after enforcement, a live MFA registration challenge).
5. **Preventive control going forward:** treat "review the exclusion list before calling any CA policy finished" as a standard checklist step for every remaining policy in this lab (Module 03 – Legacy Auth Block, Module 04 – Admin Role Protection). Do not rely on standing admin exclusions for ongoing operational access — use Privileged Identity Management (PIM) just-in-time role activation instead, reserving permanent CA exclusions exclusively for dedicated, monitored break-glass accounts.

---

## 8. Evidence

All screenshots below are in `Lab02_EntraID-Screenshots/02_mfa_enforcement_policy/`.

- **`01-1_CA01_Policy_Created_ReportOnly.png`** — CA01 policy details in Report-only, showing the exclusion count of 3 users (the bug, prior to fix).
- **`02-1_CA01_ClientApps_Verified.png`** — Client apps condition confirmed as Not Configured (no unintended scope narrowing).
- **`03-1_CA01_Alyssa_Signin_Confirmed.png`** — Confirmation of an authenticated test-user session (Alyssa Morgan) used for policy validation.
- **`03-2_CA01_SigninLogs_Alyssa_Filtered.png`** — Sign-in logs filtered to the test user, showing the sign-in event used for testing.
- **`03-3_CA01_ReportOnly_Result_Alyssa.png`** — CA01's Report-only evaluation result for that sign-in: "User action required," confirming the policy correctly identified an account with no MFA registered.
- **`04-1_CA01_Enabled_Confirmation.png`** — Portal confirmation that CA01 was successfully updated and enabled (Report-only → On), following removal of the admin exclusion.
- **`04-2_CA01_Live_MFA_Challenge_Alyssa.png`** — Live enforcement: the test user's sign-in interrupted with a mandatory "Let's keep your account secure" MFA registration prompt.
- **`04-3_CA01_Authenticator_Registration.png`** — Microsoft Authenticator registration (QR pairing) completed for the test identity.
- **`04-4_CA01_Authenticator_Registration_Successful.png`** — Registration confirmed successful; test user now fully subject to live MFA enforcement.

Related exclusion fix on `CA02 - Risk_Based Sign-In Block` (same finding pattern, corrected the same session) is documented under `Lab02_EntraID-Screenshots/05_risk_based_signin_policy/`.

---

## 9. Remediation Status

| Status | Date | Notes |
|---|---|---|
| Identified | 2026-08-25 | Found during pre-finalization review of CA01's exclusion list |
| Reported | 2026-08-25 | Self-identified during build; no external report needed |
| Remediated | 2026-08-25 | Admin exclusion removed from CA01 and CA02; break-glass-only exclusions confirmed |
| Verified | 2026-08-25 | Confirmed via sign-in logs (admin sign-in now evaluated) and live test-user enforcement (MFA challenge + registration) |

---

## 10. Analyst Notes

This finding is a good illustration of why "review the exclusion list" belongs on the checklist for every Conditional Access policy, not just the ones that feel risky. The Entra portal's default behavior — quietly suggesting the currently signed-in admin for exclusion — exists for a legitimate reason (protecting against self-lockout while a policy is still being tuned), but it's easy to let that default survive past the testing phase if you don't explicitly check for it before calling a policy done. The same pattern showed up independently on two separate policies (CA01 and CA02) built in this same lab, which suggests it's worth treating as a standing checklist item for the remaining Conditional Access modules (Legacy Auth Block, Admin Role Protection) rather than something to catch ad hoc each time.

Testing note: the plan for this module originally called for reviewing results in the Conditional Access **Insights and reporting** workbook. That workbook requires a Log Analytics workspace wired up via diagnostic settings, which this tenant does not yet have configured. Rather than skip validation, testing was done directly against raw **Sign-in logs**, using the per-sign-in **Report-only** and **Conditional Access** tabs to confirm policy evaluation results — a valid substitute for this stage of the lab, though wiring up Log Analytics/diagnostic settings is a reasonable follow-up for a future module focused on monitoring and logging.

---

*Report generated as part of IAM/PAM lab work. Environment is a controlled lab simulation (Entra ID P2 trial tenant), not a production system.*

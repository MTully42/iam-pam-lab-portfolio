# Lab 02: Conditional Access — MFA, Risk-Based Sign-In, Legacy Auth Block, and Break-Glass SOP

## Overview

This lab builds a layered Conditional Access posture in Microsoft Entra ID: a documented break-glass emergency-access SOP, a tenant-wide MFA enforcement policy, and a risk-based sign-in control powered by Entra ID P2 Identity Protection — each built in report-only mode first, tested against real sign-in activity, then moved to live enforcement.

**Environment:** Microsoft Entra ID (P2 trial for Modules 1, 2 and 5; P1 trial from November 2026), single tenant, cloud-only (no on-prem AD sync)
**Tools used:** Microsoft Entra admin center (Conditional Access, Identity Protection, Sign-in logs), Microsoft Authenticator
**AI-assisted workflow:** Used Claude throughout to plan the policy build sequence, review exclusion lists for misconfigurations before finalizing each policy, validate test results against sign-in logs, and structure this documentation.

**Status:** Modules 1, 2, 3, and 5 complete and enforced. Module 4 (Admin Role Protection) is built and in Report-only, pending validation. Modules 6 (cross-policy testing/What-If validation) and 7 (consolidated sign-in log analysis) are planned — this repo reflects the tenant's actual current state, not a finished end state.

---

## Objective

Build and document a Conditional Access environment that mirrors what an Identity Access Engineer/Administrator would configure in a real organization — including catching and fixing real misconfigurations along the way, with evidence suitable for technical review and compliance audit.

---

## Build Log

### Module 1: Break-Glass Emergency Access Accounts

Created two cloud-only emergency access accounts (`BreakGlass-Emergency_Access-01`, `BreakGlass-Emergency_Access-02`) per Microsoft's recommended minimum, each assigned **Global Administrator** as a permanent **active** assignment — deliberately not PIM-eligible, since an emergency account that requires an activation step to use during an actual emergency defeats its purpose. Both accounts are excluded from every Conditional Access policy built in this lab, and their sign-in activity is reviewed via native Sign-in logs as the monitoring control.

**Evidence:**

![BreakGlass-01 — permanent Global Administrator assignment](./01_breakglass_account/01-1_breakglass01_assigned_roles.png.png)

![BreakGlass-02 — permanent Global Administrator assignment](./01_breakglass_account/01-2_breakglass02_assigned_roles.png.png)

![BreakGlass-01 — Authenticator registration](./01_breakglass_account/02-1-Breakglass1_Authenticator.png)

![BreakGlass-02 — Authenticator registration](./01_breakglass_account/02-2-Breakglass2_Authenticator.png)

![BreakGlass-01 — sign-in log monitoring](./01_breakglass_account/03-1_BreakGlass1_signin_logs.png)

![BreakGlass-02 — sign-in log monitoring](./01_breakglass_account/03-2_Breakglass2_signin_logs.png)

**Skill demonstrated:** Emergency access account design that survives the failure of the very control it's meant to bypass — permanent (not JIT) assignment, CA-exclusion by design, and monitored via sign-in logs.

**Design note:** Both break-glass accounts have an Authenticator method registered. This is a deliberate defense-in-depth choice, not an oversight — but it's worth being explicit about the trade-off: classic break-glass guidance keeps these accounts MFA-free specifically so they still work if the MFA provider itself is down. Registering a method adds a layer of protection against credential-only compromise, at the cost of depending on Authenticator availability during a true emergency. Documenting the reasoning here rather than leaving it silent.

---

### Module 2: MFA Enforcement Policy (CA01)

Built `CA01 - Require MFA All Users`: Include all users, exclude both break-glass accounts, target all cloud apps, grant control Require Multifactor Authentication. Built in Report-only, tested, then enforced.

**What happened:** Before finalizing the policy, a review of the exclusion list showed **3 excluded users** instead of the expected 2 — the tenant's primary Global Administrator account had been included in the exclusion list by the Entra portal's default "don't lock yourself out" safety suggestion. That default is reasonable while a policy is still being tuned, but leaving it in place on a finished, enforced policy would have meant the tenant's most privileged identity was permanently exempt from MFA. The admin exclusion was removed, leaving only the two break-glass accounts excluded. Full finding writeup: [`standing-admin-excluded-from-ca-mfa-policy.md`](./02_mfa_enforcement_policy/standing-admin-excluded-from-ca-mfa-policy.md) ([.docx version](./02_mfa_enforcement_policy/standing-admin-excluded-from-ca-mfa-policy.docx)).

Testing was done against a non-privileged test account (`alyssa.morgan@...`) rather than the Insights and reporting workbook, since that workbook requires a Log Analytics workspace not yet configured in this tenant — raw Sign-in logs were used instead, checking the per-sign-in **Report-only** and **Conditional Access** tabs.

**Evidence:**

![CA01 built in Report-only — exclusion count shows the bug (3 users) before the fix](./02_mfa_enforcement_policy/01-1_CA01_Policy_Created_ReportOnly.png)

![Client apps condition confirmed Not Configured — no unintended scope narrowing](./02_mfa_enforcement_policy/02-1_CA01_ClientApps_Verified.png)

![Authenticated as test user Alyssa Morgan for policy validation](./02_mfa_enforcement_policy/03-1_CA01_Alyssa_Signin_Confirmed.png)

![Sign-in logs filtered to the test user](./02_mfa_enforcement_policy/03-2_CA01_SigninLogs_Alyssa_Filtered.png)

![Report-only result: "User action required" — policy correctly identified an account with no MFA registered](./02_mfa_enforcement_policy/03-3_CA01_ReportOnly_Result_Alyssa.png)

![Portal confirmation: CA01 successfully updated and enabled, admin exclusion removed](./02_mfa_enforcement_policy/04-1_CA01_Enabled_Confirmation.png)

![Live enforcement: test user interrupted with mandatory MFA registration challenge](./02_mfa_enforcement_policy/04-2_CA01_Live_MFA_Challenge_Alyssa.png)

![Microsoft Authenticator QR registration for the test identity](./02_mfa_enforcement_policy/04-3_CA01_Authenticator_Registration.png)

![Registration confirmed successful — test user now fully subject to live MFA enforcement](./02_mfa_enforcement_policy/04-4_CA01_Authenticator_Registration_Successful.png)

**Skill demonstrated:** Catching a real policy misconfiguration through disciplined pre-finalization review, not luck; end-to-end validation using report-only mode → live sign-in log evidence → live enforcement test, rather than assuming a saved policy works as intended.

---

### Module 3: Legacy Authentication Block (CA03)

Built `CA03 - Block Legacy Authentication`: Include all users, exclude both break-glass accounts, client apps condition limited to **Exchange ActiveSync** and **Other clients** (POP, IMAP, SMTP AUTH, older Office clients), grant control **Block access**. These protocols can't perform MFA, which makes them the standard route for password spray against Microsoft 365.

**What happened:** The policy was built on 2026-08-31 and switched On. A configuration audit on 2026-11-03 found that **Target resources had never been set**. A Conditional Access policy with no target resources never matches a sign-in, so CA03 reported "On" for 64 days while enforcing nothing. Sign-in log evaluation details confirmed it: the policy stopped at the resource check (**Resource: Not matched**) and never reached the client app condition.

The August build evidence had already captured the problem. The creation screenshot shows "Included resources: No resources," and the August Report-only test showed CA03 absent from the results entirely. That absence was read as "no legacy authentication detected" when it was the symptom. The fix was to set Target resources to **All resources**, stage in Report-only, confirm a fresh sign-in now passes the resource check and is decided on client app, then return to On. Full finding writeup: [`ca03-legacy-auth-policy-enforced-nothing.md`](./03_legacy_auth_block_policy/ca03-legacy-auth-policy-enforced-nothing.md) ([.docx version](./03_legacy_auth_block_policy/ca03-legacy-auth-policy-enforced-nothing.docx)).

**Evidence:**

![Finding: Target resources set to None](./03_legacy_auth_block_policy/01-3_FINDING_target_resources_none.png)

![Policy summary: State On, Included resources: No resources](./03_legacy_auth_block_policy/03-1_policy_details_block_access_no_resources.png)

![Pre-fix sign-in log: CA03 evaluation stopped at Resource — Not matched](./03_legacy_auth_block_policy/06-0b_FINDING_signin_1738_resource_not_matched.png)

![After fix: Included resources = All resources, staged in Report-only](./03_legacy_auth_block_policy/05-3_policy_details_all_resources_report_only.png)

![Post-fix sign-in log: Resource Matched, Client app (Browser) Not matched — policy now fully evaluated](./03_legacy_auth_block_policy/06-1_signin_1820_resource_matched_clientapp_not_matched_REDACTED.png)

![CA03 returned to On](./03_legacy_auth_block_policy/07-1_ca03_enforced_on.png)

All 15 screenshots, plus the original August build evidence, are in [`03_legacy_auth_block_policy/`](./03_legacy_auth_block_policy/).

**Skill demonstrated:** Auditing a deployed control instead of trusting its status column; using per-policy evaluation details in sign-in logs to prove *why* a policy did or didn't apply; staging a fix through Report-only before enforcement; and owning a missed test result in the original build.

---

### Module 5: Risk-Based Sign-In Block (CA02)

Built `CA02 - Risk_Based Sign-In Block`, requiring Entra ID P2 (Identity Protection). Include all users, exclude break-glass accounts, condition on sign-in risk level (High), grant control Require Multifactor Authentication.

**What happened:** Attempting to enable the policy surfaced a portal prompt: *"It looks like you're about to manage your organization's security configurations... you must first disable security defaults before enabling a Conditional Access policy."* Security defaults and custom Conditional Access policies are mutually exclusive in Entra ID — the tenant's baseline security defaults were disabled so the custom risk-based policy could take over that responsibility with more granular control. The same admin-exclusion pattern found in Module 2 was found and fixed here too: the exclusion list showed 3 users instead of 2, with the Global Administrator account included; it was removed, leaving only the break-glass accounts excluded.

**Evidence:**

![New CA02 policy — exclusion list before the fix, security defaults conflict warning visible](./05_risk_based_signin_policy/01-1_Policy_Exlusions.png)

![Exclusion list expanded](./05_risk_based_signin_policy/01-2_Policy_Exclusions_Shown.png)

![Sign-in risk condition configured (High risk level)](./05_risk_based_signin_policy/02-1_Sign-In_Risk_Condition.png)

![CA02 successfully created, security defaults disabled](./05_risk_based_signin_policy/03-1_Successfully_Created_CA02_Policy_And_Disbled_Security_Defaults.png)

![Policy confirmed in the Conditional Access policies list](./05_risk_based_signin_policy/03-1_Policy_Confirmed_CA02.png)

![CA02 enabled](./05_risk_based_signin_policy/04-1_CA02_Policy_enabled.png)

![CA02 update confirmed successful, admin exclusion removed](./05_risk_based_signin_policy/04-2_CA02_Policy_Success.png)

**Skill demonstrated:** Understanding the security-defaults-vs-Conditional-Access relationship in Entra ID (not just clicking through an error); applying the same exclusion-review discipline consistently across multiple policies rather than treating the Module 2 finding as a one-off.

---

## Findings Report — NIST 800-53 Control Mapping

**Environment:** iampamprointheknow42gmail.onmicrosoft.com (Entra ID tenant)
**Scope:** Conditional Access — MFA enforcement, risk-based sign-in, legacy authentication block, break-glass SOP
**Date:** August 25, 2026 (Finding 3 added November 3, 2026)

---

**Finding 1 — Standing administrator account excluded from MFA enforcement and risk-based sign-in policies**
*Control: NIST 800-53 IA-2 (Identification and Authentication), AC-6 (Least Privilege)*
*Risk Rating: High (lab/dev tenant; would be Critical in production if left unaddressed past initial build)*

The tenant's primary Global Administrator account was found in the exclusion list of both `CA01 - Require MFA All Users` and `CA02 - Risk_Based Sign-In Block`, a result of the Entra portal's default "don't lock yourself out" suggestion during policy creation. Left in place, this would have meant the tenant's highest-privilege identity was permanently exempt from the two controls meant to protect it. Caught during pre-finalization review on both policies; the admin exclusion was removed from both, leaving only the two dedicated, monitored break-glass accounts excluded. Full technical writeup, business impact, and remediation steps: [`standing-admin-excluded-from-ca-mfa-policy.md`](./02_mfa_enforcement_policy/standing-admin-excluded-from-ca-mfa-policy.md).

**Finding 2 — Break-glass accounts registered with an MFA method**
*Control: NIST 800-53 IA-5 (Authenticator Management), CP-9 (System Backup — access continuity)*
*Risk Rating: Informational*

Both break-glass emergency access accounts have a Microsoft Authenticator method registered. This is a defense-in-depth design choice worth documenting explicitly rather than leaving implicit: it protects against credential-only compromise of the break-glass accounts, but introduces a dependency on Authenticator availability during the exact kind of outage break-glass accounts exist to survive. No change made in this pass; flagged for a deliberate decision (keep as-is with documented rationale, or remove the MFA method and rely on credential strength + monitoring alone) in a future review.

**Finding 3 — Legacy authentication block policy showed "On" but enforced nothing**
*Control: NIST 800-53 IA-2 (Identification and Authentication), CM-6 (Configuration Settings), CA-7 (Continuous Monitoring)*
*Risk Rating: High*

`CA03 - Block Legacy Authentication` was deployed with no target resources, so it matched no sign-ins and left legacy-protocol access (which can't perform MFA) unblocked from 2026-08-31 to 2026-11-03 while reporting a state of On. The defect was visible in the original build screenshot and in a Report-only test result that was misread at the time. Found during a configuration audit, confirmed in sign-in log evaluation details, remediated through Report-only, verified with a fresh sign-in, and re-enforced the same day. Full technical writeup, business impact, and remediation steps: [`ca03-legacy-auth-policy-enforced-nothing.md`](./03_legacy_auth_block_policy/ca03-legacy-auth-policy-enforced-nothing.md).

---

## Skills Summary

| Area | Demonstrated |
|---|---|
| Emergency Access | Break-glass SOP — permanent (non-PIM) admin assignment, CA-exclusion by design, sign-in monitoring |
| Conditional Access | Tenant-wide MFA enforcement, risk-based sign-in blocking, report-only → enforced lifecycle |
| Identity Protection | Sign-in risk conditions, security-defaults-to-custom-CA migration |
| Quality Assurance | Caught and remediated a real privileged-access misconfiguration on two separate policies through disciplined exclusion-list review; audited a deployed policy and found it enforced nothing |
| Validation | Sign-in log-based testing methodology (substituted for the Insights workbook, which requires Log Analytics not yet configured) |
| Compliance | NIST 800-53 control mapping across three findings (two substantive, one informational/documented trade-off) |
| AI-augmented workflow | Used Claude for policy build guidance, exclusion-list review, test validation, and documentation throughout |

---

## Notes

This is a working lab in an active trial tenant — screenshots reflect the actual build process, including the admin-exclusion misconfiguration found and fixed on two separate policies, and the Insights-workbook-to-Sign-in-logs substitution made when the planned validation path wasn't available. Left them in deliberately: catching your own mistakes before they ship, and knowing when a documented substitute approach is valid, is part of the job. Module 3 adds a second lesson: a Report-only test where the policy doesn't appear in the results at all is a failed test, not a clean one. Every remaining module now requires the policy to show up in sign-in evaluation results before it counts as verified. Modules 4, 6, and 7 are next.

# Lab 02: Conditional Access — MFA, Risk-Based Sign-In, Legacy Auth Block, Admin Role Protection, and Break-Glass SOP

## Overview

This lab builds a layered Conditional Access posture in Microsoft Entra ID: a documented break-glass emergency-access SOP, a tenant-wide MFA enforcement policy, and a risk-based sign-in control powered by Entra ID P2 Identity Protection — each built in report-only mode first, tested against real sign-in activity, then moved to live enforcement.

**Environment:** Microsoft Entra ID (P2 trial for Modules 1, 2 and 5; P1 trial from November 2026), single tenant, cloud-only (no on-prem AD sync)
**Tools used:** Microsoft Entra admin center (Conditional Access, Identity Protection, Sign-in logs), Microsoft Authenticator
**AI-assisted workflow:** Used Claude throughout to plan the policy build sequence, review exclusion lists for misconfigurations before finalizing each policy, validate test results against sign-in logs, and structure this documentation.

**Status:** Modules 1 through 7 complete. All four Conditional Access policies (CA01–CA04) are On; CA01, CA03, and CA04 are verified enforcing against sign-in logs. CA02 is inactive while the tenant is on Entra ID P1 (see Finding 4). Module 7's log review surfaced Finding 5 (retired admin account still active), remediated the same day. This repo reflects the tenant's actual current state, not a finished end state.

---

## Contents

- [Objective](#objective)
- [Build Log](#build-log)
  - [Module 1: Break-Glass Emergency Access Accounts](#module-1-break-glass-emergency-access-accounts)
  - [Module 2: MFA Enforcement Policy (CA01)](#module-2-mfa-enforcement-policy-ca01)
  - [Module 3: Legacy Authentication Block (CA03)](#module-3-legacy-authentication-block-ca03)
  - [Module 4: Admin Role Protection (CA04)](#module-4-admin-role-protection-ca04)
  - [Module 5: Risk-Based Sign-In Block (CA02)](#module-5-risk-based-sign-in-block-ca02)
  - [Module 6: Cross-Policy Testing & Validation](#module-6-cross-policy-testing--validation)
  - [Module 7: Sign-In Logs Analysis](#module-7-sign-in-logs-analysis)
- [Findings Report — NIST 800-53 Control Mapping](#findings-report--nist-800-53-control-mapping)
  - [Finding 1 — Standing administrator account excluded from MFA enforcement and risk-based sign-in policies](#finding-1--standing-administrator-account-excluded-from-mfa-enforcement-and-risk-based-sign-in-policies)
  - [Finding 2 — Break-glass accounts registered with an MFA method](#finding-2--break-glass-accounts-registered-with-an-mfa-method)
  - [Finding 3 — Legacy authentication block policy showed "On" but enforced nothing](#finding-3--legacy-authentication-block-policy-showed-on-but-enforced-nothing)
  - [Finding 4 — Risk-based sign-in policy inactive after Entra ID P2 license lapse](#finding-4--risk-based-sign-in-policy-inactive-after-entra-id-p2-license-lapse)
  - [Finding 5 — Retired personal Global Admin account still active and signing in](#finding-5--retired-personal-global-admin-account-still-active-and-signing-in)
- [Skills Summary](#skills-summary)
- [Notes](#notes)

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

[↑ Back to contents](#contents)

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

[↑ Back to contents](#contents)

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

[↑ Back to contents](#contents)

---

### Module 4: Admin Role Protection (CA04)

Built `CA04 - Protect Privileged Roles`: Include the three directory roles that can take over or reconfigure the tenant — **Global Administrator, Privileged Role Administrator, and Security Administrator** — exclude both break-glass accounts, target all resources, grant control **Require multifactor authentication**, session control **Sign-in frequency: every 4 hours**. Created 2026-08-31 in Report-only; audited, validated, and enforced 2026-11-04.

**Audit:** Every tab was checked against the design before enforcement, applying the lesson from Module 3. Target resources was set to All resources (an August build screenshot taken mid-creation showed "No target resources selected," but it was set before the policy was saved). Exclusions are limited to the two break-glass accounts.

**Validation:** CA04 ran in Report-only from 2026-08-31, so it was evaluated on every sign-in. The per-policy evaluation detail on a Global Administrator sign-in showed **User: Matched** (via role), **Resource: Matched**, **Grant: Satisfied**, **Session: Enforced**, and **Result: Report-only: Success** — meaning enforcement would allow a properly authenticated admin rather than lock them out. After enforcement, the next admin sign-in shows CA04 **Success** with sign-in frequency applied, alongside CA01 Success and CA02/CA03 correctly Not Applied.

**Design decisions:**
- **No compliant-device requirement.** The original build plan called for Require MFA + Require compliant device. This tenant is cloud-only with no Intune and no registered devices, so that control could never be satisfied — enforcing it would have locked out every admin except the break-glass accounts. MFA plus a 4-hour sign-in frequency is the strongest control this tenant can actually honor. Device compliance (or phishing-resistant authentication strength) is the next step once device management exists.
- **Admin stays in scope.** When enabling the policy, the Entra portal pre-selects **"Exclude current user"** in its lockout warning. That default was declined in favor of "I understand that my account will be impacted" — accepting it would have silently recreated the standing-admin exclusion found in Module 2.

**Evidence:**

![CA04 policy summary: 3 roles, All resources, Require MFA + Sign-in frequency, 2 break-glass exclusions](./04_admin_role_protection_policy/00-1_policy_details_summary.png)

![Directory roles included — Global Administrator](./04_admin_role_protection_policy/01-1b_roles_global_administrator.png)

![Directory roles included — Privileged Role Administrator and Security Administrator](./04_admin_role_protection_policy/01-1c_roles_privileged_role_and_security_admin.png)

![Exclusions limited to the two break-glass accounts](./04_admin_role_protection_policy/01-2_users_excluded_breakglass_only.png)

![Grant control: Require multifactor authentication](./04_admin_role_protection_policy/03-1_grant_require_mfa.png)

![Session control: Sign-in frequency every 4 hours](./04_admin_role_protection_policy/03-2_session_signin_frequency_4h.png)

![Report-only validation: User Matched, Resource Matched, Grant Satisfied, Session Enforced — Report-only: Success](./04_admin_role_protection_policy/04-1_signin_1820_ca04_reportonly_success_REDACTED.png)

![All four policies On](./04_admin_role_protection_policy/05-2_all_four_policies_on.png)

![Post-enforcement sign-in: CA01 and CA04 Success, CA02 and CA03 correctly Not Applied](./04_admin_role_protection_policy/06-1_signin_1104_all_four_policies_enforced.png)

All screenshots are in [`04_admin_role_protection_policy/`](./04_admin_role_protection_policy/).

**Skill demonstrated:** Scoping a policy to privileged directory roles rather than named users; adapting a build plan to what the environment can actually enforce instead of deploying an unsatisfiable control; validating with per-policy evaluation detail before enforcement; and recognizing a portal default that would have reintroduced a known finding.

[↑ Back to contents](#contents)

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

[↑ Back to contents](#contents)

**Status update (2026-11-04):** CA02 depends on Microsoft Entra ID Protection sign-in risk, a P2 feature. After the P2 trial ended and the tenant moved to P1, CA02 still shows On but cannot evaluate sign-in risk. See Module 6 and Finding 4.

---

### Module 6: Cross-Policy Testing & Validation

Tested all four policies together using the Conditional Access **What If** tool, then confirmed key results with a real sign-in. Every What If test used the same target app (Office 365 Exchange Online) and device platform (Windows), changing **one variable at a time**, so each result can be attributed to that variable.

| # | Scenario | Expected | Result |
|---|---|---|---|
| 1 | Standard user (Alyssa), browser | CA01 only | ✅ CA01 applies. CA02 skipped (sign-in risk), CA03 skipped (client app), CA04 skipped (users and groups) |
| 2 | Standard user, **legacy client** ("Other clients": IMAP/POP/SMTP) | CA03 Block | ✅ CA01 + **CA03 Block access** apply. Only the client app changed from Test 1. |
| 3 | **Admin** (Global Administrator), browser | CA01 + CA04 | ✅ CA01 + **CA04** (with 4-hour sign-in frequency) apply. Only the user changed from Test 1. |
| 4 | Standard user, **sign-in risk: High** | CA02 | ⚠️ **Could not run.** What If offers no sign-in risk field without P2. This led to Finding 4. |
| 5 | **Break-glass account**, browser | Nothing | ✅ 0 policies apply. All four skipped for "Users and groups" (excluded). |
| B | **Real sign-in**, standard user (Alyssa) | CA01 Success, CA04 Not Applied | ✅ CA01 Success (MFA enforced); CA04 Not Applied, **User: Not matched**; CA02, CA03 Not Applied |

**Side result:** Test 2 also closed an open question about CA01. Its summary page reads "Client apps: 1 included," but CA01 applied to a legacy client in What If, which confirms it covers all client types. "1 included" is how the summary page displays an unconfigured client-app condition.

**Testing approach notes:** The original plan included the Conditional Access Insights workbook, which requires a Log Analytics workspace not configured in this tenant. Per-sign-in evaluation details in Sign-in logs were used instead, as in Modules 2–4. Following the Module 3 lesson, a test only passed if the policy appeared in the evaluation results for the expected reason. A missing policy counted as a failure, not a pass.

**Evidence:**

![Test 2 inputs: Alyssa Morgan, Exchange Online](./06_policy_testing_validation/02-0a_test2_inputs_user_alyssa.png)

![Test 2 conditions: Windows, "Other clients" (legacy)](./06_policy_testing_validation/02-0b_test2_inputs_conditions_other_clients.png)

![Test 2 result: CA03 Block access applies to the legacy client](./06_policy_testing_validation/02-1_test2_alyssa_other_clients_will_apply_ca03_block.png)

![Test 3 result: CA01 and CA04 (with sign-in frequency) apply to the admin](./06_policy_testing_validation/03-1_test3_admin_browser_will_apply_ca01_ca04.png)

![Test 5 result: break-glass account — 0 policies apply](./06_policy_testing_validation/05-1_test5_breakglass_will_apply_none.png)

![Test 5: all four policies skipped for "Users and groups" (excluded)](./06_policy_testing_validation/05-2_test5_breakglass_will_not_apply_all_excluded.png)

![Real sign-in (Alyssa): CA01 Success; CA02, CA03, CA04 Not Applied](./06_policy_testing_validation/06-1_alyssa_real_signin_ca_tab_ca01_success.png)

![Real sign-in (Alyssa): CA04 evaluation — User Not matched](./06_policy_testing_validation/06-2_alyssa_real_signin_ca04_user_not_matched.png)

All 17 test screenshots (inputs and both result tabs for each test) are in [`06_policy_testing_validation/`](./06_policy_testing_validation/). Finding 4 writeup: [`ca02-risk-policy-inactive-without-p2.md`](./06_policy_testing_validation/ca02-risk-policy-inactive-without-p2.md) ([.docx version](./06_policy_testing_validation/ca02-risk-policy-inactive-without-p2.docx)).

**Skill demonstrated:** Designing a controlled test matrix (one variable per test) that proves each policy applies to who it should *and* skips who it shouldn't; confirming simulation results with a real sign-in; and treating a test that couldn't run as evidence to investigate rather than a step to skip.

[↑ Back to contents](#contents)

---

### Module 7: Sign-In Logs Analysis

Pulled 30 days of sign-in logs (Download → JSON, interactive and non-interactive, **507 sign-ins**) and parsed each sign-in's per-policy Conditional Access result. The JSON export was used instead of CSV because only the JSON includes `appliedConditionalAccessPolicies`, the per-policy result for every sign-in. The raw export contains IP addresses and locations and stays off GitHub. The sanitized dataset and summary are in [`Lab02_CA_Policy_Matrix.xlsx`](./Lab02_CA_Policy_Matrix.xlsx).

| Policy | Evaluated | Success | Not applied | Report-only | What the logs show |
|---|---|---|---|---|---|
| CA01 Require MFA | 482 | 482 | 0 | – | Applied to every evaluated sign-in. The only CA failures in the period were three MFA prompts not completed on 2026-11-03. |
| CA02 Risk-based | 482 | 0 | 482 | – | Never applied once (no P2 risk signal). Confirms Finding 4. |
| CA03 Legacy auth | 482 | 0 | 410 | 72 not applied | No legacy-auth traffic at all, so there was nothing to block. Briefly in Report-only 2026-11-03 23:06 → 11-04 02:27 UTC, then back On. |
| CA04 Admin roles | 482 | 141 | 14 | 327 success | Report-only until 2026-11-04 17:54 UTC, then enforced. Never matched the standard user. **Matched the retired personal admin account → Finding 5.** |

25 sign-ins were never evaluated by Conditional Access: wrong passwords, a disabled Microsoft service (error 500014), and MFA security-info registration. CA cannot run on a sign-in that fails before authentication completes, so none of these is a bypass.

**Limits, stated plainly:** there were no lab sign-ins between 2026-10-05 and 2026-11-03, so the planned CA03 before/after comparison isn't possible from logs, and CA03's effectiveness rests on Module 6 Test 2. The three Nov 3 "Interrupted" sign-ins in 07-1 don't appear in the JSON download; the screenshots are the evidence for them.

**Evidence:**

![Sign-in logs filtered to Conditional Access failures, last month: three failures, all 2026-11-03](./07_signin_logs_analysis/07-1_signin_logs_ca_failure_filter.png)

![Retired personal admin, 21:07: CA01 matched, MFA required, not completed](./07_signin_logs_analysis/07-2a_ca01_failure_policy_detail.png)

![Retired personal admin, 21:14: same correlation ID, second attempt](./07_signin_logs_analysis/07-2b_basic_info.png)

![New cloud admin's first sign-in, 21:34: sent to MFA registration; display name identical to retired account](./07_signin_logs_analysis/07-2c_basic_info.png)

All eleven screenshots (seven log evidence, four remediation) are in [`07_signin_logs_analysis/`](./07_signin_logs_analysis/). Finding 5 writeup: [`retired-personal-admin-still-active.md`](./07_signin_logs_analysis/retired-personal-admin-still-active.md) ([.docx version](./07_signin_logs_analysis/retired-personal-admin-still-active.docx)).

**Skill demonstrated:** Turning raw sign-in telemetry into a per-policy effectiveness summary, separating "the control worked" from "the control had nothing to act on," stating data gaps instead of overstating results, and finding a lifecycle gap that no configuration screen showed.

[↑ Back to contents](#contents)

---

## Findings Report — NIST 800-53 Control Mapping

**Environment:** iampamprointheknow42gmail.onmicrosoft.com (Entra ID tenant)
**Scope:** Conditional Access — MFA enforcement, risk-based sign-in, legacy authentication block, break-glass SOP
**Date:** August 25, 2026 (Finding 3 added November 3, 2026; Finding 4 added November 4, 2026; Finding 5 added November 5, 2026)

---

### Finding 1 — Standing administrator account excluded from MFA enforcement and risk-based sign-in policies
*Control: NIST 800-53 IA-2 (Identification and Authentication), AC-6 (Least Privilege)*
*Risk Rating: High (lab/dev tenant; would be Critical in production if left unaddressed past initial build)*

The tenant's primary Global Administrator account was found in the exclusion list of both `CA01 - Require MFA All Users` and `CA02 - Risk_Based Sign-In Block`, a result of the Entra portal's default "don't lock yourself out" suggestion during policy creation. Left in place, this would have meant the tenant's highest-privilege identity was permanently exempt from the two controls meant to protect it. Caught during pre-finalization review on both policies; the admin exclusion was removed from both, leaving only the two dedicated, monitored break-glass accounts excluded. Full technical writeup, business impact, and remediation steps: [`standing-admin-excluded-from-ca-mfa-policy.md`](./02_mfa_enforcement_policy/standing-admin-excluded-from-ca-mfa-policy.md).

### Finding 2 — Break-glass accounts registered with an MFA method
*Control: NIST 800-53 IA-5 (Authenticator Management), CP-9 (System Backup — access continuity)*
*Risk Rating: Informational*

Both break-glass emergency access accounts have a Microsoft Authenticator method registered. This is a defense-in-depth design choice worth documenting explicitly rather than leaving implicit: it protects against credential-only compromise of the break-glass accounts, but introduces a dependency on Authenticator availability during the exact kind of outage break-glass accounts exist to survive. No change made in this pass; flagged for a deliberate decision (keep as-is with documented rationale, or remove the MFA method and rely on credential strength + monitoring alone) in a future review.

### Finding 3 — Legacy authentication block policy showed "On" but enforced nothing
*Control: NIST 800-53 IA-2 (Identification and Authentication), CM-6 (Configuration Settings), CA-7 (Continuous Monitoring)*
*Risk Rating: High*

`CA03 - Block Legacy Authentication` was deployed with no target resources, so it matched no sign-ins and left legacy-protocol access (which can't perform MFA) unblocked from 2026-08-31 to 2026-11-03 while reporting a state of On. The defect was visible in the original build screenshot and in a Report-only test result that was misread at the time. Found during a configuration audit, confirmed in sign-in log evaluation details, remediated through Report-only, verified with a fresh sign-in, and re-enforced the same day. Full technical writeup, business impact, and remediation steps: [`ca03-legacy-auth-policy-enforced-nothing.md`](./03_legacy_auth_block_policy/ca03-legacy-auth-policy-enforced-nothing.md).

### Finding 4 — Risk-based sign-in policy inactive after Entra ID P2 license lapse
*Control: NIST 800-53 SI-4 (System Monitoring), CA-7 (Continuous Monitoring), CM-3 (Configuration Change Control)*
*Risk Rating: Medium (compensated by CA01's universal MFA requirement; would be High in a tenant without it)*

`CA02 - Risk_Based Sign-In Block` relies on ID Protection sign-in risk, which requires Entra ID P2. After the P2 trial ended, CA02 continued to show On but could no longer evaluate sign-in risk: What If offers no risk field, and the policy's sign-in log evaluation shows no sign-in risk condition. A licensing change disabled a security control with no change to the control and no warning on the policy. Two design issues were also noted: the policy is named "Block" but grants Require MFA, and without sign-in frequency "Every time" it adds no challenge to a session that already completed MFA under CA01. P2 was not restored for the lab; CA02 was validated with P2 in Module 5. Full writeup: [`ca02-risk-policy-inactive-without-p2.md`](./06_policy_testing_validation/ca02-risk-policy-inactive-without-p2.md).

### Finding 5 — Retired personal Global Admin account still active and signing in
*Control: NIST 800-53 AC-2 (Account Management), AC-6(5) (Privileged Accounts), IA-2(1) (MFA for Privileged Accounts)*
*Risk Rating: High (MFA still enforced; no misuse observed)*

The original personal (Gmail) Global Admin was replaced by a native cloud admin on 2026-11-03 but never disabled. Sign-in logs show it signing into the Azure portal successfully about 20 minutes later. CA04, which is scoped only to admin roles, evaluated it, which proves it still holds an admin role. MFA through CA01 and CA04 is the only barrier between that unmonitored consumer account and full tenant control. Both admin accounts also share the display name "Mike Tully," so they look identical in log views. Remediated 2026-11-05: Global Admin removed, account disabled, sessions revoked, verified on the account page (Assigned roles 0, Disabled). During the fix, the duplicate display name nearly caused the role to be removed from the wrong account, so the UPN was confirmed first. Full writeup: [`retired-personal-admin-still-active.md`](./07_signin_logs_analysis/retired-personal-admin-still-active.md).

---

## Skills Summary

| Area | Demonstrated |
|---|---|
| Emergency Access | Break-glass SOP — permanent (non-PIM) admin assignment, CA-exclusion by design, sign-in monitoring |
| Conditional Access | Tenant-wide MFA enforcement, legacy auth blocking, privileged-role protection with sign-in frequency, risk-based sign-in blocking, report-only → enforced lifecycle |
| Identity Protection | Sign-in risk conditions, security-defaults-to-custom-CA migration |
| Quality Assurance | Caught and remediated a real privileged-access misconfiguration on two separate policies through disciplined exclusion-list review; audited a deployed policy and found it enforced nothing |
| Validation | What If test matrix across all four policies (one variable per test), confirmed with real sign-ins; per-policy sign-in log evaluation details (substituted for the Insights workbook, which requires Log Analytics not yet configured) |
| Log Analysis | 30-day sign-in export parsed per policy (507 sign-ins); policy matrix with live COUNTIF formulas over a sanitized dataset; found an account-lifecycle gap from telemetry |
| Compliance | NIST 800-53 control mapping across five findings (four substantive, one informational/documented trade-off) |
| AI-augmented workflow | Used Claude for policy build guidance, exclusion-list review, test validation, and documentation throughout |

---

## Notes

This is a working lab in an active trial tenant — screenshots reflect the actual build process, including the admin-exclusion misconfiguration found and fixed on two separate policies, and the Insights-workbook-to-Sign-in-logs substitution made when the planned validation path wasn't available. Left them in deliberately: catching your own mistakes before they ship, and knowing when a documented substitute approach is valid, is part of the job. Module 3 adds a second lesson: a Report-only test where the policy doesn't appear in the results at all is a failed test, not a clean one. Every remaining module now requires the policy to show up in sign-in evaluation results before it counts as verified. Module 6 found a second policy that showed On while protecting nothing — this time because of a license lapse, not a configuration error. Module 7 showed the same lesson from the account side: an admin account I considered retired was still signing in, and only the logs said so.

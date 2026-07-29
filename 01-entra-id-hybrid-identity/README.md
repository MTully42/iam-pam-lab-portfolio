Lab 01: Entra ID Hybrid Identity — Users, Groups, RBAC, PIM, SSO
Overview

This lab builds out a working Identity Governance and Administration (IGA) environment in Microsoft Entra ID, demonstrating core IAM patterns used in enterprise environments: bulk provisioning, dynamic group-based access, group-based licensing, least-privilege RBAC (including a custom role), Privileged Identity Management (PIM) for just-in-time access, SSO via SAML, JWT/SAML claims inspection, and audit log review mapped to NIST 800-53 controls.

Environment: Microsoft Entra ID (P2 trial), single tenant, cloud-only (no on-prem AD sync)
Tools used: Microsoft Entra admin center, Microsoft 365 admin center, Microsoft Graph Explorer, PowerShell / Microsoft Graph SDK, samltool.io, jwt.io
AI-assisted workflow: Used Claude throughout to plan the build sequence, generate bulk-provisioning data, troubleshoot portal and authentication errors, and structure this documentation — demonstrating practical use of AI to accelerate IAM/PAM lab development.

Objective

Build and document an IAM environment that mirrors what an Identity Access Engineer/Administrator would configure in a real organization, with evidence at each stage suitable for technical review and compliance audit.

Build Log
Step 1–3: Bulk User Provisioning

Created 32 test users across 4 departments (IT, Finance, HR, Sales — 8 users each) via CSV bulk import rather than manual one-by-one creation, reflecting how real onboarding pipelines work at scale.

What happened: First upload attempt failed with a generic "unexpected error." Diagnosed by pulling Microsoft's actual bulk-create template and comparing headers — the working template used Account enabled (true/false) with boolean values, not the Block sign-in (Yes/No) format from the older documentation. Rebuilt the CSV to match the tenant's actual schema and resubmission succeeded.

Evidence:

01_bulk_users/02_bulk_create_error.png — initial failure
01_bulk_users/03_bulk_create_success.png — corrected submission succeeding
01_bulk_users/04_all_users_33_confirmed.png — 33 users confirmed (32 provisioned + 1 admin)

Skill demonstrated: Bulk identity provisioning; troubleshooting schema mismatches between documentation and live tenant behavior.

Step 4–5: Security Groups & Dynamic Membership

Built 4 department-based security groups (SG-IT, SG-Finance, SG-HR, SG-Sales) and converted two (SG-Finance, SG-IT) to dynamic membership using rules based on the department attribute.

What happened: Caught and corrected a dynamic rule syntax issue before it silently under-populated a group — verified final state via the Entra admin center's All groups list view, which shows Membership type (Dynamic/Assigned) per group rather than relying on a summary tile.

Evidence:

02_dynamic_groups/ — dynamic rule builder showing rule syntax, and the All Groups list confirming SG-Finance/SG-IT as Dynamic and SG-HR/SG-Sales as Assigned

Skill demonstrated: Dynamic group configuration; verifying configuration state through the correct portal view rather than a summary that can mask per-object detail.

Step 6: Group-Based Licensing

Assigned Microsoft Entra ID P2 license at the group level (not per-user) across all 4 department groups, so license and feature access follow group membership rather than requiring manual per-user assignment.

Evidence:

03_licensing/ — group Licenses blade showing P2 assigned and inherited status

Skill demonstrated: Lifecycle-driven license management — the pattern real IGA platforms use instead of manual, error-prone per-user assignment.

Step 7: RBAC Matrix Design

Before touching any role configuration in the portal, mapped out a full role/permission/scope/justification matrix in Excel — 10 roles spanning all 4 departments plus a Global Admin baseline, each tagged with least-privilege rationale and PIM eligibility.

Deliverable: Lab01_RBAC_Matrix.xlsx — includes a Legend tab explaining each column, used as the working reference for Steps 8–9 below.

Skill demonstrated: Access control planning and documentation prior to implementation — the compliance-mapping habit auditors look for.

Step 8: Custom Role — IT Manager

Built a custom role (IT Manager Role) scoped to reset passwords and manage group membership, explicitly excluding Conditional Access and Global Admin rights — closing a gap where the nearest built-in role (User Administrator) was too broad for the intended use case.

Evidence:

04_rbac_custom_roles/ — custom role permissions/JSON editor view showing exactly what's scoped in and out

Skill demonstrated: Closing built-in role gaps with a purpose-scoped custom role — a concrete least-privilege example, not just a claim.

Step 9–10: PIM — Just-in-Time Privileged Access

Configured PIM on two roles from the RBAC matrix — Global Administrator and the custom IT Manager Role — requiring MFA and written justification on activation, with a defined maximum activation duration. Performed a live activation as a functional test and confirmed the resulting event in the PIM audit trail.

What happened: Activation succeeded on the first attempt (Stage 1 → 2 → 3, "Activation completed successfully"). Pulled the corresponding entry from Identity Governance → PIM → Microsoft Entra roles → My audit, which showed the full settings → eligibility → activation timeline with timestamps.

Evidence:

05_pim/01–04 — activation settings (both roles) and eligible assignment creation
05_pim/05 — activation success (Stage 3 confirmation)
05_pim/06 — pre-activation "My roles" eligible assignments view
05_pim/07 — second role (Global Administrator) activation settings saved
07_audit_logs/01_pim_activation_audit_entry — audit trail showing settings updates, eligible member assignment, and role activation events in sequence

Skill demonstrated: Just-in-time privileged access replacing standing admin rights; reading and correlating a PIM audit trail end-to-end.

Step 11: SSO / SAML Application Registration

Registered a non-gallery SAML application (IAM-Lab-SAML-Test) in Entra ID and configured Basic SAML Configuration (Entity ID, Reply URL/ACS).

What happened: Initial live sign-on test failed with AADSTS50011 (reply URL mismatch) against the placeholder localhost ACS endpoint — a correct security behavior on Entra's part, not a misconfiguration in the sense of a vulnerability. Reply URL was updated to a real reachable endpoint (httpbin.org/post) to attempt a full round-trip capture; the live capture was not completed in this pass due to authentication/session issues in the browser test flow, so claims interpretation (Step 12) was demonstrated against a reference SAML assertion instead.

Evidence:

06_sso_saml/01_create_custom_app_dialog — non-gallery app creation
06_sso_saml/02_saml_basic_configuration — Entity ID and Reply URL saved

Skill demonstrated: Non-gallery SAML app registration and configuration; understanding of ACS/reply-URL validation as a security control, not just a form field.

Step 12: SAML & JWT Claims Decoding

Decoded a reference SAML assertion and a sample JWT to document claims structure and demonstrate the ability to read and interpret federated identity tokens.

Evidence:

06_sso_saml/03_saml_decoded_sample_claims — parsed SAML Info panel: Issuer, Signature status (assertion signed, response unsigned), NameID, Conditions (NotBefore/NotOnOrAfter replay window), and Attribute claims
06_sso_saml/03-1_saml_html_sample — raw XML structure (top portion)
06_sso_saml/04, 04-1, 04-2 — JWT header, payload, and combined header+payload+signature-verification view with claim-by-claim explanation (sub, name, admin, iat)

Skill demonstrated: SAML/JWT claims interpretation, including the distinction between assertion-level and response-level signing, and awareness of token replay-window controls.

Step 13: Audit Log Review

Reviewed PIM role activation events via the Entra admin center's native audit history (Identity Governance → PIM → Microsoft Entra roles → My audit), confirming timestamped requestor, action, and scope for each activation event.

What happened: Attempted programmatic retrieval via Microsoft Graph Explorer and the Microsoft Graph PowerShell SDK (Get-MgAuditLogSignIn). Both attempts were blocked by local environment issues rather than tenant permissions — Graph Explorer's Run action did not return a response in-browser, and the PowerShell interactive sign-in defaulted to an unintended local Microsoft account and triggered a Windows Hello enrollment prompt rather than completing tenant authentication. Rather than burn further time on local auth tooling, audit evidence was captured through the portal instead, which is a fully valid and commonly used path for this kind of review.

Evidence:

07_audit_logs/01_pim_activation_audit_entry — audit trail entry (also referenced in Step 9–10)

Skill demonstrated: Native portal-based audit log review and interpretation. Follow-up item: complete a Graph API/PowerShell-based pull in a clean environment to add programmatic log retrieval to the evidence set.

Step 14: Findings Report

See Findings Report section below.

Findings Report — NIST 800-53 Control Mapping

Environment: iampamprointheknow42gmail.onmicrosoft.com (Entra ID tenant)
Scope: Identity governance, RBAC, PIM, SSO/SAML, JWT claims
Date: July 29, 2026

Finding 1 — Standing privileged access replaced with JIT activation
Control: NIST 800-53 AC-6 (Least Privilege), AC-2 (Account Management)

Global Administrator and the custom IT Manager role were configured for eligible (not permanent active) assignment via Microsoft Entra PIM, requiring MFA and written justification at activation time, with a defined maximum activation duration. This replaces standing privileged access with just-in-time elevation, reducing the attack surface of always-on admin credentials. Verified via successful activation test (audit log entry captured, timestamped).

Finding 2 — Custom role closes built-in role over-permissioning gap
Control: NIST 800-53 AC-6(1) (Least Privilege — Authorize Access to Security Functions)

Built-in Entra roles (e.g., User Administrator) grant broader permissions than required for routine password-reset and group-membership tasks, including access to Conditional Access configuration. A custom "IT Manager" role was built and scoped to exclude Conditional Access and Global Admin-level rights while retaining password reset and group management capability — a concrete least-privilege gap closure over relying on a built-in role.

Finding 3 — Group-based licensing and dynamic membership reduce manual provisioning error
Control: NIST 800-53 AC-2(1) (Automated System Account Management)

License assignment was configured at the security-group level rather than per-user, with dynamic membership rules driving group population by department attribute. This lifecycle-driven model reduces manual provisioning steps and the associated risk of orphaned or inconsistent license/access grants during joiner-mover-leaver events.

Finding 4 — SAML/SSO claims structure documented; live IdP-initiated test blocked by ACS endpoint mismatch
Control: NIST 800-53 IA-8 (Identification and Authentication — Non-Organizational Users), AC-4 (Information Flow Enforcement)

A non-gallery SAML application was registered and configured with Entity ID and Reply URL (ACS) values. Testing the live sign-on flow surfaced an AADSTS50011 reply URL mismatch, which is itself a realistic and correctly-behaving security control — Entra enforces exact match between the configured ACS URL and the URL received in the authentication request, preventing token redirection to unauthorized endpoints. A reference SAML assertion was decoded and its claims structure (NameID, Issuer, Audience, Attribute Statements, Conditions) documented to demonstrate claims interpretation capability. Recommendation: stand up a real ACS endpoint (e.g., a simple listener or public test URL) in a follow-up lab iteration to complete a full live round-trip test.

Finding 5 — Audit trail available via portal; programmatic Graph API access blocked by local authentication constraints
Control: NIST 800-53 AU-6 (Audit Review, Analysis, and Reporting), AU-12 (Audit Generation)

PIM role activation events are logged and retrievable via the Entra admin center's audit history (Identity Governance → PIM → My audit), confirmed with a captured entry showing timestamp, requestor, and action for the IT Manager role activation. Attempted programmatic retrieval via Microsoft Graph PowerShell SDK was blocked by local Windows authentication requirements (interactive browser sign-in defaulting to an unintended account, Windows Hello enforcement). Audit data remains fully accessible and reviewable through the native admin portal; Graph-based automation is a follow-up item rather than a current gap.

Skills Summary
Area	Demonstrated
Provisioning	Bulk user creation, CSV schema troubleshooting
Governance	Dynamic group membership, group-based licensing
RBAC	Least-privilege matrix design, custom role scoping
PAM	PIM just-in-time activation, approval/justification workflows, audit trail correlation
Federation	Non-gallery SAML app registration, SAML/JWT claims inspection and interpretation
Compliance	Audit log review, NIST 800-53 control mapping, 5-finding report
AI-augmented workflow	Used Claude for build planning, data generation, error diagnosis, and documentation throughout
Notes

This is a working lab in an active trial tenant — screenshots reflect the actual build process, including real obstacles and how they were diagnosed and worked through: a bulk-import schema mismatch, a dynamic rule syntax issue, a SAML reply-URL mismatch, and local authentication tooling issues that blocked a programmatic Graph API log pull. Left them in deliberately; troubleshooting and knowing when to pivot to an equally valid alternative approach is part of the job.

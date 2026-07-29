# IAM/PAM Lab Portfolio

Hands-on Identity and Access Management / Privileged Access Management lab environments covering Microsoft Entra ID, CyberArk, SailPoint, Saviynt, Okta, and AWS IAM. Each lab is self-built, documented with a findings report, and mapped to NIST 800-53 and SOX ITGC controls. AI-assisted workflows are used throughout to accelerate access reviews, orphaned-account detection, and compliance documentation.

Built to demonstrate production-relevant IAM/PAM skills for identity governance, privileged access, conditional access, and cloud IAM roles in regulated environments (finance, healthcare, utilities).

## Labs

| # | Lab | Platform | Outcome |
|---|-----|----------|---------|
| 01 | [Entra ID Hybrid Identity](./01-entra-id-hybrid-identity) | Microsoft Entra ID | Built hybrid identity environment across 30+ users with dynamic group membership, group-based licensing, a 10-role RBAC matrix with custom role, PIM just-in-time privileged access, and SAML/JWT claims inspection |
| 02 | [Conditional Access Policies](./02-conditional-access-policies) | Microsoft Entra ID | Deployed 4 CA policies (MFA, legacy auth block, admin protection, risk-based sign-in) with break-glass SOP |
| 03 | [Entra RBAC & PIM](./03-entra-rbac-pim) | Microsoft Entra ID | Designed 5-role RBAC matrix, custom roles, and PIM for JIT privileged access |
| 04 | [Saviynt IGA Onboarding](./04-saviynt-iga-onboarding) | Saviynt | Configured IGA identity repository, source connectors, and entitlement imports |
| 05 | [Saviynt Access Request Routing](./05-saviynt-access-request-routing) | Saviynt | Built and documented full application onboarding workflow |
| 06 | [CyberArk Safe Design](./06-cyberark-safe-design) | CyberArk PAS | Designed safe permission model, onboarded privileged accounts, automated CPM rotation |
| 07 | [CyberArk PSM Session Management](./07-cyberark-psm-session-mgmt) | CyberArk PAS | Configured session isolation/recording, dual control, and OTP workflows |
| 08 | [Entra Audit Log Analysis](./08-entra-audit-log-analysis) | Microsoft Entra ID | Pulled audit/sign-in logs via PowerShell/Graph API, produced 5-finding security report |
| 09 | [SailPoint JML Lifecycle](./09-sailpoint-jml-lifecycle) | SailPoint IdentityNow | Built source aggregation, access profiles, and JML lifecycle state automation |
| 10 | [SailPoint Access Certification](./10-sailpoint-access-cert-campaign) | SailPoint IdentityNow | Ran end-to-end access certification campaign, pulled data via REST API |
| 11 | [Okta Workforce Identity](./11-okta-workforce-identity) | Okta | Configured org admin, MFA policies, SSO integrations, and automated deprovisioning workflow |
| 12 | [AWS IAM Least Privilege](./12-aws-iam-least-privilege) | AWS IAM | Built least-privilege roles/policies, STS role assumption with MFA, IAM Identity Center federation |
| 13 | [AWS CloudTrail Analysis](./13-aws-cloudtrail-analysis) | AWS | Analyzed CloudTrail events to identify high-risk activity and privilege escalation paths |
| 14 | [AI-Assisted Audit Workflow](./14-ai-assisted-audit-workflow) | Cross-lab | Built AI-assisted reporting to auto-generate findings, flag orphaned/dormant accounts, draft compliance docs |
| 15 | [Lab VM Environment Setup](./15-lab-vm-environment-setup) | Infrastructure | Isolated Ubuntu VM setup used as the standard environment for all lab work |

## Certifications

- Microsoft Certified: Identity and Access Administrator Associate (SC-300)
- CyberArk Defender (PAS)
- SailPoint Certified IdentityNow Engineer
- Okta Certified Associate
- AWS Certified Cloud Practitioner
- Microsoft Certified: Azure Fundamentals (AZ-900)
- CISSP — In Progress

## Background

B.S., Cybersecurity — Norwich University (Cum Laude). 10+ years in network administration and live technical support, now focused full-time on IAM/PAM. Six IAM/PAM certifications earned in 90 days.

## Contact

Michael Tully — [LinkedIn](https://linkedin.com/in/michael-c-tully) — MTully423@outlook.com

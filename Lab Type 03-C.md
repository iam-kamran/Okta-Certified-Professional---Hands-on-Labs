# Okta Professional Performance Exam: Curriculum Design and Readiness Assessment

Prepared for Kamran Arif. Exam facts come from the official Okta Professional Performance Exam study guide (certification.okta.com/page/professional-performance-exam-study-guide). Anything not verified is marked as an assumption.

## 1. Verified exam facts

The exam has six performance-based hands-on use cases in 150 minutes, with no scheduled break. You can use the Okta Help Center during the exam, and you need Okta Verify on your phone for some tasks. This is the current Identity Engine (OIE) exam. The older Hands-On Configuration exam was discontinued June 1, 2024.

| Use case | Weight | Tasks | Priority |
| --- | --- | --- | --- |
| UC4 Security enforcement | 38% | Authenticator and enrollment policy, global session policy rule, authentication policy, password policy for self-service recovery, verify required authenticators | Exam-critical, highest |
| UC1 Account creation and user management | 26% | Create user, custom attribute, edit profile, group, manual assignment, group rule, standard admin role, custom role + resource set | Exam-critical |
| UC5 Troubleshooting | 12% | User can't sign in, expire password and clear sessions, app missing from dashboard, wrong group assignment | Exam-critical |
| UC2 App setup with OIN | 10% | OIN app via SAML SSO, provisioning to app, assign group, verify provisioning | Exam-critical |
| UC3 Attribute mapping and offboarding | 8% | Map attributes (Profile Editor and provisioning page), deactivate user and verify no app access | Exam-critical |
| UC6 Syslog and support | 6% | Find an event in System Log, identify support resources | Exam-critical |

Your original prompt covers more than the exam does. The task list contains nothing on Active Directory, Entra ID, AWS, HR-driven JML, SCIM APIs, or Workflows. Those are enterprise extensions: worth building for your career goals, but they shouldn't delay exam readiness. The guide mentions "simple directory integration" in its description of a Professional, but lists no AD task, so treat AD as a low-risk extension.

## 2. Sandbox constraints

The What's Next program gives grantees on-demand training, an Okta org, and certification vouchers for the remainder of their 90-day grant period. If that's your path, your sandbox may expire, so the plan is front-loaded: exam-critical work first, extensions after (or in a separate free org).

For extension work, a separate free org is an option. A third-party source says the Integrator Free Plan replaced the Developer Edition as the default free org in May 2025. Its feature limits (policies, AD agent, provisioning) are unverified, so check before relying on it.

### Lab 0 capability checks (verify in your org, don't assume)

- Identity Engine vs Classic (Settings, Account, or the admin console look and feel).
- Super admin access, and whether Security > Authenticators, Authentication Policies, Global Session Policy, and Custom Admin Roles are all present.
- Whether the OIN catalog allows SAML apps with provisioning (Applications > Browse App Catalog).
- Whether Directory Integrations allows an AD agent.
- Sandbox expiry date.

## 3. Enterprise architecture (fictional "Zafran Logistics")

| Role | System | Notes |
| --- | --- | --- |
| Authoritative employee source | HR simulator (CSV/JSON + small REST API) | Owns employeeId, legal name, dept, title, manager, status, start/end date |
| Authenticating directory | Active Directory (extension) | Owns sAMAccountName, UPN, password for on-prem |
| Identity provider | Okta | Central workforce IdP, policy enforcement |
| Downstream / SP | Entra ID (federation), AWS (SAML federation), OIN apps | Hold app-local accounts |
| Provisioning | Okta Lifecycle Management for OIN apps; custom script for HR to Okta | The script is lab automation, not a production integration |
| Authorization enforcement | Okta app/auth policies, AWS IAM roles, app-side roles | Okta decides who authenticates; apps decide what they can do |

### Attribute ownership

Conflict rule: the master wins, and the target gets overwritten.

- Employee ID, department, title, manager, status, dates: HR.
- Username/UPN: derived once from HR, then immutable.
- Password and AD group membership: AD (if AD is mastered) or Okta.
- Okta group membership: group rules driven by HR attributes.

Build order: Okta only, then OIN apps, then HR simulator, then AD, then Entra, then AWS. Each step is independently useful.

## 4. JML source of truth

**Recommended: a local HR simulator.** Use a CSV/SQLite store with a small Python REST API (Flask or FastAPI) and synthetic data. It's free, fully controllable, and can deliberately produce duplicate IDs, late terminations, and rehires. It is also honest that the integration is custom lab automation.

**Alternative: OrangeHRM Starter or another open-source HRIS.** It is more realistic as a product, but its current licensing, API coverage, and webhook support are unverified, so check those before committing. It also adds setup overhead with no exam benefit.

## 5. Phases and dependencies

| Phase | Content | Label | Depends on |
| --- | --- | --- | --- |
| A | Users, groups, group rules, custom attribute, admin roles, custom roles and resource sets | Exam-critical (UC1) | Lab 0 |
| D | Authenticators, enrollment policy, global session policy, authentication policy, password recovery | Exam-critical (UC4, 38%) | A |
| C-lite | SAML OIN app, group assignment, provisioning | Exam-critical (UC2, UC3) | A |
| F-lite | System Log, password expiry and session clearing, dashboard and group troubleshooting | Exam-critical (UC5, UC6) | A, D, C-lite |
| Timed drills | Free Standard and Premier practice exams, then your own 150-minute simulation | Exam-critical | All above |
| B | AD integration | Extension (license/agent dependent) | A |
| E | HR-driven JML, SCIM, reconciliation | Extension | A, C-lite |
| C-full, G | Entra and AWS federation, multi-system incidents, API operations | Extension | B, E |

Both practice exams are linked from the study guide. Treat the Premier practice exam as a measuring stick: attempt it early to find gaps, then again right before the real exam.

## 6. First-week plan

| Day | Focus |
| --- | --- |
| 1 | Lab 0 (capability checks) and read the UC1 docs |
| 2 | Lab 1 (users, attributes, groups) |
| 3 | Lab 2 (group rules, admin roles, custom role + resource set) |
| 4-5 | Lab 3 (authenticators and enrollment policy), the start of the 38% block |
| 6 | Timed UC1 drill (no steps given) |
| 7 | Baseline the free Standard Practice Exam to find real gaps |

## 7. Read before Lab 1 (from the official study guide)

- Manage users: https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-people.htm
- Edit user attributes: https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-edit-user-attributes.htm
- About groups: https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-about-groups.htm
- Manually assign people to a group: https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-assign-group-people.htm
- About user account status: https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-end-user-states.htm

After reading, you should be able to explain the Okta user lifecycle states, why a custom attribute is added in the Profile Editor rather than on a user, and when to assign groups manually versus by rule.

## 8. Lab 1: Foundation Users, Attributes and Groups

**Scenario:** Zafran's IT team needs the first employees in Okta with the right profile data and departmental groups.

**Objectives:** Create users with varied activation methods, add a custom attribute (employeeId, string), edit profiles, create departmental groups, and assign users manually.

**Acceptance criteria:**

1. Five synthetic users in at least two departments, with employeeId populated on each.
2. Two groups (for example GRP-Finance and GRP-Logistics) with correct membership.
3. You can state each user's status and explain why.
4. You can find the matching create and update events in System Log.
5. No walkthrough is provided: you get the business requirement and build it yourself.

## 9. Next step

Complete Lab 0 and bring back what you find: your org's Identity Engine status, which admin console sections are present, and your sandbox expiry date. The plan will be adjusted to match, then Lab 1's requirements are released. Diagnosis is based only on what you show, never on assumptions about your org.

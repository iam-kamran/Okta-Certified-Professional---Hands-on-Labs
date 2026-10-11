# Okta Certified Professional Performance Exam --- Enterprise Lab Curriculum

**Purpose:** Prepare for the Okta Certified Professional Performance
Exam while developing practical enterprise IAM engineering skills in a
safe sandbox.

**Method:** Documentation first → implementation → validation and
evidence → break/fix challenge → engineering review. Deliver one lab at
a time. Do not claim a tenant was inspected unless the learner provides
screenshots, configuration details, or logs.

> **Scope note:** This curriculum combines exam preparation with
> enterprise extensions. Feature availability depends on org type,
> Identity Engine capabilities, licences, permissions, and sandbox
> restrictions. Verify these before implementation. The current official
> study guide is authoritative; do not invent exam objectives, weights,
> task counts, or formats.

## 1. Readiness assessment

The learner has prior IAM knowledge and has completed foundational Okta
exercises involving Users, Groups, Universal Directory, administrative
roles, and least privilege. Available resources include the What's Next
by Okta program and sandbox, Microsoft Active Directory, Entra ID
capabilities subject to licences, AWS subject to permissions, and a
local computer for scripts and test services.

The latest **Implement Security Policies** assessment result was
**53.33%**, below the stated 70% passing threshold.

  ------------------------------------------------------------------------
  Section                                      Score Priority
  --------------------- ---------------------------- ---------------------
  Set Up Authentication                           0% Critical --- first
  Policies                                           priority

  Set Up Global Session                          75% Maintain with a short
  Policies                                           review

  Set Up MFA Enrollment                          60% High

  Set Up Password                                67% High
  Policies with                                      
  Self-Service Recovery                              
  Capability                                         
  ------------------------------------------------------------------------

Scores show study priorities, not the exact questions missed.

### Readiness strategy

1.  Prioritize Authentication Policies: rule order, conditions, app
    assignments, authentication requirements, and reauthentication.
2.  Review password policies and self-service recovery.
3.  Practise the distinction between authenticator enrollment and
    authentication requirements.
4.  Maintain Global Session Policy knowledge.
5.  Practise timed tasks: configure from a business requirement,
    validate, collect evidence, and explain the outcome.

**Official exam source:**
https://certification.okta.com/page/professional-performance-exam-study-guide

Okta announced a change effective July 6, 2026, making the Certified
Professional Performance Exam the standard Professional exam. Confirm
the current format and objectives in the official study guide before
scheduling. This document does not assert unverified exam weights or
task counts.

## 2. Proposed enterprise architecture

Use a fictional company, **Northstar Services**, throughout the labs.
Keep users, groups, applications, incidents, and changes consistent.

``` mermaid
flowchart TD
    HR["HR source of truth<br/>Synthetic employee records"]
    AD["Microsoft Active Directory<br/>Directory and optional authentication"]
    OKTA["Okta Identity Engine<br/>Profiles, groups, policies, lifecycle"]
    SF["Salesforce<br/>SAML / application access"]
    AWS["AWS IAM Identity Center<br/>Federated access and permissions"]
    ENTRA["Microsoft Entra ID<br/>Federation / hybrid identity scenarios"]
    OCI["Oracle Cloud Infrastructure<br/>Optional federation extension"]
    LOG["Okta System Log<br/>Audit and troubleshooting"]
    HR -. "Custom lab automation" .-> OKTA
    HR -. "Optional lifecycle automation" .-> AD
    AD <-->|"Agent / import / profile sourcing"| OKTA
    OKTA --> SF
    OKTA --> AWS
    ENTRA <-->|"Selected federation scenarios"| OKTA
    OKTA -. "Optional extension" .-> OCI
    OKTA --> LOG
    AD --> LOG
```

### Responsibilities and attribute ownership

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  HR source                           Authoritative employee ID,
                                      department, manager, job title,
                                      worker status, and employment dates

  Active Directory                    Existing directory and, where
                                      supported, an authentication source

  Okta                                Workforce identity profiles,
                                      groups, sign-on policies, app
                                      assignments, and identity
                                      orchestration

  Salesforce and AWS                  Downstream access and authorization
                                      enforcement

  Entra ID and OCI                    Optional federation and hybrid
                                      identity targets

  Automation service                  Translates HR events into supported
                                      API/directory operations and
                                      records actions

  Okta System Log                     Evidence for authentication,
                                      administrative changes, and
                                      supported lifecycle events
  -----------------------------------------------------------------------

**Ownership rule:** Define a source of truth for each attribute. Avoid
silent overwrites between systems. For example, HR may own `department`,
while selected directory attributes are sourced from AD. Deliberately
test conflicting updates.

Build incrementally. Document the existing Okta-to-AWS integration
before modifying it. Add Salesforce, Entra ID, and OCI only where
permissions and licensing allow.

## 3. Recommended JML source of truth

### Primary recommendation: local synthetic HR simulator

Start with CSV/JSON records and a lightweight local REST API. Add a
small database only if useful.

  ------------------------------------------------------------------------------
  Option         Cost             Setup          API and        Recommendation
                 considerations                  automation fit 
  -------------- ---------------- -------------- -------------- ----------------
  Local          No software      Low--medium    Fully          **Recommended
  CSV/JSON +     licence cost;                   controllable   starting point**
  REST API       local runtime                   lifecycle      
                                                 events,        
                                                 retries, and   
                                                 fault          
                                                 injection      

  OrangeHRM      Software may be  Medium         HR-style       Optional later
  Starter,       free; hosting                   records;       
  self-hosted    and maintenance                 verify         
  open-source    still cost                      supported API  
  edition        time/resources                  operations in  
                                                 the installed  
                                                 version        

  OrangeHRM      Confirm current  Low            Hosted service Convenient
  hosted Starter terms, trial                    may impose     alternative
                 conversion, and                 constraints    
                 API access                                     
  ------------------------------------------------------------------------------

OrangeHRM has advertised an open-source Starter edition and provides API
documentation. API documentation alone does not prove every required
lifecycle operation is available in a particular installation; verify
before relying on it.

### Synthetic employee attributes

-   `employeeId` --- immutable unique identifier
-   `firstName`, `lastName`
-   `email`, `username`
-   `department`, `jobTitle`, `managerId`, `location`
-   `workerType`, `employmentStatus`
-   `startDate`, `terminationDate`

Use synthetic data only.

### Required lifecycle events

1.  **Joiner:** Create or activate identity and assign appropriate
    access.
2.  **Mover:** Update authoritative attributes, recalculate access, and
    remove obsolete entitlements.
3.  **Leaver:** Disable access, revoke relevant sessions or credentials
    where supported, deprovision downstream accounts, and verify
    completion.
4.  **Rehire:** Reconcile the returning employee with the correct
    identity without duplicates.
5.  **Exception:** Handle missing data, duplicate IDs, failed APIs,
    delayed events, and partial provisioning.

Include reconciliation, retries, idempotency, error reporting, least
privilege, and an auditable record of each event. A locally triggered
script is custom lab automation, not a native HR integration.

## 4. Lab phases and dependencies

**Exam-critical** means directly relevant to the operational skill set.
**Enterprise extension** means additional engineering depth beyond the
minimum exam scope. Verify feature availability in the sandbox.

  --------------------------------------------------------------------------
  Phase                   Focus                      Priority
  ----------------------- -------------------------- -----------------------
  A --- Foundation        Org type, users, groups,   Exam-critical
  validation              profile attributes, admin  
                          roles, recovery, safe      
                          changes                    

  B --- Security policies Global Session Policy,     **Exam-critical; start
                          Authentication Policies,   here**
                          authenticator enrollment,  
                          password/recovery rules,   
                          precedence, lockout        
                          prevention                 

  C --- Application       Assignment, SAML/OIDC      Exam-critical +
  access                  where supported, claims,   extension
                          sign-in failures,          
                          group-based access         

  D --- Directory         AD agent prerequisites,    Exam-relevant where
  integration             imports, matching, profile supported + extension
                          sourcing, group sync,      
                          stale data                 

  E --- JML and           Joiner/mover/leaver,       Exam-relevant lifecycle
  provisioning            group-based access,        tasks + extension
                          deactivation, SCIM/API     
                          provisioning, rehire,      
                          reconciliation             

  F --- Operations and    System Log,                Exam-critical
  diagnostics             sign-in/provisioning/API   troubleshooting +
                          errors, evidence, rollback extension

  G --- Integrated        Excess access after        Enterprise extension
  capstones               transfer, termination      
                          failure, policy lockout,   
                          broken SAML, partial       
                          provisioning               
  --------------------------------------------------------------------------

``` mermaid
flowchart TD
    A["A. Validate org and baseline"]
    B["B. Security policies"]
    C["C. Application access"]
    D["D. Directory integration"]
    E["E. JML and provisioning"]
    F["F. Logs and troubleshooting"]
    G["G. Enterprise capstones"]
    A --> B
    A --> C
    A --> D
    B --> C
    C --> E
    D --> E
    B --> F
    C --> F
    E --> F
    F --> G
```

### Phase A --- Org foundation and administration

-   Explore the org, users, groups, profile attributes, and
    administrative roles.
-   Establish synthetic test identities and naming conventions.
-   Apply least-privilege administrative access.
-   Capture baseline configuration and establish recovery practices.

### Phase B --- Directory integration

-   Verify AD integration and agent prerequisites.
-   Practise imports, matching, profile sourcing, attribute mappings,
    and group imports where supported.
-   Diagnose duplicate identities, incorrect matching, stale attributes,
    import failures, connectivity, agent health, and synchronization.

### Phase C --- Authentication and application integration

-   Configure SAML/OIDC applications where available.
-   Understand IdP-initiated and SP-initiated flows.
-   Validate claims, assertions, attributes, redirect URIs,
    certificates, and metadata.
-   Test group-based assignment and access.
-   Add Entra ID and AWS scenarios where supported.

### Phase D --- Authentication security

-   Configure authenticator enrollment policies.
-   Configure Global Session and app Authentication Policies.
-   Test MFA enforcement, enrollment, session behaviour, and precedence.
-   Use network zones or contextual restrictions only where available.
-   Practise conflicting rules, unintended access, and legitimate-user
    lockout recovery.

### Phase E --- Provisioning and lifecycle

-   Define authoritative employee attributes and immutable identifiers.
-   Implement joiner, mover, and leaver flows.
-   Assign groups and applications based on department, role, location,
    and employment status.
-   Explore profile mastering and write-back where supported.
-   Test SCIM/API provisioning, reconciliation, deactivation, account
    cleanup, rehire, duplicate identity, delayed termination, and failed
    provisioning.

### Phase F --- Enterprise operations and troubleshooting

-   Investigate System Log events and correlate them with test actions.
-   Diagnose failed login, provisioning, federation, and
    synchronization.
-   Use API credentials and administrative API operations where
    relevant.
-   Handle bulk operations, rate limits, errors, retries, and
    idempotency.
-   Validate changes, preserve audit evidence, and document rollback and
    incident outcomes.

### Phase G --- Integrated enterprise exercises

-   An employee joins a department and receives correct groups, app
    assignments, and downstream access.
-   A department transfer leaves stale group membership and excessive
    privileges.
-   HR termination succeeds but synchronization failure leaves access
    active.
-   A user authenticates but cannot access an assigned application.
-   App login breaks after a certificate, claim, attribute, or policy
    change.
-   Provisioning failure creates inconsistent identities across systems.
-   An administrator change unintentionally affects a broad population.

## 5. Required structure for every lab

### A. Lab overview

Lab ID, title, difficulty, duration, prerequisites, exam-objective
mapping, business impact, objectives, and success criteria.

### B. Architecture and preparation

Systems, identity/data flows, required
accounts/groups/attributes/integrations/permissions, sandbox
limitations, cost implications, and rollback precautions.

### C. Documentation-first learning

Verified official documentation URLs; why each document matters;
sections to focus on; configuration decisions informed by the
documentation; concepts the learner should explain afterward.

### D. Implementation

Verified console navigation and configuration sequence; fields and
example values; expected results; rationale for important settings. Add
PowerShell, CLI, API requests, JSON, or scripts where appropriate,
including prerequisites and safe secret handling.

### E. Validation

Every test matrix must include test case, preconditions, action,
expected result, evidence, and pass/fail criteria. Include positive and
negative tests.

### F. Break/fix challenge

Initially give only symptoms, business impact, and available evidence.
Do not reveal the solution before the learner investigates using System
Log, configuration, API responses, directory data, and documentation.
After the learner submits a diagnosis, evaluate it and provide root
cause, corrective action, and validation.

### G. Enterprise engineering review

Explain root cause, contributing factors, security/business impact,
detection, prevention, monitoring, alerting, audit, recovery, change
management, and related failure modes.

### H. Deliverables

Where relevant, produce a configuration checklist, architecture update,
test evidence template, runbook, troubleshooting decision tree, and
concise knowledge notes.

## 6. Realistic incident design

Use symptoms such as: - Users unexpectedly lose access. - Terminated
employees retain access. - Identities are duplicated or incorrectly
linked. - Group rules grant excessive privileges. - Directory imports
fail or produce stale data. - SAML/OIDC authentication fails. -
Provisioning errors create inconsistent downstream accounts. -
MFA/session policy changes break access. - Application access differs
between user populations.

For each incident, define intended outcome, symptoms, available
evidence, diagnostic boundaries, and success criteria. Require the
learner to state a hypothesis, gather evidence, identify root cause,
implement a safe fix, and verify the fix without creating another
problem. Multi-cause incidents are useful when complexity supports the
learning objective.

## 7. Teaching and assessment method

-   Deliver one lab at a time; do not dump all implementation manuals at
    once.
-   Start with documentation and architecture context.
-   Guide the build while asking the learner to explain key decisions.
-   Introduce break/fix only after the initial build is validated.
-   Wait for screenshots, results, configuration descriptions, or logs
    before diagnosing the actual tenant.
-   Never claim to have accessed or verified the tenant without
    evidence.
-   Distinguish confirmed facts from hypotheses and intended behaviour
    from what evidence demonstrates.
-   Track completed labs, mistakes, knowledge gaps, and outstanding
    tasks.
-   Increase difficulty as performance improves.
-   For timed tasks, give requirements without click-by-click
    instructions before the attempt.
-   Assess technical accuracy, troubleshooting method, security
    judgement, and ability to explain results.

## 8. Sandbox capability checks and prerequisites

Complete these before changing configuration:

-   [ ] Confirm Identity Engine versus Classic Engine using org
    information/Admin Console.
-   [ ] Locate Global Session Policy, Authentication Policies, and
    Authenticator Enrollment policies.
-   [ ] Identify synthetic users, test groups, and a separate admin
    account.
-   [ ] Verify Salesforce and AWS IAM Identity Center assignments
    without modifying them.
-   [ ] Confirm AD domain reachability, agent-install permission, and a
    suitable host.
-   [ ] Confirm whether a scoped API credential or another supported
    administrative API method is available.
-   [ ] Confirm Entra ID/OCI access, licences, and permission to change
    configuration.
-   [ ] Capture the current policies and integration settings.

### Safety rules

-   Use synthetic identities and sandbox apps only.
-   Document or export existing configuration before changes.
-   Test policy changes with a designated user and retain a working
    admin session.
-   Avoid broad policy edits, tenant-wide deactivation, production
    credentials, and unnecessary cloud resources.
-   Record the original setting, intended change, validation result, and
    rollback procedure.

## 9. Prioritized first-week plan

Assume 30--60 minutes daily, with longer sessions if an integration
requires setup.

  ------------------------------------------------------------------------
  Day                     Focus                   Deliverable
  ----------------------- ----------------------- ------------------------
  1                       Sandbox checks and      Org capability inventory
                          baseline capture        and current policy map

  2                       Authentication Policies App-specific rule,
                                                  precedence review, and
                                                  test matrix

  3                       Global Session and MFA  Session/authentication
                          enrollment              comparison and
                                                  enrollment tests

  4                       Password policy and     Successful recovery test
                          recovery                and negative test

  5                       Application access      Validate one Salesforce
                                                  or AWS access path and
                                                  document evidence

  6                       Troubleshooting         Diagnose one
                                                  deliberately introduced
                                                  policy/access fault

  7                       Timed performance       Configure from
                          simulation              requirements, validate,
                                                  and explain
  ------------------------------------------------------------------------

## 10. Official documentation before Lab 1

These are starting references. Confirm that each URL resolves and
matches the tenant's engine/version before relying on a specific setting
name.

1.  **Professional Performance Exam Study Guide**\
    https://certification.okta.com/page/professional-performance-exam-study-guide\
    Authority for current exam objectives and format.

2.  **Okta policies overview**\
    https://help.okta.com/oie/en-us/content/topics/identity-engine/policies/about-policies.htm\
    Understand policy types and how rules affect access.

3.  **Create an app sign-in policy**\
    https://help.okta.com/oie/en-us/content/Topics/identity-engine/policies/create-auth-policy.htm\
    Study rule creation, conditions, app assignment, and priority.

4.  **Global Session Policy**\
    https://help.okta.com/oie/en-us/content/topics/identity-engine/policies/about-okta-sign-on-policies.htm\
    Study session establishment, duration, MFA prompts, and priority.

5.  **Authenticator enrollment policies**\
    https://help.okta.com/oie/en-us/content/topics/identity-engine/policies/about-mfa-enrollment-policies.htm\
    Study required/optional/disabled enrollment and rule targeting.

6.  **Self-service account recovery**\
    https://help.okta.com/oie/en-us/Content/Topics/identity-engine/authenticators/configure-sspr.htm\
    Study recovery authenticators and interaction with password/account
    management policies.

If a URL has moved or is inaccessible, search official Okta Help for the
exact title. Do not silently substitute an unverified third-party guide.

## 11. Lab 1 --- Authentication Policy Engineering

**Lab ID:** `LAB-SEC-01`\
**Title:** Design, configure, and validate app-specific authentication
requirements\
**Difficulty:** Intermediate\
**Duration:** 45--60 minutes\
**Environment:** Sandbox only\
**Priority:** Highest, based on the assessment breakdown

### Business scenario

Northstar Services wants stronger authentication for Salesforce without
unnecessarily changing the sign-in experience for every other
application.

### Objectives

-   Identify the current Global Session Policy and Salesforce app
    sign-in policy.
-   Determine which rule applies to the test user and why.
-   Configure an app-specific authentication requirement if the sandbox
    supports the needed controls.
-   Verify the outcome through positive and negative tests.
-   Explain why enrolling an authenticator does not automatically
    require it at sign-in.

### Acceptance criteria

1.  Identify the applicable policy and rule for the test user.
2.  Enforce the Salesforce requirement as intended.
3.  Preserve baseline behaviour for an unrelated application.
4.  Explain rule order and catch-all behaviour.
5.  Collect evidence from successful and unsuccessful tests.
6.  Document rollback to the baseline.

### Lab sequence

1.  Confirm the org engine and feature availability.
2.  Capture current policy configuration and rule order.
3.  Identify the test user, target app, and baseline behaviour.
4.  Read the official policy references.
5.  Attempt configuration from the business requirement.
6.  Test expected and negative cases.
7.  Record evidence and explain the outcome.
8.  After successful validation, proceed to a break/fix challenge.

**Performance standard:** Configure from the business requirement,
validate, and explain the result without relying on a click-by-click
guide.

## 12. First action

Complete the sandbox capability checklist and record whether the Admin
Console uses **Identity Engine or Classic Engine**. Also confirm whether
Salesforce is available in the sandbox. This determines the correct
terminology and safe implementation path for Lab 1. Do not change tenant
configuration until the current state and test scope are understood.

## Source and assumption notes

-   Readiness priorities are based on the learner's assessment scores.
-   Architecture, lab phases, safety rules, and acceptance criteria are
    proposed training design.
-   Feature availability, licence entitlement, current UI labels, and
    integration capabilities must be verified in the actual sandbox.
-   The current official exam study guide takes precedence if exam scope
    or format changes.

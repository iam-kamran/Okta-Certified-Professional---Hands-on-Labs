# Okta Enterprise IAM Lab Architect: Curriculum & Lab 1

## 1. Enterprise Architecture: Acme Cloud Services

To simulate a real-world environment, we will build a Hub-and-Spoke identity architecture where Okta serves as the central identity plane.

```text
[ Authoritative Source ]      [ Identity Control Plane ]       [ Downstream Services ]
       (HR IS)                           (Okta)                       (SP / IaaS)
          |                                |                               |
    Python REST API  ============>   Okta Universal  ============> Microsoft Entra ID
  (Mock JSON Data)    (POST/PUT)    Directory (UD)     (SCIM)    (M365/Azure access)
          |                                |                               |
          |                                |                             AWS IAM
          |                                |=======(SAML 2.0)====> (Identity Center)
          |                                |
          v                                v
  [ Legacy Directory ]             (OIE Policies)
  Active Directory   <===========>  Auth / MFA / Session
 (On-Prem Simulation)   (Agent)      Evaluation
```

*   **Source of Truth:** HR data dictates Joiner, Mover, and Leaver events.
*   **Active Directory (AD):** Treated as a legacy directory. It will feed legacy attributes (like `sAMAccountName`) to Okta, but Okta will win conflicts on HR-driven fields (Title, Department).
*   **Okta:** Handles aggregation, attribute-level mastering, policy enforcement, and routing.

## 2. JML Integration Strategy (Zero Cost)

Instead of a heavy third-party HR system, you will act as the "HR Integration Middleware."
*   **Data Source:** You will generate a `users.json` file using Mockaroo.
*   **Mechanism:** You will write a Python script using the `requests` library to interact with the Okta `/api/v1/users` endpoint.
*   **Learning Value:** This forces you to understand Okta's JSON payload structure, rate limiting (HTTP 429), and lifecycle states (Staged vs. Active), which are critical for the Professional exam and real-world troubleshooting.

## 3. Curriculum Phases (Expanded)

*   **Phase A: Foundation (Lab 1)** - Schema extensions, custom RBAC admin roles, break-glass accounts.
*   **Phase B: Directory Integration** - Deploying the Okta AD agent, mapping AD attributes, handling Profile Mastering conflicts.
*   **Phase C: Applications & Federation** - Configuring SAML 2.0 for AWS, mappings claims, testing IdP vs. SP initiated flows.
*   **Phase D: OIE Security** - Global Session Policies vs. Authentication Policies, Contextual access (Network Zones), MFA enrollment rules.
*   **Phase E: Automation & JML** - Group Rules based on custom attributes, SCIM provisioning to Entra ID, lifecycle API scripting.
*   **Phase F: Operations & Troubleshooting** - Syslog deep dives, identifying token exhaustion, resolving overlapping group rule conflicts.

---

# LAB 1: Org Foundation and Administration

**Lab ID:** IAM-LAB-01 | **Difficulty:** Intermediate | **Est. Time:** 45 mins

**Business Scenario:** Acme Cloud Services needs a secure administrative baseline before integrating HR or AD. You must configure the Okta User Profile schema to support future HR data, establish a highly privileged "break-glass" account, and delegate a least-privilege Helpdesk role for managing Contractor passwords.

### A. Documentation-First Learning
Review these before starting:
1.  [Okta Custom Attributes](https://help.okta.com/oie/en-us/content/topics/users-groups-profiles/usgp-custom-attributes.htm) - Understand the difference between Base and Custom attributes.
2.  [Custom Admin Roles](https://help.okta.com/oie/en-us/content/topics/security/custom-admin-roles/custom-admin-roles.htm) - Grasp the concept that `Role + Resource Set = Assignment`.

### B. Implementation Steps

#### Task 1: Create the Break-Glass Administrator
*Why: A cloud-only account bypasses federation or downstream directory outages.*
1. Navigate to **Directory > People** and click **Add Person**.
2. **User type:** Okta profile.
3. **First Name:** `Break` | **Last Name:** `Glass`
4. **Username/Primary Email:** `breakglass@<your-domain>.com` (Use a valid email you can access).
5. **Password:** Set by admin (create a strong temporary password).
6. Save the user.
7. Navigate to **Security > Administrators > Admins** tab.
8. Click **Add Administrator**.
9. Select your `Break Glass` user. Assign the **Super Administrator** role.

#### Task 2: Extend the Universal Directory Schema
*Why: HR will send Cost Center and Employment Status data. Okta needs fields to store them.*
1. Navigate to **Directory > Profile Editor**.
2. Locate the **Okta User (default)** profile and click it.
3. Click **Add Attribute**.
4. Configure Attribute 1:
   *   **Data type:** `string`
   *   **Display name:** `Cost Center`
   *   **Variable name:** `costCenter`
   *   Leave other settings default and click **Save and Add Another**.
5. Configure Attribute 2:
   *   **Data type:** `string`
   *   **Display name:** `Employment Status`
   *   **Variable name:** `employmentStatus`
   *   **Enum:** Check "Define list of values"
   *   Add values: `FTE` (Display: Full Time), `Contractor` (Display: Contractor).
6. Click **Save**.

#### Task 3: Implement Least-Privilege Custom Admin Roles
*Why: Helpdesk should only be able to reset passwords for Contractors, not Executives or Super Admins.*
1. **Create the Target Group:** Navigate to **Directory > Groups** -> **Add Group**. Name it `Population - Contractors`.
2. **Create the Role:** Navigate to **Security > Administrators > Roles** tab.
   *   Click **Create new role**. Name it `Helpdesk - Password Reset Only`.
   *   Under permissions, expand **User Management** and check *only* **Reset passwords**. Save.
3. **Create the Resource Set:** Navigate to the **Resources** tab.
   *   Click **Create new resource set**. Name it `Contractor Population`.
   *   Click **Add Resource**, select **User group**, and type/select `Population - Contractors`. Save.
4. **Bind them together:** Navigate to the **Admins** tab.
   *   Click **Add Administrator**.
   *   Select a test user (e.g., create a user named `Helpdesk Admin`).
   *   Under Role, select `Helpdesk - Password Reset Only`.
   *   Under Resource set, select `Contractor Population`. Save.

### C. Validation Matrix

| Test Case | Action | Expected Result | Pass/Fail |
| :--- | :--- | :--- | :--- |
| Schema Update | Go to Directory > People > Add Person. | `Cost Center` and `Employment Status` (as a dropdown) are visible in the form. | |
| Least Privilege 1 | Log in as `Helpdesk Admin`. Try to reset the password of `Break Glass`. | Action should be denied / button unavailable. | |
| Least Privilege 2 | Add a test user to `Population - Contractors`. As `Helpdesk Admin`, try to reset their password. | Action should succeed. | |

### D. Break/Fix Challenge

**Symptom:** You log in as your `Helpdesk Admin` account. You navigate to the `Population - Contractors` group to reset a user's password, but the "Reset Password" button is grayed out or missing entirely for a user in that group.

**Your Task:** Review your configuration. Assuming the steps above were followed exactly, why might the Helpdesk Admin still be unable to reset a specific contractor's password? What Okta administrative hierarchy rule is preventing this?

*Investigate this in your sandbox and report your hypothesis before I reveal the answer.*
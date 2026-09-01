# Dunder Mifflin Enterprise Identity Suite

This repository documents the architecture and validation procedures for a centralized enterprise identity infrastructure built within a standalone Microsoft Entra ID tenant. The environment mirrors a distributed branch-network scenario for Dunder Mifflin Paper Company.

---

## 🛠️ Infrastructure Labs

<details>
<summary><b>LAB 1: Hybrid Directory Architecture & Core Synchronization Lifecycle</b></summary>

### 1. Scenario Context
* On-premises legacy domain metadata stores active profiles, organizational units, and branch policies.
* User Creed Bratton arbitrarily modifies system credentials out-of-band, fracturing directory alignment.
* Engineering Objective: Connect local identities securely to Microsoft Entra ID via automated sync engines.

### 2. Implementation Walkthrough

#### Phase 1: Local Server Provisioning
1. Deploy an isolated virtual machine running Windows Server 2022 Standard.
2. Initialize Active Directory Domain Services (AD DS) through Server Manager.
3. Establish a new forest root domain designated as `dunmifflin.local`.
4. Open Active Directory Domains and Trusts to append the `dunmifflin.org` UPN suffix.
5. Launch Active Directory Users and Computers (ADUC) to build `OU=Scranton-Branch`.
6. Provision target employee accounts inside the new container.
7. Map Creed Bratton's account to the routing identity `cbratton@dunmifflin.org`.

> **[INSERT SCREENSHOT 1.1: ADUC SHOWING THE SCRANTON-BRANCH OU AND PROVISIONED USER OBJECTS]**

#### Phase 2: Synchronization Topology Configuration
1. Initialize the Microsoft Entra Connect configuration assistant on the Domain Controller.
2. Select the Custom Installation wizard path to control configuration scaling rules.
3. Authenticate with the tenant utilizing dedicated Hybrid Identity Administrator credentials.
4. Input local administrative parameters to bind the `dunmifflin.local` forest profile.
5. Access the Domain and OU Filtering window to uncheck the entire directory tree.
6. Check only the explicit box tracking the `OU=Scranton-Branch` container layer.
7. Choose Password Hash Synchronization (PHS) as the underlying sign-in structure.
8. Toggle on the Password Writeback setting inside the Optional Features dashboard view.
9. Finish the engine configuration window to fire the baseline replication loop.

> **[INSERT SCREENSHOT 1.2: ENTRA CONNECT SETTINGS SHOWING DIRECTORY FILTRATION FOR THE SCRANTON BRANCH ONLY]**

### 3. Engineering Challenge & Troubleshooting Logs
* **The Failure:** The synchronization service manager flagged an `AttributeValueMustBeUnique` validation fault. Replication froze across the gateway directory stream due to a legacy cloud-only user record sharing Creed's mail attribute string.
* **The Fix:** Cleaned the stale object out of the cloud directory using the command-line interface.

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
Get-MgUser -Filter "UserPrincipalName eq 'cbratton@dunmifflin.org'" | Select-Object Id
Remove-MgUser -UserId "[Target-Object-GUID]"
```

* **The Result:** Re-triggered a manual initialization replication loop from the server room terminal screen.

```powershell
Start-ADSyncSyncCycle -PolicyType Initial
```

### 4. Verification Benchmarks
1. Access the Microsoft Entra cloud directory interface under Identity ➔ Users.
2. Filter the user listing table to display the synchronized branch accounts.
3. Confirm the **On-premises sync enabled** dashboard column flags a value of **Yes**.

> **[INSERT SCREENSHOT 1.3: CLOUD USER MATRIX VERIFYING SYNCHRONIZED ACCOUNTS ARE ACTIVE]**

</details>

---

<details open>
<summary><b>LAB 2: Zero-Trust Conditional Access & Sign-In Risk Policies</b></summary>

### 1. Scenario Context
* Sales rep Jim Halpert travels out-of-state to Philadelphia to execute high-value contracts.
* Security risks elevate due to management paranoia regarding threat actors stealing sales leads.
* Engineering Objective: Build an identity perimeter to intercept malicious connections from unverified networks.

### 2. Implementation Walkthrough

#### Phase 1: Policy Scope Setup
1. Launch the Entra portal panel using Security Administrator permissions.
2. Open the left-side navigation blade and head to Protection ➔ Conditional Access.
3. Click the Create New Policy link button to open the rule authoring workspace.
4. Set the policy system name parameters to `DM-CA02: Sign-In Risk Enforcement Perimeter`.
5. Select Users ➔ check Select users and groups ➔ search and assign Jim Halpert's identity object.
6. Open Target resources ➔ choose Cloud apps ➔ flip the application dropdown to All cloud apps.

> **[INSERT SCREENSHOT 2.1: CONDITIONAL ACCESS INTERFACE FOCUSING TARGETING SCAPE ON JIM HALPERT]**

#### Phase 2: Threat Detection & Access Controls
1. Click the Conditions sidebar window and select the Sign-in risk selection block.
2. Flip the configuration option toggle switch state over from No to Yes.
3. Check the specific risk level assessment checkboxes tracking High and Medium threat profiles.
4. Scroll down the panel structure view to open the Access controls ➔ Grant menu block.
5. Click Grant access to reveal enforcement rules and check **Require multi-factor authentication**.
6. Switch the Enable policy toggle control location at the footer from Report-only to **On**.
7. Select Create to compile the policy rules across global network data centers.

> **[INSERT SCREENSHOT 2.2: CONDITIONS INTERFACE MAP CONFIGURING HIGH AND MEDIUM SIGN-IN RISK ASSIGNMENTS]**

### 3. Engineering Challenge & Troubleshooting Logs
* **The Failure:** Ran an exploitation simulation via a Tor browser node using Jim's login parameters. The connection bypassed security checks because the policy rule targeted *User Risk* (compromised credentials) instead of *Sign-In Risk* (connection mechanics anomalies).
* **The Fix:** Opened the policy configuration console workspace. Disabled the User Risk tracking rule parameters and toggled on the Sign-in risk conditions configuration switch instead.

### 4. Verification Benchmarks

#### Active Exploitation Run
1. Launch an isolated anonymous Tor window instance to act as the adversarial threat vector.
2. Navigate to `://office.com` and type in Jim Halpert's corporate email credentials.
3. Observe the cloud identity suite immediately evaluate the anonymous routing layer traffic.
4. Verify the player screen freezes authentication and pops a mandatory MFA roadblock challenge.

> **[INSERT SCREENSHOT 2.3: ATTEMPTS VIA PROXY BLOCKED BY CONDITIONAL ACCESS CHALLENGES]**

#### Security Audit Trail
1. Return to the primary admin console browser window viewing the monitoring directory tree.
2. Navigate to Identity ➔ Monitoring & health ➔ Sign-in logs.
3. Drill down into the specific failed authentication session row logging Jim's account ID.
4. View the Conditional Access detail blade panel view to verify the policy row details.
5. Confirm the system applied a green **Success** evaluation status rule confirmation stamp.

> **[INSERT SCREENSHOT 2.4: SYSTEM SIGN-IN LOG ANALYSIS TRACKING THE INTERCEPTED THREAT LOG]**

</details>

---

<details>
<summary><b>LAB 3: Privileged Identity Governance & Just-In-Time Lifecycle</b></summary>

### 1. Scenario Context
* Executive absence leaves a branch network leadership space wide open for exploitation.
* User Dwight Schrute demands permanent root Global Administrator visibility to run the network layer.
* High permissions introduce privilege vulnerabilities and insider-threat risk vectors.
* Engineering Objective: Configure a lifecycle perimeter providing temporary, auditable privilege elevations.

### 2. Implementation Walkthrough

#### Phase 1: Governance Structure Setup
1. Log into the administrative console and track to Privileged Identity Management (PIM).
2. Expand the Manage menu layout items to open Azure AD roles ➔ Roles.
3. Locate Global Administrator to review the existing system assignment list view.
4. Click Add assignments to link a new identity token tracking policy.
5. Select members ➔ search for Dwight Schrute's account ➔ link the target profile object.
6. Swap Assignment type configurations from static Active parameters down to **Eligible**.

> **[INSERT SCREENSHOT 3.1: PIM PERMISSIONS DIRECTORY TREE SHOWING DWIGHT ASSIGNED TO THE ELIGIBLE MATRIX]**

#### Phase 2: Role Lifecycle Hardening
1. Click the Global Administrator settings gear icon link inside the PIM control space.
2. Select Edit to modify baseline role assignment activation parameters.
3. Pull the Maximum activation duration slider down to lock role elevation at **2 hours**.
4. Check the activation criteria requirement box forcing Multi-Factor Authentication.
5. Check the rule tracking option for **Require justification on activation**.
6. Hit the Save button choice at the bottom edge to write changes across the infrastructure.

> **[INSERT SCREENSHOT 3.2: PIM CONTROL PANE SHOWING HOURLY LIFETIME RESTRICTIONS AND MANDATORY FIELDS]**

### 3. Engineering Challenge & Troubleshooting Logs
* **The Failure:** Executed a role escalation test simulation using Dwight's user account. The platform verified the justification entry text but failed to trigger the automated timeout cleanup script, leaving root privileges active permanently.
* **The Fix:** Conducted an administrative trace and discovered Dwight's profile was nested inside a legacy security group that inherited direct, permanent Global Admin permissions outside PIM visibility rules. Purged the account from that legacy security group container to clear the inheritance leak.

### 4. Verification Benchmarks
1. Access the Privileged Identity Management console ➔ Azure AD roles ➔ Audit history.
2. Review the timeline logs to track the chronological lifecycle events.
3. Confirm the log records the entry text business case justification statement.

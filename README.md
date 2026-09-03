# $${\color{green}\text{📄 Dunder Mifflin Cloud Identity Security Labs}}$$

Hello and Welcome to Dunder Mifflin! These projects document the architecture and validation procedures for a centralized identity environment built within a standalone Microsoft Entra ID tenant. This environment mirrors corporate branch-network fictional scenarios from the TV Show, "The Office" across **three distinct infrastructure labs**. 

To maintain strict structural continuity, each lab is systematically broken down into a standard four-part engineering lifecycle:
1. **Scenario Context:** The business vulnerabilities and operational storylines driving the project.
2. **Implementation Walkthrough:** The granular, step-by-step technical deployment procedures executed in the portal.
3. **Engineering Challenge & Troubleshooting Logs:** Real-world roadblocks encountered during staging and the custom engineering fixes used to overcome them.
4. **Verification Benchmarks:** Definitive log tracking data and simulation testing to verify infrastructure compliance.


---

## 🛠️ Infrastructure Labs  $${\color{Green}\text{* CLICK BELOW *}}$$


<details>
<summary><b>⚙️LAB 1: Hybrid Directory Architecture & Core Synchronization Lifecycle</b></summary>

### 1. Scenario Context
* On-premises legacy domain stores active profiles, organizational units, and branch policies.
* User Creed Bratton arbitrarily modifies system credentials out-of-band, fracturing directory alignment.
* Engineering Objective: Connect local identities securely to Microsoft Entra ID via automated sync engines.

### 2. Implementation Walkthrough

#### Phase 1: Local Server Provisioning
1. Deploy an isolated virtual machine running Windows Server 2022 Standard.
2. Initialize Active Directory Domain Services (AD DS) through Server Manager.
3. Establish a new forest root domain designated as `dunmifflin.local`.
4. Open Active Directory Domains and Trusts to append the `dunmifflin.org` UPN suffix.
5. Launch Active Directory Users and Computers (ADUC) to build `OU=Dunder Mifflin`.
6. Provision target employee accounts inside the new container.
7. Map Creed Bratton's account to the routing identity `cbratton@dunmifflin.org`.

> <img width="1920" height="984" alt="image" src="https://github.com/user-attachments/assets/7e9893bb-9b20-4dde-8a83-a82e129ee22d" />

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

> <img width="1313" height="924" alt="image" src="https://github.com/user-attachments/assets/c69498f2-4428-4ea1-bd37-d8cfc1aefd8b" />

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

> <img width="611" height="401" alt="image" src="https://github.com/user-attachments/assets/c59a1089-c6b1-4809-ad99-2fcc2b89af66" />

</details>

---

<details>
<summary><b>🛡️LAB 2: Zero-Trust Conditional Access & Sign-In Risk Policies</b></summary>

### 1. Scenario Context
* Sales rep Jim Halpert travels out-of-state to Philadelphia to close a major paper deal.
* Security risks elevate due to Dwight's paranoia regarding threat actors stealing sales leads.
* Engineering Objective: Build an identity perimeter to intercept malicious connections from unverified networks.

### 2. Implementation Walkthrough

#### Phase 1: Policy Scope Setup
1. Launch the Entra portal panel using Security Administrator permissions.
2. Open the left-side navigation blade and head to Protection ➔ Conditional Access.
3. Click the Create New Policy link button to open the rule creation panel.
4. Set the policy system name parameters to `Risky Sign-ins MFA`.
5. Select Users ➔ check Select users and groups ➔ search and assign Jim Halpert's identity object.
6. Open Target resources ➔ choose Cloud apps ➔ flip the application dropdown to All cloud apps.

> <img width="362" height="434" alt="image" src="https://github.com/user-attachments/assets/2a39f970-8947-4f60-b478-0b5c16419bc8" />

#### Phase 2: Threat Detection & Access Controls
1. Click the Conditions sidebar window and select the Sign-in risk selection block.
2. Flip the configuration option toggle switch state over from No to Yes.
3. Check the specific risk level assessment checkboxes tracking High and Medium threat profiles.
4. Scroll down the panel structure view to open the Access controls ➔ Grant menu.
5. Click Grant access to reveal enforcement rules and check **Require multi-factor authentication**.
6. Switch the Enable policy toggle control location at the footer from Report-only to **On**.
7. Select "Create" to compile the policy rules across the global network data centers.

> <img width="950" height="423" alt="image" src="https://github.com/user-attachments/assets/664b11be-82e1-4115-b4a5-4a295fe7caf6" />

### 3. Engineering Challenge & Troubleshooting Logs
* **The Failure:** Ran an exploitation simulation via a Tor browser node using Jim's login parameters. The connection bypassed security checks because the policy rule targeted *User Risk* (compromised credentials) instead of *Sign-In Risk* (connection mechanics anomalies).
* **The Fix:** Opened the policy configuration console workspace. Disabled the User Risk tracking rule parameters and toggled on the Sign-in risk conditions configuration switch instead.

### 4. Verification Benchmarks

#### Active Exploitation Run
1. Launch an isolated anonymous Tor window instance to act as the adversarial threat vector.
2. Navigate to `https://myapplications.microsoft.com` and type in Jim Halpert's email credentials.
3. Observe the cloud identity suite immediately evaluate the anonymous network mask.
4. Verify the screen freezes authentication and pops a mandatory MFA roadblock challenge.

> <img width="557" height="373" alt="image" src="https://github.com/user-attachments/assets/b04eff28-aed2-4ecd-b2fa-18f49f42ca4e" />

#### Security Audit Trail
1. Return to the primary admin console browser window viewing the monitoring directory tree.
2. Navigate to Identity ➔ Monitoring & health ➔ Sign-in logs.
3. Drill down into the specific failed authentication session row logging Jim's account ID.
4. View the Conditional Access detail blade panel view to verify the policy row details.
5. Confirm the system applied a green **Success** evaluation status rule confirmation stamp.

> <img width="518" height="340" alt="image" src="https://github.com/user-attachments/assets/61d4752a-adb7-49e3-bfbf-1fd39b8bc627" />

</details>

---

<details>
<summary><b>🔑LAB 3: Privileged Identity Governance & Just-In-Time Lifecycle</b></summary>

### 1. Scenario Context
* The absence of manager Michael Scott leaves a branch network leadership space wide open for exploitation.
* User Dwight Schrute demands permanent Global Administrator visibility to run the network layer.
* High permissions introduce privilege vulnerabilities and insider-threat risk vectors.
* Engineering Objective: Configure a lifecycle perimeter providing temporary, auditable privilege elevations.

### 2. Implementation Walkthrough

#### Phase 1: Governance Structure Setup
1. Log into the administrative console and scroll to Privileged Identity Management (PIM).
2. Expand the Manage menu layout items to open Azure AD roles ➔ Roles.
3. Locate Global Administrator to review the existing system assignment list view.
4. Click Add assignments to link a new identity token tracking policy.
5. Select members ➔ search for Dwight Schrute's account ➔ link the target profile object.
6. Swap Assignment type configurations from "Active" parameters down to **Eligible**.

> <img width="336" height="415" alt="image" src="https://github.com/user-attachments/assets/4a2a7c06-e330-4130-836b-6b90a5c0e0f7" />

#### Phase 2: Role Lifecycle Hardening
1. Click the Global Administrator settings gear icon link inside the PIM control space.
2. Select Edit to modify baseline role assignment activation parameters.
3. Pull the Maximum activation duration slider down to lock role elevation at **8 hours**.
4. Check the activation criteria requirement box forcing Multi-Factor Authentication.
5. Check the rule tracking option for **Require justification on activation**.
6. Hit the Save button choice at the bottom edge to write changes across the infrastructure.

> <img width="276" height="415" alt="image" src="https://github.com/user-attachments/assets/f845a057-2eea-4a0d-b1f4-0405c07fa941" />

### 3. Engineering Challenge & Troubleshooting Logs
* **The Failure:** Executed a role escalation test simulation using Dwight's user account. The platform verified the justification entry but failed to trigger the automated timecap script, leaving privileges active permanently.
* **The Fix:** Conducted an administrative trace and discovered Dwight's profile was nested inside a legacy security group that inherited direct, permanent Global Admin permissions outside PIM visibility rules. Purged the account from that legacy security group container to clear the inheritance leak.

### 4. Verification Benchmarks
1. Access the Privileged Identity Management console ➔ Azure AD roles ➔ Audit history.
2. Review the timeline logs to track the chronological lifecycle events.
3. Confirm the log records the entry and business case justification statement.

# Dunder Mifflin Enterprise Identity & Security Lab Suite

This repository documents the technical design, deployment configuration, and validation procedures for a centralized enterprise identity infrastructure built within a standalone Microsoft Entra ID tenant. The environment mirrors a distributed branch-network scenario for Dunder Mifflin Paper Company, addressing legacy synchronization vulnerabilities, zero-trust perimeter access controls, and automated privilege governance lifecycles.

---

## LAB 1: Hybrid Directory Architecture & Core Synchronization Lifecycle

### 1. Scenario & Business Objective
Management has initiated a cloud-first infrastructure modernization mandate to migrate distributed branch resources into the Microsoft Entra cloud. However, the existing directory control plane relies on a legacy, on-premises directory infrastructure that houses active user profiles, organizational units (OUs), and regional security policies. 

Compounding this, erratic end-user behavior—specifically exemplified by employee Creed Bratton arbitrarily modifying his local system passwords outside of standard IT visibility—creates a fractured identity boundary. The technical objective is to engineer a secure hybrid directory bridge that establishes automated password hash replication and identity provisioning from the on-premises environment to Microsoft Entra ID without exposing local domain assets to external entry vectors.

### 2. Detailed Implementation Walkthrough

#### Phase 1: On-Premises Active Directory Staging
* **Operating System Isolation:** Provisioned a virtual machine running Windows Server 2022 Standard Edition to act as the primary domain controller (DC) for the local branch.
* **Directory Initialization:** Configured Active Directory Domain Services (AD DS) via Server Manager. Promoted the server to a domain controller in a new forest, defining the internal root domain namespace as `dunmifflin.local`.
* **Domain Routing Alignment:** Navigated to Active Directory Domains and Trusts. Added the public, verified domain routing suffix `dunmifflin.org` as an available User Principal Name (UPN) suffix to ensure cloud-routing compatibility.
* **Organizational Structure Provisioning:** Opened Active Directory Users and Computers (ADUC). Structured a dedicated Organizational Unit labeled `OU=Scranton-Branch,DC=dunmifflin,DC=local`. Inside this container, provisioned the targeted employee objects, explicitly modifying Creed Bratton's account to utilize the `cbratton@dunmifflin.org` UPN mapping.

> **[INSERT SCREENSHOT 1.1: ACTIVE DIRECTORY USERS AND COMPUTERS SHOWING SCRANTON-BRANCH OU AND USER OBJECTS]**

#### Phase 2: Microsoft Entra Connect Architecture & Deployment
* **Sync Engine Initialization:** Downloaded and launched the Microsoft Entra Connect configuration wizard on the local domain controller. Selected the Custom Installation path to maintain granular control over object scoping.
* **Tenant Credential Mapping:** Authenticated to the cloud environment using dedicated Hybrid Identity Administrator credentials. Connected the local directory by inputting the `dunmifflin.local` enterprise administrator credentials.
* **Directory Scoping and Filtering:** Navigated to the Domain and OU Filtering screen. Modified the default scoping from synchronization of the entire domain to sync only the explicit `OU=Scranton-Branch` container. This prevents default built-in service accounts from polluting the cloud directory space.
* **Credential Replication Tuning:** Selected Password Hash Synchronization (PHS) as the primary sign-in method. Enabled Password Writeback to allow cloud-initiated password modifications to securely replicate back down to the local server room, resolving the local configuration vulnerability.

> **[INSERT SCREENSHOT 1.2: MICROSOFT ENTRA CONNECT CONFIGURATION SCREEN SHOWING OU FILTERING EXCLUSIVELY SCOPED TO THE SCRANTON-BRANCH]**

### 3. Engineering Challenge & Resolution Strategy
* **The Vulnerability/Error:** During the initial execution of the delta synchronization cycle, replication stalled entirely. The Microsoft Entra Connect Synchronization Service manager flagged a critical loop failure labeled `AttributeValueMustBeUnique` alongside an ongoing `DirSync` replication freeze. Tracing the event metadata revealed that a legacy cloud-only test account for Creed Bratton shared an identical mail attribute with the incoming on-premises user object, causing an immutable identity conflict at the cloud gateway.
* **The Resolution:** To remediate the sync failure without destroying directory integrity, the hardcoded cloud conflict had to be resolved. Connected to the tenant via the Microsoft Graph PowerShell SDK using the `User.ReadWrite.All` permission scope. Executed a command to locate the conflicting cloud-only object GUID and purged it completely using `Remove-MgUser -UserId [Object-ID]`. Back on the local domain controller, forced an immediate, full directory sync cycle via PowerShell:
  `Start-ADSyncSyncCycle -PolicyType Initial`
  The replication engine cleared instantly, cleanly mapping the local on-premises identity to a synchronized cloud object.

### 4. Technical Validation Logs
To confirm the integrity of the hybrid identity bridge, navigate to the Entra admin portal and inspect the operational health metrics:
* Expand Identity ➔ Users ➔ All Users.
* Verify the user table contains the synchronized object profiles.
* Audit the **On-premises sync enabled** attribute column; it must explicitly display a status value of **Yes** with a source indicator matching the Microsoft Entra Connect sync engine.

> **[INSERT SCREENSHOT 1.3: ENTRA PORTAL ALL USERS DASHBOARD SHOWING SYNCHRONIZED USER OBJECTS MARKED WITH ON-PREMISES SYNC ENABLED AS YES]**
---

## LAB 2: Zero-Trust Conditional Access & Sign-In Risk Policies

### 1. Scenario & Business Objective
Sales representative Jim Halpert is executing an out-of-state travel itinerary to Philadelphia to negotiate a high-revenue paper procurement contract with an international account. Concurrently, regional management—driven by Dwight Schrute's highly critical risk assessments—is deeply concerned that an external adversarial entity (specifically classified as the Scranton Strangler) will attempt an identity impersonation exploit to breach the corporate network and compromise proprietary sales leads. 

The engineering objective is to architect and push live a Zero-Trust Conditional Access policy that dynamically monitors identity authentication behavior and intercepts unauthorized, high-risk connection attempts from untrusted proxy architectures without degrading standard user workflows.

### 2. Detailed Implementation Walkthrough

#### Phase 1: Policy Scope Targeting
* **Perimeter Engine Initialization:** Logged into the Microsoft Entra admin center using Security Administrator privileges. Expanded the Protection menu and navigated to Conditional Access ➔ Policies. Initialized a new policy rule labeled `DM-CA02: Edge Access - Sign-In Risk Enforcement Perimeter`.
* **Identity Object Assignment:** Clicked on Users ➔ Select users and groups. Searched the integrated cloud directory specifically for Jim Halpert's identity account and added it to the mandatory enforcement group.
* **Resource Boundary Mapping:** Selected Target resources ➔ Cloud apps. Swapped the selector to All cloud apps. This configuration builds an explicit security canopy over the entire office application matrix, including corporate mailboxes, file shares, and customer lead databases.

> **[INSERT SCREENSHOT 2.1: CONDITIONAL ACCESS CONFIGURATION SHOWING USER ASSIGNMENTS SCOPED EXCLUSIVELY TO JIM HALPERT]**

#### Phase 2: Risk Engine & Access Control Gateways
* **Sign-In Risk Factor Activation:** Expanded the Conditions menu and selected Sign-in risk. Toggled the configuration switch to Yes. Under the risk level selection matrix, checked the checkboxes for High and Medium. This instructs the cloud's real-time machine learning engines to evaluate the behavioral metadata of every single authentication request. If the login attempts exhibit indicators of anonymous proxy routing, impossible travel velocities, or unmapped IP subnets, the session profile risk score is elevated instantly.
* **Enforcement Matrix Mapping:** Scrolled down to the Access controls workspace and selected Grant. Modified the logic parameters from passive tracking to active blocking. Checked the rule for Grant Access but appended a strict conditional dependency requirement: **Require multi-factor authentication**.
* **Global Tenant Activation:** Toggled the Enable policy switch at the base of the portal from Report-only straight to **On**. Clicked Create to push the policy configuration out to Microsoft's global edge infrastructure.

> **[INSERT SCREENSHOT 2.2: CONDITIONS CONFIGURATION WINDOW WITH SIGN-IN RISK LEVELS SET TO HIGH AND MEDIUM]**

### 3. Engineering Challenge & Resolution Strategy
* **The Vulnerability/Error:** During initial penetration testing of the perimeter, I launched an anonymous Tor browser session on a test device to mimic adversarial traffic routing. Upon attempting a login as Jim Halpert, the authentication session went straight through to the dashboard without triggering any security challenges. Reviewing the developer portal configuration revealed a massive structural flaw: the policy had been accidentally scoped under the *User Risk* condition rather than the *Sign-In Risk* engine. Because Jim's core password data had not been actively leaked onto the dark web, the User Risk score remained clean, allowing the malicious anonymous connection straight past our gates.
* **The Resolution:** To remediate this gaping security hole, I opened the Conditional Access policy settings pane. Navigated back into Conditions, turned User Risk to No, and explicitly activated the Sign-in risk policy block. This ensured that the rule focused strictly on the behavioral mechanics of the connection stream (the unverified anonymous proxy network) rather than the static state of the password object itself, immediately aligning the cloud's response with our real-world attacker model.

### 4. Technical Validation Logs

#### The Live Exploitation Simulation Test
To validate the active enforcement capabilities of the security perimeter, open an isolated, anonymous browser instance (such as a Tor browser node or an unmapped proxy connection) to simulate an outside threat vector. Navigate to the cloud user login page (`://office.com`) and input Jim Halpert's corporate email credentials. 

The authentication engine instantly intercepts the incoming network metadata, evaluates the connection profile as an untrusted high-risk anomaly, and halts the login session entirely, throwing up an absolute Multi-Factor Authentication enforcement wall on the monitor.

> **[INSERT SCREENSHOT 2.3: LIVE SIMULATION VIEW SHOWING THE SIGN-IN ATTEMPT REJECTED AND INTERCEPTED BY AN MFA ROADBLOCK]**

#### The Post-Mortem Audit Verification
To gather definitive, data-backed proof that the policy successfully applied and executed at the gateway, navigate back to the primary administrator browser session:
* Go to Identity ➔ Monitoring & health ➔ Sign-in logs.
* Locate the specific, failed authentication log row for Jim Halpert originating from the anonymous IP address block.
* Click the log entry to open the right-side detail blade and select the **Conditional Access** tab.
* Locate the policy line item `DM-CA02: Edge Access - Sign-In Risk Enforcement Perimeter`. The Result column must display a definitive, green **Success** status stamp, proving the identity engine correctly calculated the threat risk score and intercepted the exploit attempt.

> **[INSERT SCREENSHOT 2.4: ENTRA SIGN-IN LOGS DETAIL BLADE PROVING THE CONDITIONAL ACCESS POLICY SUCCESSFULLY APPLIED A GREEN SUCCESS STAMP]**
---

## LAB 3: Privileged Identity Governance & Just-In-Time Lifecycle

### 1. Scenario & Business Objective
The regional director is executing an extended off-site corporate leave, triggering a complete management power vacuum inside the local branch. Seizing this opportunity, staff member Dwight Schrute has formally demanded permanent Global Administrator directory privileges to execute high-level IT adjustments as the Assistant Regional Manager. 

Granting permanent, unrestricted root administrative rights to an active end-user account violently breaches the core security principle of Least Privilege and introduces severe configuration drift and inside-threat vulnerabilities to the enterprise tenant. The engineering objective is to deploy an enterprise-grade identity governance perimeter that provisions temporary, highly auditable administrative access that automatically revokes itself after a strict operational lifetime window.

### 2. Detailed Implementation Walkthrough

#### Phase 1: Privileged Identity Management Governance Staging
* **Governance Layer Activation:** Logged into the administrative tenant and navigated to Microsoft Entra Privileged Identity Management (PIM). Expanded the Manage menu and selected Azure AD roles ➔ Roles.
* **Role Vulnerability Remediation:** Searched the directory role database for Global Administrator. Inspected the existing assignment matrix to ensure Dwight Schrute's account possessed zero permanent, active administrative footprints. 
* **Eligible Status Provisioning:** Clicked Add assignments. Set the role target to Global Administrator. Clicked Select members, searched for Dwight Schrute, and mapped his profile. Under the Setting type configuration panel, changed the Assignment type dropdown from Active straight to **Eligible**. This ensures his default security state remains a low-privilege standard user account until he executes a formal escalation request.

> **[INSERT SCREENSHOT 3.1: PIM ROLES INTERFACE SHOWING DWIGHT SCHRUTE PROVISIONED EXCLUSIVELY AS AN ELIGIBLE GLOBAL ADMINISTRATOR]**

#### Phase 2: Lifecycle & Just-In-Time Policy Hardening
* **Activation Guardrails Configuration:** Selected the Global Administrator role settings menu and clicked Edit to rewrite the activation governance rules.
* **Temporal Window Restriction:** Modified the Maximum activation duration slider downward, locking it to a maximum threshold of **2 hours**. 
* **Mandatory Justification Enforcement:** Checked the checkbox requiring Multi-Factor Authentication on activation to verify identity truth. Checked the checkbox for **Require justification on activation**, forcing the end-user to type a detailed business-case statement into the system log before elevation is authorized.

> **[INSERT SCREENSHOT 3.2: PIM ROLE SETTINGS PANEL SHOWING THE 2-HOUR MAX LIFETIME BOUNDARY AND MANDATORY JUSTIFICATION CONTROLS]**

### 3. Engineering Challenge & Resolution Strategy
* **The Vulnerability/Error:** During validation testing, I logged into Dwight’s account and initiated the JIT role activation sequence. The system accepted the justification and successfully escalated his account permissions to Global Admin. However, upon auditing the system clock after 3 hours had elapsed, Dwight's profile remained permanently elevated as an Active Global Admin, failing the automatic revocation sweep and leaving the entire tenant completely exposed.
* **The Resolution:** Ran a full administrative role tracking audit. Discovered that Dwight’s account had been previously nested inside a legacy IT Security AD security group that had been manually granted permanent, direct Global Administrator rights outside of PIM visibility. This static inheritance layer overrode the PIM dynamic engine rules. I navigated to Groups, purged Dwight’s account entirely from the legacy security group container, and re-synchronized the identity. Running a second activation test confirmed that the tenant now tightly monitors the 2-hour window and executes a clean, automated system sweep to strip away his root administrative privileges the exact second the timer hits zero.

### 4. Technical Validation Logs
To verify that the privileged access lifecycle is functioning under tight automated guardrails, navigate to the Entra PIM resource dashboard:
* Open Privileged Identity Management ➔ Azure AD roles ➔ Audit history.
* The logs must show a multi-stage lifecycle trail: 
  1. An elevation request event capturing the user's explicit text business justification input.
  2. A multi-factor verification pass logging successful identity truth validation.
  3. A final automated system event log tracking the exact timestamp where the PIM background engine executed a clean, unassisted revocation to strip the administrative rights away, successfully locking Dwight back into a standard user security state.

> **[INSERT SCREENSHOT 3.3: PIM AUDIT HISTORY LOG SHOWING THE AUTOMATED ACCESS ELEVATION AND SUBSEQUENT SYSTEM REVOCATION TIMESTAMP]**

# Dunder Mifflin Enterprise Identity & Security Lab Suite

This repository documents the deployment, testing, and validation of an enterprise identity infrastructure built within a custom Microsoft Entra ID tenant. The environment uses a fictional branch-network scenario for Dunder Mifflin Paper Company to demonstrate core engineering capabilities across hybrid directory services, zero-trust access control, and identity governance.

---

## 🛠️ Infrastructure Labs

<details>
<summary><b>🔹 LAB 1: Hybrid Directory Architecture & Object Synchronization</b></summary>

### Scenario & Business Problem
Management wants to modernize infrastructure by migrating resources to the cloud. However, employee accounts, HR databases, and local shares are locked on a legacy on-premises server. To make matters worse, Creed Bratton has been changing his local system passwords arbitrarily, completely out of sync with IT policy. The objective is to build a unified hybrid directory control plane where on-premises identities safely replicate to Microsoft Entra ID without exposing local server assets.

### Engineering Solution
* **Local Directory Staging:** Configured a Windows Server 2022 Active Directory Domain Services (AD DS) environment inside a local virtual machine. Provisioned the organizational units and user objects using the corporate UPN suffix domain (`@dunmifflin.org`).
* **Sync Engine Deployment:** Installed and configured the Microsoft Entra Connect staging engine on the local server. Set up custom filtering rules to scope synchronization strictly to active staff members.
* **Identity Mapping:** Verified password hash synchronization (PHS) pipelines to guarantee local password changes update securely to the cloud.

### 🛑 Engineering Challenge & Resolution
* **The Roadblock:** During initial testing, the synchronization engine stalled completely. The sync logs flagged object collision errors because local accounts were not mapping to cloud destination profiles. Microsoft Entra ID locked the tenant state machine into an extended backend replication delay.
* **The Fix:** Investigated the metadata attributes and traced the issue to mismatched User Principal Names (UPNs) and missing `mailNickname` properties on the local AD server. Used a PowerShell script to audit and format the local Active Directory attributes, aligning the local `UserPrincipalName` fields exactly with the verified cloud domain routing prefix. After a directory service restart, replication cleared instantly.

### Verification Logs
The directory dashboard now confirms an active hybrid control plane showing `On-premises sync enabled: Yes` for the managed user objects.

</details>

---

<details open>
<summary><b>🔹 LAB 2: Zero-Trust Conditional Access & Sign-In Risk Policies</b></summary>

### Scenario & Business Problem
Jim Halpert is traveling out of state to Philadelphia to close a major paper deal with an international client. Dwight Schrute is highly paranoid that an outside adversary—specifically the Scranton Strangler—is impersonating Jim to steal active company sales leads. The objective is to implement an identity perimeter that stops credential theft attacks without disrupting standard employee workflows.

### Engineering Solution
* **Policy Scoping:** Built an Entra ID Conditional Access policy targeted specifically at Jim Halpert's identity object and scoped to protect all enterprise cloud applications.
* **Risk Engine Configuration:** Configured the Sign-In Risk condition. This flags anomalous background network traffic or impossible travel profiles, instantly moving the session profile into a High or Medium risk tier.
* **Enforcement Gateway:** Set the Access Control parameter to Grant Access, but appended a strict mandate requiring a Multi-Factor Authentication (MFA) challenge if the sign-in risk is tripped.

### 🛑 Engineering Challenge & Resolution
* **The Roadblock:** To validate the policy live on camera, I initially used an anonymous Tor browser routing network to simulate an attacker. However, because the policy was originally looking for *User Risk* (leaked credentials) rather than *Sign-In Risk* (anomalous connections), the proxy login attempt did not trip the firewall, allowing the connection right past the gateway.
* **The Fix:** Redesigned the security architecture to focus explicitly on authentication traffic behavior. Swapped the portal configuration toggle over to the Sign-In Risk engine to align with the real-world attacker profile. This change preserved the storyline logic while ensuring immediate threat interception at the boundary.

### Verification Logs
* **The Live Attacker Test:** When running a login attempt as Jim inside the anonymous browser routing network, the cloud immediately flagged the metadata profile and dropped a mandatory MFA roadblock onto the screen.
* **The Sign-In Analytics Log:** Checked the Entra ID Monitoring & Health dashboard. The specific log entry for Jim's anonymous attempt shows a green `Success` status for the policy rule, proving the session was actively caught and intercepted.

</details>

---

<details>
<summary><b>🔹 LAB 3: Privileged Identity Governance & Just-In-Time Lifecycle</b></summary>

### Scenario & Business Problem
Michael Scott is leaving the office for an extended corporate vacation, leaving a massive administrative power vacuum. Dwight Schrute has demanded full global admin privileges to run the branch network as the Assistant Regional Manager. Giving Dwight permanent global admin credentials violates the Principle of Least Privilege and introduces severe inside-threat risks to the tenant. The objective is to grant temporary, auditable administrative access that expires automatically.

### Engineering Solution
* **Governance Staging:** Deployed Microsoft Entra Privileged Identity Management (PIM) to take control of highly sensitive directory roles.
* **Role Configuration:** Stripped Dwight's account of all permanent administrative designations. Assigned his profile to the target security roles as an Eligible asset rather than Active.
* **Lifecycle Controls:** Configured role activation rules requiring a strict 2-hour maximum expiration lifetime, a mandatory corporate business justification statement, and multi-factor verification before elevation is granted.

### 🛑 Engineering Challenge & Resolution
* **The Roadblock:** During a simulation run, Dwight was able to activate the privileges but the system failed to trigger the automatic expiration timeout loop, leaving the account dangerously elevated in a permanent Active state.
* **The Fix:** Traced the issue to a conflicting administrative assignment group role overlap. The account was hardcoded into an old security group that bypassed PIM governance rules. Removed the direct group inheritance, re-scoped the assignment policy directly to the identity object, and verified that the tenant strictly automatically revokes the privileges the exact minute the 2-hour lifecycle expires.

### Verification Logs
The PIM governance log trail cleanly tracks the lifecycle: Dwight's account sits as an inactive "Eligible" object, captures his text justification when he requests activation, and confirms the automated system sweep successfully locks down his permissions when his time limit expires.

</details>

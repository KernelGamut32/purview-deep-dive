# Lab 07 — Cross-Solution Design: Purview + Defender XDR + Sentinel (Zero Trust in action)

**Scenario:** A “Highly Confidential – PII” file is mishandled. Microsoft Purview DLP detects it, Microsoft Defender XDR raises an incident, and Microsoft Sentinel ingests the signal for hunting and correlation. You’ll also protect a labeled SharePoint site with Conditional Access so governance and identity controls work together.

---

## What you’ll build

- A **Purview DLP** policy that alerts on U.S. SSNs leaving the organization (Exchange, SharePoint, OneDrive, Teams).
- (Optional) **Adaptive Protection** so DLP enforcement changes based on Insider Risk user risk level.
- A **Defender XDR** incident generated from the DLP alert.
- A **Sentinel** data connector + **KQL** to hunt DLP activity (`DataSecurityEvents`) and correlate alerts/incidents.
- A **sensitivity label** for SharePoint/Teams containers bound to an **Authentication context** and **Conditional Access** (CA), enforcing MFA/compliant device posture on a labeled SharePoint site.

> **Time:** 60–120 minutes  
> **Roles you’ll need (or equivalent):**  
> • Compliance Administrator (Purview)  
> • Security Administrator (Defender XDR / Sentinel)  
> • SharePoint Administrator (to apply site label)  
> • Entra Conditional Access Administrator  
> **Licensing:** Microsoft 365 E5/A5 (or E3 + relevant add-ons). Sensitivity labels for groups/sites, Authentication Contexts, DLP alerts in Defender, and Adaptive Protection require E5-class capabilities. If a feature is not present in your tenant, treat it as optional.

---

## 0) Prep

1. **Create two lab users** (examples: `Pat Sender` to simulate exfiltration; `Alex Auditor` to verify and review).  
2. **Create a SharePoint site** (example: `/sites/HC-PII`).  
3. **Verify portal access:**
   - **Microsoft Purview** (compliance) portal → *Data loss prevention*, *Information protection*, *Insider risk management*.
   - **Microsoft Defender XDR** portal → *Incidents & alerts* → *Incidents*.
   - **Azure portal** → **Microsoft Sentinel** → your Log Analytics workspace.

---

## 1) Governance + Identity: label a SharePoint site and enforce Conditional Access

### A. Create an **Authentication context** (Entra)

1. Entra admin center → **Protection** → **Conditional Access** → **Authentication context**.  
2. **New authentication context** → Name: `HC-Site-Access` → *Publish to apps* → **Save**.

### B. Create a **Conditional Access** policy that targets that context

1. Entra admin center → **Protection** → **Conditional Access** → **New policy**.  
2. **Users**: select *All users* (or a lab group).  
3. **Target resources**: choose **Authentication context** → select **HC-Site-Access**.  
4. **Grant**: require **Multi-factor authentication** (and/or **Require device to be marked as compliant**).  
5. **Enable policy**: **On** → **Create**.

### C. Create a **sensitivity label** for groups & sites

1. Purview portal → **Information protection** → **Labels** → **Create a label**.  
2. Name: `Highly Confidential – PII (Site)`; Description: “Requires MFA/compliant device via CA.”  
3. In **Groups & sites** settings, enable protection for **SharePoint sites and Microsoft Teams**.  
4. In **External sharing and device access**, select **Use Microsoft Entra Conditional Access to protect labeled SharePoint sites**, and choose the **HC-Site-Access** authentication context.  
5. **Publish** the label to your lab users.

### D. Apply the label to your site

- In SharePoint admin (or the site’s **Site information** panel), set the site’s sensitivity label to **Highly Confidential – PII (Site)**.

> **Expected outcome:** Access to `/sites/HC-PII` now invokes your CA policy (e.g., prompts for MFA). This demonstrates “verify explicitly” (Zero Trust).

---

## 2) Build a **Purview DLP policy** that alerts on PII

1. Purview portal → **Data loss prevention** → **Policies** → **Create policy** → choose **Custom**.  
2. **Name**: `Block/Alert PII Exfiltration`.  
3. **Locations**: select **Exchange**, **SharePoint**, **OneDrive**, **Microsoft Teams**.  
4. **Rules** → **Add a condition** → **Content contains** → **Sensitive info types** → add **U.S. Social Security Number (SSN)**.  
5. **Instance count / confidence**: keep default or lower the threshold for lab (e.g., ≥1).  
6. **Actions**: choose **Block with override** *or* **Audit (test with notifications)** for a low-impact demo.  
7. **User notifications**: On (policy tips optional).  
8. **Incident reports (alerts)**: On; **Severity**: Low (lab); **Alert every time**: On.  
9. **Create policy** (finish the wizard).

> **Note:** DLP alerts created here will surface as incidents/alerts in **Defender XDR**.

---

## 3) (Optional) Enable **Adaptive Protection** (risk-based DLP)

1. Purview portal → **Insider risk management** → **Adaptive protection**.  
2. Run **Quick setup** (simulation is fine for a lab), or configure policies that assign risk levels.  
3. Return to your **DLP** policy and add **Insider risk level for Adaptive Protection** conditions:
   - **Elevated risk** → **Block** (or block with override)  
   - **Moderate/Minor** → **Audit**  
   Save the policy.

---

## 4) Generate a safe DLP incident

### A. Create test content

- In Word or Excel, paste a few fake PII lines, for example:  

```Text
John Doe
SSN: 123-45-6789
```

(Use obviously fake numbers.) Save the file into the labeled site’s document library (or your OneDrive).

### B. Trigger the policy

- From **Pat Sender**:
- **Email** the file externally (attach and send to a consumer email), **or**
- **Share** the file externally from SharePoint/OneDrive (use a people-specific external share or “Anyone” link if allowed).  
- Also try accessing `/sites/HC-PII` from a session that has not completed MFA to observe the CA prompt/requirement.

> Within a few minutes, your DLP policy should generate an **alert/incident**.

---

## 5) Investigate the incident in **Microsoft Defender XDR**

1. Defender portal → **Incidents & alerts** → **Incidents**.  
2. Open the new DLP incident. Review **Alert story**, **Entities**, **Evidence**.  
3. Use **Go hunt** (if available) to pivot to advanced hunting for related activity.

---

## 6) Connect **Defender XDR** (and optionally **Purview Info Protection**) to **Microsoft Sentinel**

### A. Microsoft Defender XDR connector

1. Azure portal → **Microsoft Sentinel** → your workspace → **Data connectors**.  
2. Open **Microsoft Defender XDR** connector page.  
3. **Connect incidents & alerts**.  
4. *(Optional but recommended)* **Connect events** to stream advanced hunting events (this is what populates `DataSecurityEvents` and other tables).

### B. (Optional) Microsoft Purview Information Protection connector

- If available in your tenant, open **Microsoft Purview Information Protection (Preview)** connector and follow the setup steps to ingest label/classification activity.

> Allow a few minutes for data to begin flowing after you generate new activity.

---

## 7) Hunt the DLP activity in **Sentinel** (KQL)

Open **Microsoft Sentinel** → your workspace → **Logs** and run:

### A. Recent DLP policy matches (Advanced Hunting events)
```kusto
DataSecurityEvents
| where TimeGenerated >= ago(1d)
| where isnotempty(DlpPolicyMatchInfo) or isnotempty(SensitiveInfoTypeInfo)
| project TimeGenerated, AccountUpn, Workload, ActionType,
        DlpPolicyEnforcementMode, DlpPolicyMatchInfo,
        SensitiveInfoTypeInfo, ObjectName, SiteUrl
| order by TimeGenerated desc

SecurityAlert
| where TimeGenerated >= ago(1d)
| where ProductName has_any ("Defender", "Data Loss")
   or AlertName has_any ("Data Loss", "DLP")
| project TimeGenerated, AlertName, ProductName, Severity, ProviderName, Entities
| order by TimeGenerated desc

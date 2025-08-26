# Lab 05: Implementing Data Loss Prevention (DLP) in Microsoft Purview

> **Goal:** Plan, build, test, and monitor a Microsoft Purview DLP policy for Microsoft 365 workloads (Exchange Online, SharePoint/OneDrive, Teams). Optionally enable Endpoint DLP and Adaptive Protection. Everything below reflects current Purview UI and capabilities.

**Estimated time:** 60–90 minutes (add ~30–45 min if you also do Endpoint DLP and Adaptive Protection)

---

## Prerequisites

- **Licensing**
  - Cloud DLP for Exchange/SharePoint/OneDrive is available in Microsoft 365 E3; **Teams DLP and Endpoint DLP require E5 or E5 Compliance** (or trials).
- **Roles**
  - You need one of: **Compliance Administrator**, **Compliance Data Administrator**, **Information Protection Admin**, or **Security Administrator** to create DLP policies.
- **Portals & navigation**
  - **Microsoft Purview portal** (https://compliance.microsoft.com) → **Data loss prevention** for policies and alerts.
  - **Data classification → Activity explorer** for detailed activity views (also available from the DLP page).
- **Audit (for alerts/activity)**
  - Ensure **Audit** is on: Purview portal → **Audit** → if prompted, select **Start recording user and admin activity** (may take up to 60 minutes to take effect).
- **Test accounts/content**
  - One **sender** (your admin account is fine) and one **external address** (e.g., a personal Gmail) to test email sharing.
  - Use the test creditcard number **4111 1111 1111 1111** to trigger the **Credit Card Number** sensitive information type (SIT).
- **Optional**
  - For **Teams** file-sharing enforcement, remember Teams DLP acts on files via **SharePoint/OneDrive**—make sure those locations are included.
  - For **Endpoint DLP**, Windows 10/11 or recent macOS devices must be **onboarded**; Microsoft Defender for Endpoint (MDE) or MDM onboarding is supported.

---

## Exercise 1 — Plan your policy (5–10 min)

1. Draft a **policy intent statement** answering:
   - What data? (e.g., **Payment card data**)
   - Where? (Exchange, SharePoint/OneDrive, Teams; optionally **Devices**)
   - What behaviors to restrict? (external sharing, emailing outside, copying to USB, printing, etc.)
   - User experience? (show **policy tips** first, then **block with override**, then **block**)
2. Map the intent to Purview DLP components: **Locations** (workloads), **Conditions** (Sensitive information types), **Actions** (block/override/audit), **User notifications**, and **Incident/alerting**.

> **Tip:** Start in **simulation** (test) mode with notifications, then move to enforcement after you validate matches.

---

## Exercise 2 — Create a DLP policy (simulation mode) (10–15 min)

1. In the **Microsoft Purview** portal, go to **Data loss prevention → Policies → Create policy**.
2. **Choose a template**: select **Financial → U.S. Financial Data** (or **Custom policy** if you prefer to build from scratch). Templates include **Credit Card Number** and other common SITs.
3. **Name & describe** the policy (e.g., “DLP – Payment Cards (Pilot)”).
4. **Choose locations**: **Exchange email**, **SharePoint sites**, **OneDrive accounts**, **Microsoft Teams chat and channel messages**. (Leave **Devices** unchecked for now; you’ll add it in the endpoint exercise.)
5. **Configure rules** (edit the default rule if using a template):
   - **Condition:** *Content contains* → **Sensitive info types** → **Credit Card Number** → **min count = 1**.
   - **Action (simulation):** No blocking yet.
   - **User notifications:** **On** → show **policy tips** and send email notification to the user who triggered the match.
   - **Incident reports (alerts):** **On** → **Severity = Medium**, **Send alert every time an activity matches**, **Recipients = you**.
6. **Mode**: choose **Test it out first (simulation)** with notifications. Finish and **Create**.

---

## Exercise 3 — Generate matches & see the end-user experience (10–15 min)

### A. Outlook on the web (policy tips)

1. Open **Outlook on the web (OWA)** as the **sender**.
2. Compose a new email **to an external address**. Subject: “Expense receipts”. Body: include the test number `4111 1111 1111 1111`.
3. **Observe** the **DLP policy tip** banner in OWA warning about sensitive info.  
   - **Note:** Policy Tips support varies by client; for this lab use **Outlook on the web** to ensure you see them.
4. **Send** the message (in simulation it will send). You should also receive the **notification email** you configured.

### B. OneDrive/SharePoint & Teams (file sharing)

1. In **OneDrive**, create a **Word document** “CustomerAccounts.docx” with the same test number inside.
2. Attempt to **share externally** (e.g., “Anyone with the link” or a specific external user).  
   - In **Teams**, share or attach the same file in a chat/channel.  
   - **Note:** Teams file protection relies on SharePoint/OneDrive being included in the policy.
3. In simulation mode, you’ll see **policy tips** (where supported) and the share may still complete.

---

## Exercise 4 — Review alerts & activity (10–15 min)

> There can be a short delay before events appear. If you don’t see anything, wait a few minutes and refresh.

1. **Alerts:** Purview portal → **Data loss prevention → Alerts**.  
   - Open your new alert → **View details** → examine **Events** and try **Copy event link**.
2. **Activity explorer:**  
   - Purview portal → **Data loss prevention → Activity explorer** (or **Data classification → Activity explorer**).  
   - Filter by **Location**, **Policy/Rule**, **Action**, and **User** to see your test activities for the last 30 days.

---

## Exercise 5 — Move to enforcement & retest (10–15 min)

1. Edit your policy rule: **Actions** → choose **Block with override** for low match count (e.g., ≥1) and **Block** for higher threshold (e.g., ≥5). Save. (Exact thresholds are up to you.)
2. Change **Mode** to **Turn it on right away** (enforced).
3. **Retest OWA** and **OneDrive/Teams**:
   - You should now be **blocked** or **asked to justify** (override) depending on thresholds.
   - Verify **new alerts** are generated and review in **Activity explorer**.

---

## SC-100 Tie-in — Strategy & architecture (5–10 min)

- **Zero Trust pillars**  
  - **Identity** (Adaptive Protection ties user **risk** to controls),  
  - **Data** (DLP policies & labels),  
  - **Devices** (Endpoint DLP),  
  - **Apps** (Teams/SharePoint/OneDrive),  
  - **Network/Threat** (alerts feed investigations).
- **Adaptive controls**  
  - Risk-based DLP and Conditional Access can block downloads or access for elevated-risk users—strong example of **defense in depth**. (Entra ID P2 is required for the Conditional Access insider-risk condition.)

---

## Troubleshooting notes

- **Don’t see alerts/activity yet?** Short processing delays are normal; refresh after a few minutes. Verify **Audit** is turned on.
- **Policy tips didn’t appear?** Use **Outlook on the web** (supported). Other clients have varying support; for this lab, stick to OWA to validate behavior.
- **Teams file scenarios** require that **SharePoint/OneDrive** locations are included in your policy.

---

## Cleanup

1. Purview → **Data loss prevention → Policies**: set your policy to **Off** or delete it.
2. **Alerts**: resolve/close test alerts to keep dashboards clean.
3. If you onboarded devices for Endpoint DLP, offboard test devices when finished.

---

## What you should have by the end

- A documented **policy intent** tied to real DLP components.
- A working **DLP policy** in simulation and enforcement modes.
- Hands-on experience with **policy tips**, **alerts**, and **Activity explorer**.

---

### Appendix — Fast test content you can paste

```Text
Here’s the customer’s card for the hotel:
4111 1111 1111 1111
```

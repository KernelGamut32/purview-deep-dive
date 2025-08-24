# Lab 03: Creating & Managing Sensitive Information Types (SITs) in Microsoft Purview

## What you’ll do

1. Explore built-in vs. custom Sensitive Information Types (SITs)  
2. Create a custom **regex-based** SIT and test it  
3. Create a **document fingerprinting** SIT and enforce it with a DLP policy  
4. Use **Named Entities** in a DLP policy (understand what you can/can’t “define”)

**Estimated time:** 75–90 minutes  
**Where:** Microsoft Purview portal — <https://purview.microsoft.com>

---

## Prerequisites

- **Licensing/roles:** Microsoft 365 E5 (or an active Purview trial) and one of: **Compliance Administrator**, **Compliance Data Administrator**, or **Information Protection Administrator**
- **Test users:** At least one **standard user** (for end-user validation) and your **admin** account
- **Locations enabled:** Exchange Online, SharePoint Online, OneDrive for Business, Microsoft Teams (for DLP validation)
- **Portal navigation used in this lab:**  
  **Information protection → Classifiers → Sensitive info types**  
  **Data loss prevention → Policies**

> Note: For at-rest discovery of *new* custom SITs in SharePoint/OneDrive, existing content may need to be **recrawled** to reflect detections. For immediate checks, use the **Test** feature on a SIT or validate through DLP events (policy tips/blocks).

---

## Part A — Built-in vs. Custom SITs (10–15 min)

1. Sign in to <https://purview.microsoft.com>.  
2. Go to **Information protection → Classifiers → Sensitive info types**.  
3. Use **Search** to find built-in SITs such as **U.S. Social Security Number** or **Credit Card Number**.  
4. Open a built-in SIT and review:
   - **Patterns** (primary element + supporting elements)
   - **Confidence levels** (Low/Medium/High) and thresholds
5. In a built-in SIT, click **Copy** (you can create a customizable copy). Cancel/discard the copy—next you’ll build your own from scratch.

## Key takeaways

- Built-in SITs are Microsoft-defined. You **can’t** edit them directly, but you **can** copy many of them and customize the copy.
- Custom SITs are fully authored by you (regex/keywords/functions, proximity, and confidence scoring).

---

## Part B — Create & Test a Custom (Regex-Based) SIT (20–25 min)

**Scenario:** Detect “Contoso Project Codes” that look like `CPC-123AB4567` when they appear near relevant keywords.

### B1) Create the SIT

1. Go to **Information protection → Classifiers → Sensitive info types**.  
2. Select **Create sensitive info type**.
3. **Name:** `Contoso Project Code (CPC)`  
   **Description:** Detects project codes like `CPC-123AB4567` in context.
4. **Create pattern** → **Default confidence:** *Medium*.
5. **Primary element**
   - **Type:** Regular expression
   - **Regex:** `CPC-\d{3}[A-Z]{2}\d{4}`
   - **Character proximity:** `50`
6. **Supporting elements** (recommended)
   - **Keyword list:** `Project Code`, `Contoso Program`, `CPC`
   - Keep default proximity (e.g., 50)
7. Leave other options at defaults → **Next** → confirm **Recommended confidence**: *Medium* → **Create/Save**.

> Tip: Avoid anchoring with `^`/`$` in SIT regex unless specifically documented—content can be scanned in segments.

### B2) Prepare test files

- **Positive sample** (Word or .txt) containing:  
  “The assigned **Project Code** for phase one is **CPC-123AB4567** and will be referenced in all SOWs.”
- **Negative sample** with either a near-miss (e.g., `CPC-123A-4567`) or no CPC at all.

### B3) Test the SIT (ad-hoc)

1. In **Sensitive info types**, open **Contoso Project Code (CPC)**.  
2. Click **Test** → **Upload** the positive sample → **Test**. Expect a **match** (Medium or above).  
3. Upload the negative sample → confirm **no match**.

---

## Part C — Document Fingerprinting SIT + DLP Policy (25–30 min)

**What it is:** Upload a *standard form/template* (e.g., an HR form). Purview creates a **fingerprint-based** SIT that detects documents derived from that template. You can tune **Partial** vs **Exact** matching per confidence level.

### C1) Create a fingerprint-based SIT

1. Create a template file (≥256 characters, not password-protected), e.g., **HR-NewHire-Template.docx** with content like:

```Text
CONTOSO – NEW HIRE INFORMATION FORM (TEMPLATE)

Section 1 – Employee Details
Full Name: ___________________________
Start Date: ___________________________
Department: __________________________
Position Title: _______________________
Manager Name: ________________________
Work Location: _______________________

Section 2 – Personal Contact
Personal Email: ______________________
Mobile Phone: ________________________
Emergency Contact Name: ______________
Emergency Contact Phone: _____________

Section 3 – Acknowledgements
I acknowledge receipt of the Contoso Employee Handbook and agree to follow applicable policies including security, privacy, and acceptable use. I understand that my manager will assign me a project code and that I will comply with data handling requirements. (Initial here): ______
```

2. In the portal, navigate to **Information protection → Classifiers → Sensitive info types**.  
3. Select **Create fingerprint-based SIT** (wording may appear as **+ Create fingerprint sensitive info type**).
4. **Name:** `Contoso New Hire Form (Fingerprint)`  
**Description:** Detects documents derived from the Contoso New Hire template.  
5. **Upload** the template file (**HR-NewHire-Template.docx**).  
6. Leave default confidence settings (you can edit later to tune **Partial** vs **Exact**).  
7. **Create**.

> Notes and limits: Use template files with sufficient plain text; avoid embedded/linked docs; keep files under size limits; don’t password-protect.

### C2) Create a DLP policy using the fingerprint SIT

1. Go to **Data loss prevention → Policies → Create policy → Custom**.  
2. **Name:** `Block New Hire Form external` → **Next**.  
3. **Locations:** Enable **Exchange**, **SharePoint**, **OneDrive**, **Teams** (you can also extend to **Endpoints** later). → **Next**.  
4. **Rules** → choose **Create or customize advanced DLP rules** → **Create rule**:
- **Condition:** *Content contains* → **Sensitive info types** → add **Contoso New Hire Form (Fingerprint)** with **Medium** (or **High**) confidence.
- **Actions:** **Block** external sharing/sending; enable **User override with justification** (optional).
- **User notifications:** **On** (policy tips + email).  
- **Incident reports:** **On** (to admins/SECops).  
- **Mode:** Start in **Test with notifications**; switch to **Enforce** after validation.
5. **Publish** the policy.

### C3) Validate the DLP policy

1. Create a filled-in copy of the template named **HR-NewHire-Filled.docx** (retain most of the template text).  
2. Validate one or more of the following:
- **Exchange:** Send the file to an **external** recipient (e.g., a personal mailbox). Expect a **policy tip** and a **block** or **override** prompt.  
- **SharePoint/OneDrive:** Upload the file, then attempt to **Share** with an external email. Expect block/override flow.  
- **Teams:** Paste substantial portions of the form text in a chat/channel. Expect a tip/block (depending on your action settings).

---

## Part D — Named Entities: Use in Policies (15–20 min)

> **Important:** **Named Entities** (e.g., *Person name*, *Physical address*, *Medical terms*) are **prebuilt** Microsoft classifiers. You **use** them in policies, but you **cannot author, copy, or edit** them like custom SITs.

### D1) (Optional) Endpoint/advanced classification prerequisites

- If you plan to enforce on **Endpoints** (devices), ensure **Advanced classification** / endpoint DLP is enabled under **Data loss prevention → Settings** and that the client prerequisites are met.

### D2) Create a DLP policy that uses Named Entities

1. Go to **Data loss prevention → Policies → Create policy** (choose **Custom** or a relevant template).  
2. **Locations:** Select **Exchange**, **SharePoint**, **OneDrive**, **Teams** (Named Entities are supported here; not in on-prem sources).  
3. **Rules** → **Create or customize advanced DLP rules** → **Create rule**:
- **Condition:** *Content contains* → **Sensitive info types** → add:
  - **All physical addresses** (bundled named-entity SIT)
  - **Person name** (unbundled named-entity SIT)
- Set **min instances** = 1 for each, **Confidence** = Medium.  
- **Actions:** Notify + block external sharing; allow overrides for specific groups if desired.  
- **Mode:** Start in **Test with notifications**, then **Enforce**.
4. **Publish**.

### D3) Validate Named Entities detection

- Send/upload content like:  
“**John Rivera** visited **1200 5th Ave, Seattle, WA 98101** to meet Dr. Patel regarding follow-up care.”  
Expect detections for **Person name** and **Physical address** per your policy settings.

**Named Entities recap**
- Prebuilt, Microsoft-maintained; **not** user-authorable.  
- Usable in DLP and auto-labeling (where supported).  
- Good for PII scenarios when you don’t have fixed identifiers.

---

## Clean-up (2–5 min)

- In **Data loss prevention → Policies**, **turn off** or **delete** the policies you created.  
- In **Information protection → Classifiers → Sensitive info types**, delete the **Contoso Project Code (CPC)** and **Contoso New Hire Form (Fingerprint)** SITs if this is a shared/teaching tenant.

---

## Tips & Troubleshooting

- **Custom SIT not matching in existing files?** Allow time for SharePoint/OneDrive **recrawl**, or use **Test** on sample content to verify the pattern first.  
- **Over-triggering?** Increase **confidence thresholds**, require **supporting keywords**, or reduce **proximity** range.  
- **Under-triggering?** Lower thresholds, expand supporting keywords, increase proximity, or relax your regex pattern.  
- **Fingerprint SIT too “sensitive”?** Adjust **Partial** vs **Exact** matching at the confidence level for fewer/more matches.  
- **Policy tips not showing?** Confirm the policy is published to the **right locations**, in **Enforce** or **Test with notifications** mode, and that client apps/users are in scope.

---

## What students should be able to explain after the lab

- Differences between **built-in**, **custom**, **fingerprint**, and **named-entity** SITs  
- How **patterns**, **proximity**, and **confidence** drive detection  
- How to **validate** SITs quickly (Test) and operationally (DLP)  
- Practical trade-offs when tuning **false positives/negatives** in real content

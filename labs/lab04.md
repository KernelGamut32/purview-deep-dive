# Lab 04: Sensitivity Labels in Microsoft Purview

> **Goal:** Create, publish, and monitor sensitivity labels—including one with encryption—then simulate and enable an auto-labeling policy. Close with a quick Zero Trust/SC-401 debrief.

---

## What students will do
1. Create three sensitivity labels (one with encryption)
2. Publish labels via a label publishing policy (default/mandatory/justification)
3. Build an **auto-labeling policy** in simulation mode, review results, then enable it
4. Track label usage with **Reports**, **Activity explorer**, and **Content explorer**
5. Connect the work to Zero Trust / SC-401 strategy

---

## Prerequisites (instructor checklist)

- **Tenant & permissions**
  - Access to the **Microsoft Purview (compliance) portal**: https://compliance.microsoft.com
  - Role group membership for label administration, e.g. **Information Protection Administrator** or **Compliance Administrator**
- **Enable labeling in SharePoint/OneDrive**
  - In Purview, go to **Solutions → Information protection → Sensitivity labels**
  - If you see a banner to enable sensitivity labels for SharePoint and OneDrive, select **Turn on now**
- **Auditing**
  - Microsoft 365 **auditing** should be on (required for Activity explorer and auto-labeling simulation visibility)
- **Content Explorer permissions (optional, for preview)**
  - Add your learner/demo accounts to:
    - **Content Explorer List Viewer** (to list locations)
    - **Content Explorer Content Viewer** (to preview file contents)

> **Tip:** Use a small **pilot group** (e.g., `Label-Pilot`) for publishing, so students immediately see the impact without affecting your whole tenant.

---

## Exercise 1 — Create three sensitivity labels

### 1.1 Create “Public” (no protection)
1. In Purview: **Solutions → Information protection → Sensitivity labels → Create label**
2. **Name:** `Public - Test`  
   **Description for users:** `Intended for non-sensitive content.`
3. **Scope:** Select **Files & emails** (or **Files & other data assets** + **Emails**, depending on your tenant wording)
4. Leave **Access control (encryption)** and **Auto-labeling** off
5. **Content marking:** Off
6. **Review and create** → **Create**

### 1.2 Create “Confidential” (visual markings only)
1. **Create label** → **Name:** `Confidential - Test`
2. **Scope:** Files & emails
3. **Access control (encryption):** Off
4. **Content marking:** On  
   - **Header:** `Confidential - Test` (set font/size as desired)  
   - (Optional) **Footer** or **Watermark** with “Confidential”
5. **Review and create** → **Create**

### 1.3 Create “Highly Confidential – Finance (Encrypted)”
1. **Create label** → **Name:** `Highly Confidential – Finance (Encrypted)`
2. **Scope:** Files & emails
3. **Access control (encryption):** **On**
   - **Assign permissions** → **Add groups or users** → choose your **Finance** M365 group (or a demo group)
   - **Permissions:** Select a preset (e.g., **Editor**) or choose **Custom** and allow: View, Edit; block Print/Copy as needed
   - (Optional) **Content expiration**: Off for class; **Offline access**: allow a few days
4. (Optional) **Dynamic watermark**: On (supported in Office apps; demo value adds)
5. **Content marking:** (Optional) Add **Header**: `Highly Confidential – Finance`
6. **Review and create** → **Create**

> **Label order matters.** In the labels list, ensure higher-sensitivity labels (e.g., *Highly Confidential*) are ordered **lower** than less-sensitive ones (lower in the list = higher priority).

---

## Exercise 2 — Publish labels with a label policy

> Newer tenants may list this as **Publishing policies** under Information protection.

1. Go to **Solutions → Information protection → Publishing policies** (or **Label policies**) → **Publish label**
2. **Select labels:** Add `Public - Test`, `Confidential - Test`, and `Highly Confidential – Finance (Encrypted)`
3. **Choose users and groups:** Select your pilot group (e.g., `Label-Pilot`)  
   > Keep scope small for fast, observable results
4. **Policy settings (Documents & Email)**
   - **Require users to apply a label:** **On** (mandatory labeling)
   - **Require justification to remove or lower a label:** **On**
   - **Apply a default label to documents and emails:** **Confidential - Test**
5. **Review and finish** → **Create**

> **Propagation:** Allow time for labels/policies to appear across clients. Office on the web typically reflects changes sooner than desktop apps.

---

## Exercise 3 — Test labeling in Office

1. Sign in to **Word for the web** (or desktop) with a user in the policy’s target group
2. Create a document in **OneDrive** or a **SharePoint** library
3. Use the **Sensitivity** menu to:
   - Apply **Public - Test** → Save → observe absence of markings
   - Apply **Confidential - Test** → Save → verify header/footer/watermark
   - Apply **Highly Confidential – Finance (Encrypted)** → Save → sign out/in with a **non-Finance** user to confirm access is blocked
4. Try to **downgrade** from **Highly Confidential – Finance (Encrypted)** to **Confidential/Public - Test** and capture the **justification** prompt (policy setting)

---

## Exercise 4 — Build an **auto-labeling policy** (service-side)

> Auto-labeling can target **Exchange**, **SharePoint**, and **OneDrive**. Policies start in **simulation mode** so you can validate before applying.

### 4.1 Create the policy
1. In Purview: **Solutions → Information protection → Policies → Auto-labeling policies → Create auto-labeling policy**
2. **Choose a label to auto-apply:** select `Confidential - Test` (or create a dedicated sublabel if you prefer)
3. **Choose info to match:** Select a built-in template (e.g., **Privacy**) and include **U.S. Personally Identifiable Information (PII) Data Enhanced**
4. **Locations:** Select **Exchange**, **SharePoint**, and **OneDrive** (or limit to specific sites/mailboxes for class)
5. **Rules:** Keep **Common rules**; ensure SSN condition is enabled
6. **Mode:** **Run policy in simulation mode**
7. **Create**

### 4.2 Generate matches and review
1. In **OneDrive**, create a Word document containing sample data such as:  
   `Employee: Jane Doe — SSN: 123-45-6789` *(use obviously fake values)*
2. Back in **Auto-labeling policies**, open your policy and go to **Items to review**
3. Confirm the uploaded file appears as a **match**; repeat with 1–2 more files to see multiple results
4. If the simulation shows too many or too few hits, **Edit** the policy and adjust conditions, then **rerun simulation**

### 4.3 Enable the policy
1. When satisfied, change **Simulation** → **On (apply)** to begin applying the label
2. For labels with **encryption**, ensure requirements are met (e.g., defined permissions). If you used `Confidential` (no encryption), this is straightforward

---

## Exercise 5 — Track label usage and outcomes

### 5.1 Information protection **Reports**
- **Solutions → Information protection → Reports**  
  Review trend charts for label application, most-used labels, and locations

### 5.2 **Activity explorer** (who did what, where)
1. **Data classification → Activity explorer**
2. Filter **Activity** to **Label applied**, **Label changed**, and **Auto-labeling simulation**
3. Narrow by **Workload** (SharePoint/OneDrive/Exchange) and timeframe to see your lab actions

### 5.3 **Content explorer** (what/where the content is)
1. **Data classification → Content explorer**
2. Filter by **Sensitivity label** (e.g., `Confidential`)
3. Drill down into **SharePoint** or **OneDrive**; if you have **Content Viewer** rights, preview redacted content where supported

---

## SC-401 / Zero Trust tie-in (debrief)

- **Least privilege & data-centric protection:** Labels with encryption restrict *who* can access and *how* they can use content (view/edit/print/copy)
- **Defense in depth:** Combine **mandatory/default labeling** (client prompts), **service-side auto-labeling** (at rest/in transit), and **monitoring** (Reports/Explorers)
- **Assume breach posture:** Even if a file leaves your perimeter, **usage rights** persist with the file—aligning with Zero Trust principles

---

## Cleanup (post-class)

1. **Publishing policies:** Edit the policy to remove your pilot group or **Delete** the policy
2. **Auto-labeling:** Set the policy **Off** or **Delete** it
3. **Labels:** Keep for future modules or delete the three labels created for this lab

---

## Troubleshooting

- **Labels/policy not visible in Office:** Give it time; confirm the user is in the policy scope and using a supported client
- **Cannot co-author encrypted files:** Ensure SharePoint/OneDrive support is enabled; note limitations for HYOK/DKE
- **No simulation results:** Confirm **Audit** is on; verify your test content matches the selected **sensitive info types**; rerun simulation
- **Label priority issues:** Reorder labels so that **most sensitive** are **lowest** in the list (highest priority)

---

## Appendix — Quick reference paths

- **Create labels:** Solutions → Information protection → **Sensitivity labels**
- **Publish labels:** Solutions → Information protection → **Publishing policies**
- **Auto-labeling:** Solutions → Information protection → Policies → **Auto-labeling policies**
- **Reports:** Solutions → Information protection → **Reports**
- **Activity explorer:** **Data classification → Activity explorer**
- **Content explorer:** **Data classification → Content explorer**

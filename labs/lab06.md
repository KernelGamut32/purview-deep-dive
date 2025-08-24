# Lab 06: Data Retention & Records Management in Microsoft Purview (Module 7)

A hands-on lab that matches today’s **Microsoft Purview** portal UI. Students will create static and adaptive retention policies, build adaptive scopes, configure event-based retention, complete a disposition review, and connect the design to SC-100 architecture considerations.

---

## 📌 Learning objectives

By the end of this lab, you will be able to:

- Create **static** and **adaptive** retention policies in Microsoft Purview.
- Build **adaptive scopes** (Users/Sites/Groups) and use them in policies and label publishing.
- Configure **event-based retention** (event types, labels, asset IDs, events).
- Run a **disposition review** (single or multi-stage) and export proof of disposition.
- Tie retention and records decisions to **data residency** and **regulatory** requirements (SC-100 context).

---

## 🧰 Prerequisites (instructor/tenant)

- Microsoft 365 test tenant with access to the **Microsoft Purview portal**.
- Roles:
  - **Compliance Administrator** (or equivalent) for policy/label configuration.
  - **Scope Manager** (or equivalent) for **Adaptive scopes**.
  - **Disposition Management** for reviewers (often via a dedicated **mail-enabled security group**, e.g., `Disposition Reviewers`).
- Licensing: Microsoft Purview **Records management** features (E5 or Purview solutions trial).
- A SharePoint site for the lab (e.g., `https://<tenant>.sharepoint.com/sites/RecordsLab`) with a **Documents** library.
- (Recommended) Upload a few test files **2–3 days before class** so disposition demos have content ready.

> **Propagation timing note**  
> Retention policies, label publishing, and event synchronization can take time to deploy. Pre-stage content so you can demonstrate results during the session.

---

## 🧭 Scenario used in this lab

**Contoso Finance**

- Teams **chats** for Finance users: retain **3 years** and then delete.
- All SharePoint/OneDrive content: retain **7 years** and then delete.
- HR records: labeled as **records** and require a **2-stage disposition**.
- Offboarding: **event-based retention** triggered by an **Employee Separation** event using an **Asset ID** (employee ID).

---

## Part A — Static retention policy for SharePoint & OneDrive (7 years)

1. Go to **Microsoft Purview portal** → **Solutions** → **Data Lifecycle Management** → **Policies** → **Retention policies** → **New retention policy**.
2. **Name**: `Global SPO/OD Retain 7y`.
3. **Scope type**: **Static**.
4. **Choose locations**: Turn **On** *SharePoint sites* and *OneDrive accounts* (leave others Off for this policy).
5. **Retention settings**:  
   - **Retain items for a specific period** → `7 years`.  
   - **At the end of the retention period** → **Delete items automatically**.
6. **Review** → **Create**.

> **Tip**: Teams **files** are stored in SharePoint/OneDrive and are covered by SharePoint/OneDrive retention. Teams **messages** require their own locations in a separate policy.

---

## Part B — Adaptive scopes + adaptive Teams chats policy (3 years)

### B1. Create an **adaptive scope** (Users)

1. In the Purview portal, select **Settings** (gear) → **Roles and scopes** → **Adaptive scopes** → **+ Create scope**.
2. **Scope type**: **Users**.
3. **Query builder**:  
   - `Department = Finance`  
   - `Country or region = United States`
4. **Name**: `UserScope-Finance-US` → **Create**.

> **About adaptive scopes**: They evaluate daily against Entra ID/SharePoint attributes and keep policy/label targeting current as people or sites change—no manual includes/excludes.

### B2. Create an **adaptive** retention policy for **Teams chats** (3 years)

1. **Solutions** → **Data Lifecycle Management** → **Policies** → **Retention policies** → **New retention policy**.
2. **Name**: `Teams Chats (Finance US) Retain 3y`.
3. **Scope type**: **Adaptive**.
4. **Add scope**: Select `UserScope-Finance-US`.
5. **Locations**: Turn **On** **Teams chats** (leave channel/private channel messages Off for this policy).
6. **Retention settings**:  
   - **Retain items for** `3 years`.  
   - **Delete items automatically** after the period.
7. **Create**.

---

## Part C — Records management labels with disposition

We’ll create a label that **marks items as records** and uses **2-stage disposition**.

### C1. Create a **retention label** (record, 2-stage disposition)

1. **Solutions** → **Records Management** → **File plan** → **+ Create a label** → **Retention label**.
2. **Name**: `HR-Record-2y-2StageDisp`.  
   Add user/admin descriptions as needed.
3. **Retention**:  
   - **Retain items for a specific period** → `2 years`.  
   - **At end of retention** → **Start a disposition review**.  
   - **Mark items as a record** (standard record).
4. **Disposition reviewers**:  
   - **Stage 1**: e.g., `HR Reviewers` (mail-enabled security group).  
   - **Stage 2**: e.g., `Compliance Officers` (mail-enabled security group).  
   - (You can define up to **5 stages**; prefer groups, max **10 reviewers per stage**.)
5. **Create** the label.

### C2. Publish the label (label policy)

1. **Records Management** → **Policies** → **Label policies** → **Publish labels**.
2. Select **`HR-Record-2y-2StageDisp`**.
3. **Scope type**:  
   - **Static** → include your `RecordsLab` SharePoint site, **or**  
   - **Adaptive** → select an adaptive **Site/User** scope if you built one.
4. **Create** the policy and **turn it on**.

### C3. Apply the label to SharePoint documents

1. In the `RecordsLab` SharePoint site → **Documents** library.
2. Select several files → **Details** pane → **Compliance details** → **Apply retention label** → choose **`HR-Record-2y-2StageDisp`**.

---

## Part D — Event-based retention (Offboarding)

You’ll create an event-based label, publish it, apply it to a document, set an **Asset ID**, and then trigger an **Event**.

### D1. Create an **event-based** retention label

1. **Records Management** → **File plan** → **+ Create a label**.
2. **Name**: `HR-Offboarding-Event-1d-Disp` (short period for demo).
3. **Retention**:  
   - **Retain items for a specific period** → `1 day`.  
   - **Start retention based on** → **Create new event type** → **Name**: `Employee Separation` → **Create**.  
   - **At end of retention** → **Start a disposition review**.  
   - (Optionally) **Mark items as a record**.
4. **Create** the label.

### D2. Publish the event-based label

- **Records Management** → **Policies** → **Label policies** → **Publish labels** → select **`HR-Offboarding-Event-1d-Disp`** → target the `RecordsLab` site (static or adaptive) → **Create**.

### D3. Apply the label and set an **Asset ID** on a document

1. In `RecordsLab` → **Documents**, upload or pick a file (e.g., `OffboardingChecklist-EID1001.docx`).
2. **Details** pane → **Compliance details** → **Apply** label **`HR-Offboarding-Event-1d-Disp`**.
3. In the **Asset ID** field (SharePoint/OneDrive), enter `EID-1001`.  
   - This is stored in the **ComplianceAssetId** managed property.

> **Note**: If the Asset ID field is read-only, ensure an **event-based** retention label is applied first.

### D4. Trigger an **Event** to start retention

1. **Solutions** → **Records Management** → **Events** → **+ Create**.
2. **Event type**: **`Employee Separation`**.
3. **Asset matching**: `ComplianceAssetId:EID-1001` (you can match by property:value or keywords).  
   - If you omit asset IDs/keywords, the event targets **all** content with labels of that event type.
4. **Event date**: Today → **Create**.

---

## Part E — Disposition review (reviewers’ experience)

> Use pre-staged items so you can demonstrate items ready for disposition.

1. **Solutions** → **Records Management** → **Disposition**.
2. (One-time admin) **Settings** → **Solution settings** → **Records Management** → **Disposition**: choose a **mail-enabled security group** whose members can see **all** disposition items (e.g., `Records Managers`).
3. In **Disposition**, filter by **Label**, **Location**, or **Reviewer stage**.  
4. Select an item → review metadata and (if permitted via **Content Explorer Content Viewer**) preview content.  
5. Choose an action: **Dispose (delete)**, **Extend**, **Relabel**, or **Export proof of disposition**.

---

## Part F — Validate and troubleshoot

- **Policy/label status**:  
  - **Data Lifecycle Management** → **Policies** → **Retention policies** → open a policy to view deployment status.  
  - **Records Management/Data Lifecycle Management** → **Policies** → **Label policies** for publishing status.
- **Find labeled content**: Use **Content search** (e.g., search by **Retention label**) or KQL in SharePoint search (e.g., `ComplianceAssetId:EID-1001`).
- **Common blockers**:
  - Propagation delays (policy/label/event distribution).  
  - Missing roles (e.g., **Disposition Management** for reviewers, **Content Explorer Content Viewer** for previews).  
  - Using static scopes where adaptive scopes would reduce include/exclude limits.

---

## Part G — Optional: Adaptive **label** policies

- Publish labels using **adaptive** label policies so only scoped users/sites see them:  
  **Records Management**/**Data Lifecycle Management** → **Policies** → **Label policies** → **Publish labels** → **Adaptive** → select your scopes → **Create**.

---

## Part H — Clean up (optional)

- Disable or delete demo **retention policies** and **label policies**.  
- Delete demo **labels** (only if unused and not regulatory/event-based).

---

## SC-100 tie-in: Residency, retention, and regulatory alignment

- **Data residency**: Design for where data at rest resides (e.g., multi-geo). Confirm which Purview data is stored in which regions and align to regulatory constraints.  
- **Retention as a control**: Map your policies/labels to obligations (e.g., SEC/FINRA/GDPR) using **Compliance Manager** assessments and templates.  
- **Operating model**: Prefer adaptive scopes for scale, mail-enabled security groups for disposition reviewers, and clear RBAC separation of duties.

---

## Knowledge checks

1. Where do you build **adaptive scopes** in the Purview portal?  
2. What document property ties a SharePoint/OneDrive file to an **Event** for event-based retention?  
3. Where do reviewers go to act on items that have reached the end of retention?

**Answers**:  
1) **Settings → Roles and scopes → Adaptive scopes**.  
2) The **Asset ID** (managed property **ComplianceAssetId**).  
3) **Solutions → Records Management → Disposition**.

---

## Quick UI path cheat sheet

- **Retention policies** (static/adaptive): **Solutions → Data Lifecycle Management → Policies → Retention policies**  
- **Adaptive scopes**: **Settings → Roles and scopes → Adaptive scopes**  
- **Retention labels (File plan)**: **Solutions → Records Management → File plan → + Create a label**  
- **Publish labels** (label policies): **Solutions → Records Management** or **Data Lifecycle Management → Policies → Label policies → Publish labels**  
- **Events** (event types & events): **Solutions → Records Management → Events**  
- **Disposition**: **Solutions → Records Management → Disposition**

## Prep (do this first)

1. Verify **auditing is on** in your tenant (Purview → Audit).  
2. **Seed activity** so there’s data to find later:
   - Send/receive a few emails between two test users (e.g., **Allan Deyoung [alland@`tenant-name`]** and **Adele Vance [adelev@`tenant-name`]**).
   - In OneDrive/SharePoint, upload, rename, share, and download a test file.
   - Post a message in a Microsoft Teams channel.

---

## Standard vs Premium quick recap

- **Standard Audit**  
  - Enabled by default in most tenants.  
  - Retention commonly up to **180 days** (varies by record type and licensing).  
  - Portal search and CSV export.
- **Audit (Premium) / Advanced Audit**  
  - Longer default retention for key workloads (e.g., Exchange, SharePoint/OneDrive, Entra ID).  
  - **Audit log retention policies** to keep specific record types/users longer (up to **10 years** with the add-on).  
  - Access to additional mailbox activities (e.g., **MailItemsAccessed**) and higher-throughput APIs for export.

---

# Lab 10: Audit Essentials — Access, Roles, Search & Export

**Estimated time:** 45–60 minutes

### Learning objectives
- Verify tenant audit status and required permissions.
- Generate common audited activities.
- Search the unified audit log and interpret results.
- Export and shape audit data for simple analysis.

---

## A) Verify auditing & enable if needed

1. Open the **Microsoft Purview** portal and sign in.
2. From the left nav, go to **Solutions → Audit** (or **View all solutions → Audit**).
3. If a banner appears that says **Start recording user and admin activity**, select it to enable auditing. Note: after enabling, events can take up to about an hour to appear.

---

## B) Assign the correct Purview roles

1. In Purview, select **Settings** (gear) → **Roles & scopes** → **Role groups**
2. Search for and open **Audit Reader** (minimum to search/export).  
   - Alternative: **Audit Manager** (includes search/export and management of audit settings).
3. Edit, add the `MOD Administrator` account, click **Select**, click **Next**, and click **Save**
4. Click **Done**

> Minimum required permission to search the audit log is included in the **Audit Reader** role group.

---

## C) Generate audited activities

Have **Allan Deyoung** perform the following:

- **Exchange (Outlook on the web)**: Send an email to **Adele Vance** with subject “Audit Lab”.
- **OneDrive/SharePoint**: Upload a file, rename it, and share it with **Adele Vance** (People in your org).
- **Teams**: Post a message in a team’s General channel or between **Adele Vance** and **Allan Deyoung**: “Hello, Audit!”  

---

## D) Search the audit log

1. Go to **Purview → Solutions → Audit → Search**.
2. Configure the query:
   - **Date and time (UTC)**: Last **24 hours** (or a range that covers your seeded actions).
   - **Users**: choose **Allan Deyoung** and **Adele Vance**.
   - **Activities - operation names**: pick likely activities (e.g., *Send*, *FileAccessed*, *FileModified*, *FileRenamed*, *FileDownloaded*, *MessageSent*, *MailItemsAccessed*). See <https://learn.microsoft.com/en-us/purview/audit-log-activities> for operation names.  
     *(You can also search by **Record types**, keywords, etc.*
3. Select **Search**. When the **Search job** finishes, open it to view **Search job details**.
4. Explore the results grid:
   - Useful columns: **Date (UTC)**, **User**, **IP Address**, **Workload/Record type**, **Activity**, **Item**, **Details**.
   - Sort by **Date (UTC)**; filter on **Workload** to focus on Exchange/SharePoint/Teams.
5. Select any row to open the **details flyout** and review the full JSON payload.

---

## E) (Premium) Create an Audit log retention policy

1. In **Purview → Solutions → Audit → Policies**, select **Create audit retention policy**.
2. **Policy name**: `Teams-1yr-All`.
3. **Users**: Specify **Allan Deyoung** and **Adele Vance**
4. **Record Type**: select **MicrosoftTeams** (choose others as desired).
5. **Duration**: **1 year**  
   - (If you have the 10-year add-on, you can choose **3/5/7/10 years**.)
6. **Priority**: use **1** (you can adjust later if you add multiple policies).  
7. Click **Save**

> Notes:  
> • A built-in Premium default policy may already keep certain workloads longer (e.g., 1 year). Custom policies can extend other record types or specific users.  
> • When multiple policies apply, precedence is determined by policy priority (lower number = higher precedence).

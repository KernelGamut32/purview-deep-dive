
# Lab 01: Microsoft Purview - Compliance Manager

## Lab Duration: 30-40 minutes

## Lab Objectives

By the end of this lab, you will:

- Understand what Compliance Manager is and how it can help an organization manage compliance against risk and regulation
- See Compliance Manager in action against a specific regulation (or set of regulations)
- Explore options for assigning specific findings to a technical resource for remediation

---

## Lab Instructions

### 1. Explore Compliance Manager Within the Microsoft Purview Portal

**Activity:**

1. Log in to the Microsoft Purview portal (<https://purview.microsoft.com>)
2. Under "Solutions", explore some of the various tools that Purview provides (we will be drilling down into some of these various tools throughout the course)
3. After exploring, select "Compliance Manager" under "Solutions"
4. Select "Assessments" in the menu on the left-hand side of the screen
5. Click "Add assessment"
6. Click "Select regulation" and search for/select a regulation (or set of regulations) to include in this assessment (e.g., ISO 27001)
7. Once all target regulations have been included, click "Save"
8. Click "Next"
9. Specify a name for your assessment (e.g., "Assessment Against 2022 Iteration of ISO 27001")
10. Under "Assessment group", leave "Use existing group" selected and ensure "Default Group" is selected in the provided dropdown; click "Next"
11. On the "Select services" page, "Microsoft 365" should already be selected - if not, click "Select services" and check "Microsoft 365"; click "Next"
12. On the "Review and finish" page, click "Create assessment"
13. After a few seconds, you should see a "New assessment created" message - click "Done" on that page
14. Review the "Details" and "About" sections for the assessment
15. Drill into "Progress", "Controls", "Your improvement actions", and "Microsoft actions" to review the specifics that inform the resulting compliance score for this assessment

---

### 2. Assign Specific Improvement Tasks

**Activity:**

1. In the previously created assessment results, click "Your improvement actions"
2. Set the "Control family" filter to include "People controls" and "Technological controls"; click "Apply"
3. Here you can select individual improvement actions (single or multiple) and assign to a user in your organization (using the "Assign to user" option)
4. Alternatively, you can click the checkbox in the table's heading to select all
5. When you click "Assign to user", you can search for and select a user in your organization's Entra tenant and click "Assign"
6. **NOTE:** This will send an email to that user so if you are operating in a live environment keep that in mind

---

### 3. Manage User Access for Remediation

**Activity:**

1. In the assessment details on the "Your improvement actions" tab, click "Manage user access" in the upper right-hand corner of the screen
2. Note the different types of permissions that can be assigned
3. Select the "Assessor" permission and click "Add assessors"
4. Select the same user account used in the previous section
5. Click "Apply"
6. Click "Save"

---

### 4. Optional: Review the Corresponding Email Message

**Activity:**

1. Access the email account for the user assigned in the previous sections
2. Review the email message received
3. Click "View action item(s)" in the email body
4. On the page listing the "Improvement actions", you can filter (e.g., by "Assigned To")
5. Select one of the improvement actions to review details including the "Evidence" and "Related controls" tabs
6. This is where the user can attest and document progress and test status against the action
7. You can also export improvement actions to an Excel spreadsheet, manage there, and then apply updates to multiple actions in batch
8. As those improvement actions are implemented (or remediated), the associated compliance score will adjust allowing objective tracking against the regulation (**NOTE:** It can take up to a day for some of the updates to reflect on the compliance score)

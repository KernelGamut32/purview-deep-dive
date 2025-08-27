# Lab 09 - Keyword Query Language (KeyQL)

- Login to <https://purview.microsoft.com>
- In the left-hand menu, select Solutions | eDiscovery
- If not taken there automatically, select Cases and click "Create case"
- Assign a name to your case (e.g., "Query Language Searches")
- Click "Create"
- After the case has been created, click "Cases" in the top-level breadcrumb to return to the "Cases" dashboard
- Review the dashboard including the various attributes associated to each case
- Notice the "Delete case" and "Close case" option for each case
- Click the case to drill down into case details
- Click "Case settings" to review (and adjust if required) the case's configuration
- Close the settings and navigate to the "Data sources" tab and add 2 - 3 individuals
- From the "Searches" tab, create a new search - assign a name (e.g., "Recordings Sent")
- Click "Add sources" to add specific sources to the search
- For "Scope items by", click "All sources in this case" and select the 2 or 3 users previously added to the case
- Alternatively, you can add additional sources by setting "Scope items by" to "All sources in the tenant (default)"; try adding a group
- Click "Manage" to define the data sources that will be searched as part of the case
- Add a KeyQL query to focus on MP4 (recordings) sent as attachments - use:

```KeyQL
((Kind:email) AND (HasAttachment=true) AND (AttachmentNames:.mp4)) OR ((Kind:microsoftteams) AND (HasAttachment=true) AND (AttachmentNames:.mp4))
```

- Test by sending an email and a Teams chat message with an attachment having the specified extension
- Also, send separate emails/chats with attachments that do not have the specified extension
- Run the query and update statistics and samples views to see what was uncovered with the latest configuration

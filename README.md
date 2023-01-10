---
title: #Required; page title displayed in search results. Include the word "tutorial". Include the brand.
description: #Required; article description that is displayed in search results. Include the word "tutorial".
author: ingebeumer
ms.author: invand
ms.service: #Required; service per approved list. service slug assigned to your service by ACOM.
ms.topic: tutorial
ms.date: 01/10/2023
---

# Azure DevOps for On-Demand Assessment remediation
## Introduction
Please note this guidance intentionally does not extensively go into concepts of Scrum and Agile methodologies nor how to apply the Scrum Framework with Boards of Azure DevOps. The guidance focuses on doing more with less, on gaining automation savings and keep developers and administrators focused on high-impact and not administrative work.

![image](https://user-images.githubusercontent.com/40343254/211434877-594ee4b6-1283-4b59-bce0-16fdb99765dd.png)

EMPOWER EVERY DEVELOPER AND EVERY ORGANIZATION TO ACHIEVE MORE WITH THEIR DATA.

## What, why, and how
**What**: Easier tracking of remediation progress of On-Demand Assessment conform the Agile working method in your organization.

**Why**: Track progress on what the team is working on.

**How**: This document gives guidance like in a QuickStart in Microsoft Learn to:

- Export the Focus Area and findings and recommendations from the Log Analytics workspace from your On-Demand Assessment and transform them to look like Epics and Product Backlog Items used in Azure DevOps.
- Bulk import the Epics and Product backlog items in Azure DevOps. 

WORK CAREFULLY BECAUSE YOUR AZURE DEVOPS PROJECT MAY HAVE CONTENT ALREADY.

## Prerequisites
### Prerequisites and Permissions
- Browser like [Microsoft Edge]((https://www.microsoft.com/en-us/edge?form=MA13FJ)) with Internet Access.
- Azure DevOps Organization. 
If there is no organization available to you now you can start with a free organization for [Azure DevOps Services]((https://azure.microsoft.com/en-us/products/devops/)). Going through the steps in [Sign up for Azure DevOps]((https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates?toc=%2Fazure%2Fdevops%2Fget-started%2Ftoc.json&view=azure-devops)) there is a Basic license where the first five users are free.
- Microsoft Office 2016 or higher (optional)
- Discuss with your Azure DevOps project owner so they are aware of this import.
- Log Analytics workspace permissions to export data. Validate permissions with principle of least privilege. Log Analytics Reader role.
- Azure DevOps permissions for creating items needed through import. Validate permissions with principle of least privilege.
 
### Azure DevOps project
If you do not have an existing Azure DevOps project in your organization, you can create one. Create a project in Azure DevOps and Choose a process for your Azure DevOps project that suits your needs.
 
### Azure DevOps project process
Depending on which process was chosen on creation of your Azure DevOps project Work Item Type for backlog levels is different.  
Agile	:	User Story, Feature, and Epic
Basic	:	Issue and Epic
Scrum	:	Product Backlog Item, Feature, and Epic
CMMI	:	Requirement, Feature, and Epic
- Go to your Azure DevOps project and confirm the Process in Project details. If the process is other than Scrum, you need to change the query parameter for Work Item Type later in the KQL query (Kusto Query Language).

Examples throughout this document are from an Azure DevOps Scrum project and an On-Demand Assessment for SQL. 
 

**Note:**
There may be possibilities for similar bulk imports, for example into Jira. The process was only tested on Azure DevOps. Adjust the export from Log Analytics workspace to match the import expected by Jira or other tools supporting Agile methodologies.
How to bulk import Components into JIRA Issues | Jira | Atlassian Documentation
Importing data from CSV | Administering Jira applications Data Center and Server 9.4 | Atlassian Documentation


## Export from Log Analytics workspace
We start by exporting On-Demand Assessment findings and recommendations from the Log Analytics workspace.
	In the Azure portal go to the specific Log Analytics workspaces from which you want to export.
	Go to Logs in the workspace.
 
	Close the Queries window.
 

### Assessment Name
On-Demand Assessment names differ per product. To export from the assessment you are interested in, we need the name. 
	Copy below KQL query (Kusto Query Language) to Log Analytics workspace > Logs.
	Select Run to get the assessment name. 
If there’s no immediate result, change the Time range; the default schedule to run On-Demand Assessments is weekly, but the default time range for KQL is the last 24 hours. 
This example results to SQLAssessmentRecommendation.
 	 
//Assessment
search *
| where $table contains "Assessment"
| summarize by $table
| project $table
 
Note
Rows are highlighted in the table below where the guidance is confirmed. The table is matching On-Demand Assessment Prerequisite Documents with Log Analytics workspace supported tables containing ‘Assessment’.

| **Assessment name** | **Assessment technical name** | 
|--------|---------|
| **Active Directory** | **ADAssessmentRecommendation** | 
| **Active Directory Security** | **ADSecurityAssessmentRecommendation** | 
| Microsoft Azure | AzureAssessmentRecommendation | 
| Exchange Server | ExchangeAssessmentRecommendation | 
| System Center Configuration Manager | 	SCCMAssessmentRecommendation | 
| System Center Operations Manager | SCOMAssessmentRecommendation | 
| Skype for Business | SfBAssessmentRecommendation | 
| Office 365 Skype and Teams  | SfBOnlineAssessmentRecommendation | 
| Office 365 SharePoint Online | SharePointOnlineAssessmentRecommendation | 
| SharePoint Server | SPAssessmentRecommendation | 
| **SQL Server** | **SQLAssessmentRecommendation** | 
| Windows Server (Server, Server Security, Hyper-V, Failover Cluster, IIS) | 
| Windows Client |  | 
| Office 365 Exchange Online |  | 

### Export Focus Area
Start by exporting the Focus Areas.
- Copy below KQL query to Log Analytics workspace > Logs. 
- Select Ctrl + H to search for MyAssessmentRecommendation and replace this with the result from Assessment Name.
- Adjust query parameter qpAssignTo and replace My Full Name by your full name as shown in Azure DevOps. 
 
- Adjust the Time range if you are on the default schedule to run On-Demand Assessments weekly.
- Select Run the query to get the Focus Area that will transform to Epics. 
- Export the results to csv through Export | Export to CSV - displayed columns.
- Rename query_data.csv to _1_Epic.csv_.
- If you reorder columns or change the sort order in the csv files, the later import in Azure DevOps may break or have unexpected results.

This example results from items found in the On-Demand Assessment for SQL.
 

//Focus Area
declare query_parameters (
    qpAssignTo: string = "My Full Name" //your full name as shown in Azure DevOps
);
//
MyAssessmentRecommendation //replace with result from Query Assessment Name
| where (RecommendationResult == "Failed")
| summarize by AssessmentName, FocusArea
| project 
    ID = ' '
    , ['Work Item Type'] = 'Epic'
    , ['Assigned To'] = qpAssignTo
    , ['Title 1'] = strcat('On-Demand Assessment for ', AssessmentName, ' | ', FocusArea)
    , ['Title 2'] = ''
    , Description = strcat('Epic imported from "On-Demand Assessment for ', AssessmentName, '" and Focus Area "', FocusArea
    , '" in Log Analytics workspace.')
    , Priority = 2;
//Export | Export to CSV - displayed columns.
//Rename query_data.csv to _1_Epic.csv_

### Export findings and recommendations
Like the export of Focus Area, we export findings and recommendations.
- Copy below KQL query to Log Analytics workspace > Logs. 
- Use Ctrl + H to search for MyAssessmentRecommendation and replace this with the result from Assessment Name.
- Adjust query parameter qpAssignTo and replace My Full Name by your full name as shown in Azure DevOps. 
- If needed adjust query parameter qpWorkItemType and replace Product Backlog Item by your Work Item Type as confirmed in Azure DevOps project process.
- Adjust the Time range if you are on the default schedule to run On-Demand Assessments weekly.
- Select Run the query to get the findings and recommendations that will transform to product backlog items.
- Export the results to csv through Export | Export to CSV - displayed columns.
- Rename query_data.csv to _2_ProductBacklogItems.csv_.
- If you reorder columns or change the sort order in the csv files, the later import in Azure DevOps may break or have unexpected results.

This example results from items found in the On-Demand Assessment for SQL. 
 
//Findings and recommendations
declare query_parameters (
    qpAssignTo: string = "My Full Name" //your full name as shown in Azure DevOps.
    , qpWorkItemType: string = "Product Backlog Item" //Work Item Type as in your Azure DevOps project process.
);
//
MyAssessmentRecommendation
| where (RecommendationResult == "Failed")
| where not (FocusArea has_any("InternalAssessmentQuality", "Prerequisite"))
| summarize by AssessmentName, FocusArea
| project 
    ID = '<EpicID>'
    , ['Work Item Type'] = 'Epic'
    , ['Assigned To'] = qpAssignTo
    , ['Title 1'] = strcat('On-Demand Assessment for ', AssessmentName, ' | ', FocusArea)
    , ['Title 2'] = ''
    , Description = strcat('Epic imported from "On-Demand Assessment for ', AssessmentName, '" and Focus Area "', FocusArea, '" in Log Analytics workspace.')
    , Priority = 2
    , tmpFocusArea = FocusArea
| union (
    MyAssessmentRecommendation
    | where (RecommendationResult == "Failed")
    | where not (FocusArea has_any("InternalAssessmentQuality", "Prerequisite"))
    | summarize by AssessmentName, FocusArea, Recommendation, Description, RecommendationScore
    | sort by FocusArea, Recommendation
    | project 
        ID = ' '
        , ['Work Item Type'] = qpWorkItemType
        , ['Assigned To'] = qpAssignTo
        , ['Title 1'] = ''
        , ['Title 2'] = Recommendation
        , Description
        , RecommendationScore
        , tmpFocusArea = FocusArea
    | extend Priority = //convert RecommendationScore in On-Demand Assessment to Priority in Azure DevOps
        iif(RecommendationScore >= 30, 1, iif(RecommendationScore >= 20, 2, iif(RecommendationScore >= 10, 3, iif(RecommendationScore >= 0, 4, 4))))
    | project-away RecommendationScore
    )
| sort by tmpFocusArea asc, ['Work Item Type'] asc 
| project-away tmpFocusArea;
//Export | Export to CSV - displayed columns.
//Rename query_data.csv to _2_ProductBacklogItem.csv_

## Import into Azure DevOps
Now the export is complete, we can import Azure DevOps Epics and Product Backlog Items.
### Import Epics
Go to your Azure DevOps project and select the Queries menu item on the left.
 
- In the top menu select Import Work Items.
 
- On the right select Choose File. Navigate to _1_Epic.csv_ and select Import.
 
- Review the imported items to see if there are no data issues highlighted that you need to resolve before you can Save items.
 
- Keep the screen open because you need the Epic IDs in a minute.

### Import Product Backlog Items
- If you don’t have the screen open with the Epics you just created, go to your Azure DevOps project and Recently created in Work items.
 
- First, you need to prepare file _2_ProductBacklogItems.csv_ for import in Azure DevOps.
- Open file _2_ProductBacklogItems.csv_.
- For each row in file _2_ProductBacklogItems.csv_ where Work Item Type equals Epic, replace the placeholder text <EpicID> by the ID from the Azure DevOps Epics we just imported and paste it in the row with the same Epic title in 2_ProductBacklogItems.csv.
 
- Save and close file _2_ProductBacklogItems.csv_.
- Return to Import Work Items in the **Queries** menu item in the left in Azure DevOps.
- Choose **Import Work Items**.
 
- On the right **Choose File**. Navigate to file _2_ProductBacklogItems.csv_ and select **Import**.
- Review the imported items to see if there are no data issues highlighted that you need to resolve before you can continue. Depending on the number of error messages either remediate manually in Azure DevOps or change in file _2_ProductBacklogItems.csv_ and restart the import.
Figure 1. Example with import errors where the name in the csv file does not match an identity in Azure DevOps to assign Work Items to.
 
- Save Items when all is ready for import.
- The import process has populated the backlog with all the findings and recommendations from your assessment. 
 
 
Next you can work on your On-Demand Assessment remediation in Azure DevOps as you would with any other Work Item in your Agile working method.

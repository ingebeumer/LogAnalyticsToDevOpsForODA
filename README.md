# Tutorial: Azure DevOps for On-Demand Assessment remediation

**Applies to:** :white_check_mark:ADAssessmentRecommendation :white_check_mark:ADSecurityAssessmentRecommendation :white_check_mark:SQLAssessmentRecommendation

**Subject to validation:** :small_orange_diamond:AzureAssessmentRecommendation :small_orange_diamond:ExchangeAssessmentRecommendation :small_orange_diamond:SCCMAssessmentRecommendation :small_orange_diamond:SCOMAssessmentRecommendation :small_orange_diamond:SfBAssessmentRecommendation :small_orange_diamond:SfBOnlineAssessmentRecommendation :small_orange_diamond:SharePointOnlineAssessmentRecommendation :small_orange_diamond:SPAssessmentRecommendation 

In this tutorial, you will export and transform Focus Area as well as findings and recommendations from Log Analytics workspace for On-Demand Assessment and import them as Epics and Product backlog items in Azure DevOps.

![Screenshot with basic agile user story: As a \<type of user\>, I want \<some goal\> so that \<some reason\>.](https://user-images.githubusercontent.com/40343254/211532236-1bda1e60-0961-4910-9b69-4cb190eb0867.png)

As an Administrator, I want to have the findings and recommendations of an On-Demand Assessment in Azure DevOps Work Items so that I can work with On-Demand Assessment remediation as with any other Work Item in my organization.

The result of this tutorial allows for easier tracking of remediation progress of On-Demand Assessment conform the Agile working method in your organization. Tracking progress on what the team is working on then on also conforms to your Agile working method.

The tutorial does not go further into concepts of Scrum and Agile methodologies nor how to apply the Scrum Framework with Boards in Azure DevOps. The focus is on doing more with less, on gaining automation savings and keeping developers and administrators focused on high-impact and not administrative work. To empower every developer and every organization to achieve more with their data. 

You will learn how to:

:white_check_mark:Export Focus Area and findings and recommendations from Log Analytics workspace for On-Demand Assessment.

:white_check_mark:Use relevant queries in Kusto Query Language to perform export and transformation.

:white_check_mark:Bulk import Epics and Product backlog items in Azure DevOps.

## Prerequisites
Prerequisites to complete this tutorial.
### Prerequisites and Permissions
- Browser like [Microsoft Edge](https://www.microsoft.com/en-us/edge?form=MA13FJ) with Internet Access.
- Azure DevOps Organization. 
If there is no organization available to you now you can start with a free organization for [Azure DevOps Services](https://azure.microsoft.com/en-us/products/devops/). Going through the steps in [Sign up for Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates?toc=%2Fazure%2Fdevops%2Fget-started%2Ftoc.json&view=azure-devops) there is a Basic license where the first five users are free.
- [Microsoft Office 2016](https://www.microsoft.com/en-us/microsoft-365/get-started-with-office-2021) or higher (optional)
- Discuss with your Azure DevOps project owner so they are aware of this import. Work carefully because your Azure DevOps project may have content already.
- Log Analytics Reader role on Log Analytics workspace to export data.
- Azure DevOps permissions for importing Work Items. `ToDo --> Validate permissions with principle of least privilege.`
 
### Azure DevOps project
If you do not have an existing Azure DevOps project in your organization, you can create one. [Create a project in Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops&tabs=browser) and [Choose a process for your Azure DevOps project](https://learn.microsoft.com/en-us/azure/devops/boards/work-items/guidance/choose-process?view=azure-devops&tabs=agile-process#choose-a-basic-agile-scrum-and-cmmi-process) that suits your needs.
 
### Azure DevOps project process
Depending on which process was chosen on creation of your Azure DevOps project [Work Item Type](https://learn.microsoft.com/en-us/azure/devops/boards/boards/kanban-quickstart?view=azure-devops#add-a-kanban-board) for backlog levels is different.  
- [Agile](https://learn.microsoft.com/en-us/azure/devops/boards/work-items/guidance/agile-process?view=azure-devops) : User Story, Feature, and Epic
- [Basic](https://learn.microsoft.com/en-us/azure/devops/boards/get-started/plan-track-work?view=azure-devops) : Issue and Epic
- [Scrum](https://learn.microsoft.com/en-us/azure/devops/boards/work-items/guidance/scrum-process?view=azure-devops) : Product Backlog Item, Feature, and Epic
- [CMMI](https://learn.microsoft.com/en-us/azure/devops/boards/work-items/guidance/cmmi-process?view=azure-devops) : Requirement, Feature, and Epic

> **Note**
> 
> Examples throughout this document are from an Azure DevOps Scrum project (and an On-Demand Assessment for SQL).
> Go to your [Azure DevOps project](https://dev.azure.com/) and confirm the Process in Project details. If the process is other than Scrum, you need to change the query parameter for Work Item Type later in the KQL query (Kusto Query Language).
> 
> ![Screenshot showing project Process type in Project details.](https://user-images.githubusercontent.com/40343254/211513340-b1529c40-6fcb-4601-8f5c-74af66b427c9.png)

> **Note**
> 
> This tutorial is using Azure DevOps. However, there may be possibilities for similar bulk imports, for example into Jira. Adjust the export from Log Analytics workspace to match the import expected by Jira or other tools supporting Agile methodologies.
> 
> [How to bulk import Components into JIRA Issues | Jira | Atlassian Documentation](https://confluence.atlassian.com/jirakb/how-to-bulk-import-components-into-jira-720418562.html)
>
> [Importing data from CSV | Administering Jira applications Data Center and Server 9.4 | Atlassian Documentation](https://confluence.atlassian.com/adminjiraserver/importing-data-from-csv-938847533.html)


## 1 - Export from Log Analytics workspace
We start by exporting On-Demand Assessment findings and recommendations from the Log Analytics workspace.

1.	In the Azure portal select the specific [Log Analytics workspace](https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.OperationalInsights%2Fworkspaces) from which you want to export.
1.	Select **Logs** in the workspace menu.

    ![Screenshot of partial menu in Log Analytics workspace to navigate to Logs.](https://user-images.githubusercontent.com/40343254/211514234-3eb0fa29-33ef-44be-95fc-de1bdb85ad5b.png)

1.	Close the Queries window.

    ![Screenshot of closing the Queries window.](https://user-images.githubusercontent.com/40343254/211514249-a22c2050-becb-4e65-9532-2fa97ed777bc.png)

### 1.1 - Assessment Name
On-Demand Assessment names differ per product. To export from the assessment you are interested in, you need the name. 
1.	Copy the KQL query (Kusto Query Language) to Log Analytics workspace > Logs.
	   ```kusto
    //Assessment
    search *
    | where $table contains "Assessment"
    | summarize by $table
    | project $table
    ```
    
1.	Select **Run** to get the assessment name. 
If there’s no immediate result, change the **Time range**; the default schedule to run On-Demand Assessments is weekly, but the default time range for KQL is the last 24 hours. 

This example results to SQLAssessmentRecommendation.

![Screenshot of KQL query with a 24 hours time range showing 1 result.](https://user-images.githubusercontent.com/40343254/211517173-16d3f0c0-36a6-4ad3-a1d8-3bf1a0eb9222.png)
![Screenshot of KQL query with a 7 days time range showing 3 results.](https://user-images.githubusercontent.com/40343254/211517206-980548ad-c707-4680-ba0e-88f3c8050ba6.png)
 
### 1.2 - Export Focus Area
Start by exporting the Focus Areas.
1. Copy below KQL query to Log Analytics workspace > Logs. 
1. Select <kbd>Ctrl</kbd>+<kbd>H</kbd> to search for `<MyAssessmentRecommendation>` and replace this with the result from the Assessment Name query.
1. Adjust query parameter **qpAssignTo** and replace `<MyFullName>` by your full name as shown in Azure DevOps. 
    ![Screenshot of KQL query and modifications needed before running the query.](https://user-images.githubusercontent.com/40343254/211526368-79e47bc9-ac49-468d-81e4-8efd5ff87013.png)
1. Adjust the **Time range** if you are on the default schedule to run On-Demand Assessments weekly.
    ```kusto
	   //Focus Area
	   declare query_parameters (
	        qpAssignTo: string = "<MyFullName>" //your full name as shown in Azure DevOps
	   );
	   //
	   <MyAssessmentRecommendation> //replace with result from Query Assessment Name
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
	   //**Export | Export to CSV - displayed columns**.
	   //Rename _query_data.csv_ to _1_Epic.csv_
    ```

1. Select **Run** the query to get the Focus Area that will transform to Epics. 
1. Export the results to csv through **Export | Export to CSV - displayed columns**.
1. Rename _query_data.csv_ to _1_Epic.csv_.
1. If you reorder columns or change the sort order in the csv files, the later import in Azure DevOps may break or have unexpected results.

This example results from items found in the On-Demand Assessment for SQL.
    ![Screenshot of running the KQL query for Focus Area and Exporting the results to CSV.](https://user-images.githubusercontent.com/40343254/211526439-91deac6d-9e32-4466-aaa5-4939ff85ac84.png)

### 1.3 - Export findings and recommendations
Like the export of Focus Area, we export findings and recommendations.
1. Copy below KQL query to Log Analytics workspace > Logs. 
1. Use <kbd>Ctrl</kbd>+<kbd>H</kbd> to search for `<MyAssessmentRecommendation>` and replace this with the result from the Assessment Name query.
1. Adjust query parameter **qpAssignTo** and replace `<MyFullName>` by your full name as shown in Azure DevOps. 
1. If needed adjust query parameter **qpWorkItemType** and replace **Product Backlog Item** by your Work Item Type as confirmed in Azure DevOps project process.
1. Adjust the **Time range** if you are on the default schedule to run On-Demand Assessments weekly.
    ```kusto
    //Findings and recommendations
    declare query_parameters (
        qpAssignTo: string = "<MyFullName>" //your full name as shown in Azure DevOps.
        , qpWorkItemType: string = "Product Backlog Item" //Work Item Type as in your Azure DevOps project process.
    );
    //
    <MyAssessmentRecommendation>
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
        <MyAssessmentRecommendation>
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
    //**Export | Export to CSV - displayed columns**.
    //Rename _query_data.csv_ to _2_ProductBacklogItem.csv_
    ```
1. Select **Run** the query to get the findings and recommendations that will transform to product backlog items.
1. Export the results to csv through **Export | Export to CSV - displayed columns**.
1. Rename _query_data.csv_ to _2_ProductBacklogItems.csv_.
1. If you reorder columns or change the sort order in the csv files, the later import in Azure DevOps may break or have unexpected results.

This example results from items found in the On-Demand Assessment for SQL.
![Screenshot of results of KQL query for Findings and Recommendations in Logs in Log Analytics workspace.](https://user-images.githubusercontent.com/40343254/211528711-ec383683-6ee3-42a1-8458-cb0e763130dc.png)
 
## 2 - Import into Azure DevOps
Now the export is complete, we can import Azure DevOps Epics and Product Backlog Items.
### 2.1 - Import Epics
1. Go to your [Azure DevOps](https://dev.azure.com/) project and select the **Queries** menu item on the left.

    ![Screenshot of Azure DevOps menu to navigate to Queries.](https://user-images.githubusercontent.com/40343254/211528944-f36255bd-8b1d-4973-abb7-b060633cd456.png)
 
1. In the top menu select **Import Work Items**.

    ![Screenshot of Azure DevOps Queries to Import Work Items.](https://user-images.githubusercontent.com/40343254/211529193-6b36e5e4-668f-4156-b6fd-7ef28a211a99.png)

1. On the right select **Choose File**. Navigate to _1_Epic.csv_ and select **Import**.

    ![Screenshot of choosing your csv file to import into Azure DevOps.](https://user-images.githubusercontent.com/40343254/211529308-1ad956f5-3c7c-4263-b63d-042ffc58a4dc.png)

1. Review the imported items to see if there are no data issues highlighted that you need to resolve before you can **Save items**.

    ![Screenshot of Azure DevOps Import Work Items to confirm there are no items highlighted before you can Save items.](https://user-images.githubusercontent.com/40343254/211529360-db545e6a-ad53-410a-af22-00fa5d29a2de.png)

1.  Keep the screen open because you need the Epic IDs in a minute.

### 2.2 - Import Product Backlog Items
1. If you don’t have the screen open with the Epics you just created, go to your [Azure DevOps](https://dev.azure.com/) project and **Recently created** in **Work items**.

    ![Screenshot of how to Save items in Azure DevOps that were just imported.](https://user-images.githubusercontent.com/40343254/211529610-9b467733-5300-4617-8f15-e50da37b52d7.png)

1. First, you need to prepare file _2_ProductBacklogItems.csv_ for import in Azure DevOps.
1. Open file _2_ProductBacklogItems.csv_.
1. For each row in file _2_ProductBacklogItems.csv_ where Work Item Type equals Epic, replace the placeholder text `<EpicID>` by the ID from the Azure DevOps Epics we just imported and paste it in the row with the same Epic title in _2_ProductBacklogItems.csv_.

    ![Screenshot of csv file in Excel with highlighted rows of Work Item Type equals Epic. Overlay of Azure DevOps imported and saved Epics with the ID column highlighted to copy the ID to the matching row in the csv file.](https://user-images.githubusercontent.com/40343254/211529781-07d067d5-42f0-4e6d-ba19-1ab002850239.png)

1. Save and close file _2_ProductBacklogItems.csv_.
1. Return to **Import Work Items** in the **Queries** menu item in the left in Azure DevOps.
1. Select **Import Work Items**.

    ![Screenshot of how to Import Work Items in Queries in Azure DevOps.](https://user-images.githubusercontent.com/40343254/211529900-3e2be81d-3179-4a6a-915a-b5f8f90feac1.png)

1. On the right **Choose File**. Navigate to file _2_ProductBacklogItems.csv_ and select **Import**.
1. Review the imported items to see if there are no data issues highlighted that you need to resolve before you can continue. Depending on the number of error messages either remediate manually in Azure DevOps or change in file _2_ProductBacklogItems.csv_ and restart the import.

> **Warning**
> 
> Example with import errors where the name in the csv file does not match an identity in Azure DevOps to assign Work Items to. For a successful import, you need to resolve errors and warnings.
> 
> ![Screenshot of Azure DevOps Import Work Items with items with errors that need to be resolved before the items can be saved.](https://user-images.githubusercontent.com/40343254/211530058-801e2183-5175-48d1-94f1-6004aed0d4b8.png)

1. Select **Save Items** when all is ready for import.
1. The import process has populated the backlog with all the findings and recommendations from your assessment. 

    ![Screenshot of Azure DevOps Backlog populated with Product Backlog Items from findings and recommendations from the On-Demand Assessment. Parent Epic relates to Focus Area from the assessment.](https://user-images.githubusercontent.com/40343254/211530143-6c1536ce-3550-4669-bdfc-ddef9b60bc51.png)

###Clean up resources
When no longer needed, delete the csv files generated earlier.

### Next Steps
Next you can work on On-Demand Assessment remediation in Azure DevOps as you would with any other Work Item in your Agile working method.

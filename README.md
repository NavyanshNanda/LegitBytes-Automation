\# How Every List is Connected to its Flow -



\## SOD/EOD Collection in SharePoint from Planner -



There are 8 flows responsible for extracting and managing SOD/EOD tasks from Planner and then adding them to SharePoint. All 8 of these flows are in Power Automate. Some details about all of each of these -



\- \*\*SOD Tasks Extraction Flow (Dayshift and Nightshift):\*\* There are 2 flows responsible for SOD tasks extraction, one for Dayshift and another for Nightshift.  

&#x20;  <br/>Dayshift extraction flow runs at - 12:30PM From Monday to Saturday.  

&#x20;  Nightshift extraction flow runs at - 07:00PM From Tuesday to Saturday.  

&#x20;  <br/>Any tasks placed in the SOD bucket before the extraction time are considered the users SOD tasks for that day.



\- \*\*EOD Tasks Extraction Flow (Dayshift and Nightshift):\*\* There are 2 flows responsible for EOD tasks extraction, one for Dayshift and another for Nightshift.  

&#x20;  <br/>Dayshift extraction flow runs at - 11:00PM From Monday to Saturday.  

&#x20;  Nightshift extraction flow runs at - 03:00AM From Tuesday to Saturday.  

&#x20;  <br/>Any tasks placed in the EOD bucket or Marked as completed in any bucket before the extraction time are considered the users EOD tasks for that day.  

&#x20;  <br/>Once the EOD task details have been extracted from the planner, they are moved to the Past EOD's bucket for 1 week, for users to review.



\- \*\*Blocked Tasks Extraction Flow (Dayshift and Nightshift):\*\* There are 2 flows responsible for Blocked tasks extraction, one for Dayshift and another for Nightshift.  

&#x20;  <br/>Dayshift extraction flow runs at - 11:00PM From Monday to Saturday.  

&#x20;  Nightshift extraction flow runs at - 03:00PM From Tuesday to Saturday.  

&#x20;  <br/>Any tasks placed in the Blocked bucket before the extraction time are considered the users Blocked tasks for that day.



\- \*\*Past EOD's Cleanup (Dayshift and Nightshift):\*\* There are 2 flows responsible for cleaning up the Past EOD's Bucket in the planner.  

&#x20;  As all the EOD tasks don't get deleted instantaneously, they are placed in the Past EOD's bucket for a week for review.  

&#x20;  Once the week is up they are deleted completed removing them from the planner and only having their details saved in SharePoint.  

&#x20;  <br/>The Flow runs every Monday at the same time for both Dayshift and Nightshift.



\## SOD/EOD Lists in SharePoint -



In SharePoint, look for Team Daily Tracker Site. Inside of it there are 2 different lists, called SOD/EOD list and All SOD/EOD.  

<br/>Link to the SOD/EOD list - \[LINK](https://legitbytes.sharepoint.com/sites/DevTeamDailyTracker/Lists/SODEOD/AllItems.aspx)



Link to the All SOD/EOD list - \[LINK](https://legitbytes.sharepoint.com/sites/DevTeamDailyTracker/Lists/AllSODEOD/AllItems.aspx)



The difference between them is that the SOD/EOD list is used for multiple other flows because of which it only has 45 Days of data, whereas the All SOD/EOD list contains all the data since the data collection had begun, without the restrictions of number of days it contains all the data.



\## SOD/EOD Summary Flows -



Every time data is extracted from planner to store into SharePoint, after half an hour of that, a \*\*n8n\*\* flow executes and creates a summary of the data that was just collected and then posts that summary on a Microsoft Teams channels. There are 4 flows responsible for posting these summaries on Microsoft Teams channels.



Also, all these flows require Microsoft graph API permissions for them to work. I have added these permissions to a new registered app on Azure Portal.



\- \*\*SOD Summary Flow (Dayshift and Nightshift):\*\* Both flows run on identical n8n flow structure, the only differences are the trigger time and the change of one word in the entire code, i.e., Dayshift and Nightshift in the 4<sup>th</sup> line of the code.  

&#x20;  <br/>Dayshift SOD Summary Flow - Runs at 01:00PM  

&#x20;  Nightshift SOD Summary Flow - Runs at 07:30PM  

&#x20;  <br/>They use the data already in SharePoint and then using JavaScript code, the flow creates summary messages in the form of adaptive cards, Best format for posting messages on teams, and then finally posts the message on the teams channel of dayshift and nightshift separately.



\- \*\*EOD Summary Flow (Dayshift and Nightshift):\*\* Both flows also run on identical n8n flow structure, the only differences are the trigger time and the change of one word in the entire code, i.e., Dayshift and Nightshift in the 4<sup>th</sup> line of the code  

&#x20;  <br/>Dayshift EOD Summary Flow - Runs at 11:30PM  

&#x20;  Nightshift EOD Summary Flow - Runs at 03:30AM



\## Lists Sync Flow -



The main list called SOD/EOD list is being used for a lot of n8n flows. Because of a limitation in n8n, i.e., it is only able to fetch 5000 rows of a SharePoint list in one go. So because of this limitation SOD/EOD list needs to no more than 5000 rows. And we have done this by creating a 45-day limit in the list, any task which is prior to 45 days from today will get deleted.



But to preserve all the past SOD/EOD tasks, I have created a new list called 'All SOD/EOD'. A very small 2 block power automate flow works on maintaining this new list.



\- \*\*List Sync Flow:\*\* When a new row is added in the 45 days only SOD/EOD list, this flow triggers and creates the same row in the All SOD/EOD list.

\- \*\*List Cleanup Flow:\*\* In order for the 45-day limit to work we needed to create a flow that would read the snapshot dates of all the tasks and if that task is older than 45 days, it will get deleted automatically.  

&#x20;  This Flow runs every day at 09:00AM.



\#



IMPORTANT -



Similarly to the above-mentioned flows, a new set of flows has also been created for HR.



As the tasks for HR need to be independent of the rest of the company, their tasks will be added a new planner in a separate team, called LegitBytes, on the channel called HR.



The timings of these channels/planner's flows are the same as that of the Dayshift.



\## Story Point/Utilization flows -



While adding tasks, users are required to add labels to each of their tasks. These labels include their team/project name and Story Points.



These Story Points represent the amount of time a task might require.



How it works -



1 Story Point is for a 2-hour task.



2 Story Points are for a 4-hour task.



3 Story Points are for a 1-day task.



5 Story Points are for a 3-day task.



So, for a new task that I am about to create, for example, RND on new software for cold emailing, I would add 3 Story Points label, as I think it would take me an entire day to research on this software.



Now comes the use and the flows with these story points -



As these story points are being stored in the SharePoint list 'SOD/EOD list' I can use the story points column in this list to create flows related to utilization.



\- \*\*Task Utilization Flow:\*\* On the first of every month this flow extracts all the EOD tasks from the 'SOD/EOD list' and calculates the total Story Points of each employee. These total Story Points of each employee are then converted to number of hours they have spent, based on the above-mentioned logic.  

&#x20;  <br/>This flow runs at 09:00AM on the first of every month.



\- \*\*Monthly Utilization Flow:\*\* Right after the previous flow is completed, this flow runs and collects all the data gathered by the previous flow in the list 'Task Utilization list' and then calculated all the necessary metrics for the 'Monthly Utilization list' of each employee, which majorly has Utilization % (This metric helps managers understand the amount of time a employee is being utilized).  

&#x20;  <br/>This flow runs at 11:00AM on the first of every month.  

&#x20;  Link to the Monthly Utilization List - \[LINK](https://legitbytes.sharepoint.com/sites/DevTeamDailyTracker/Lists/Monthly%20Utilization%20List/AllItems.aspx)



\## Attendance Summary Flow -



Legitbytes uses greythr for tracking leaves and times of arrival/leave from the office. As we don't have the API of greythr, we had to extract the "\*\*Monthly Attendance Muster Report\*\*" from greythr's reports generation section manually.



HR extracts the report and adds that to a folder called 'Attendance Report Monthly' this folder can be found in the HR team in SharePoint, in the following path or link -



Path to the folder - Documents/HR/Attendance Reports Monthly  

Link to the folder - \[LINK](https://legitbytes.sharepoint.com/sites/HR142/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2FHR142%2FShared%20Documents%2FHR%2FAttendance%20Reports%20Monthly\&viewid=ea1fae62%2D9024%2D48af%2D998a%2Dd44fad5bc890\&newTargetListUrl=%2Fsites%2FHR142%2FShared%20Documents\&viewpath=%2Fsites%2FHR142%2FShared%20Documents%2FForms%2FAllItems%2Easpx)



Runs for the entire first week of every month at 09:00PM, runs every day so that the HR has an entire week to upload the Attendance Muster Report in SharePoint.



There is a n8n flow called 'MonthlyAttendanceListSummary' it runs every month and extracts data from the "\*\*Monthly Attendance Muster Report\*\*" placed in the folder 'Attendance Report Monthly', and the provides summarized data of each employee in a SharePoint list, the list tells number of times the employee was late, leaves, WFH and Absences of each employee per month.



Link to the list - \[LINK](https://legitbytes.sharepoint.com/sites/DevTeamDailyTracker/Lists/MonthlyAttendanceSummary/AllItems.aspx)



\## Compliance Summary Flow -



Every single day there are more than 50 tasks being uploaded to SharePoint, including all SOD, EOD and Blocked tasks. The issue was that there was too much data to estimate any employee's performance.



To deal with this situation, we needed a list that can summarize all the employee compliance performance.



There are 3 flows and 3 lists that deal with this situation -



\- \*\*Daily Compliance List Update Flow (Dayshift \& Nightshift):\*\* This flow runs one hour after every EOD task extraction flow. All this flow does is summarize the performance of each employee that day, i.e., if the employee has posted a SOD or EOD that day, or if the employee has any delayed tasks(Also the number of delayed tasks) that day, and finally adds this data to a list in SharePoint called 'Employee Compliance'.  

&#x20;  Link to the list - \[LINK](https://legitbytes.sharepoint.com/sites/DevTeamDailyTracker/Lists/Employee%20Compliance/AllItems.aspx)  

&#x20;  <br/>Dayshift flow runs at - 08:00PM  

&#x20;  Nightshift flow runs at - 04:00AM

\- \*\*Monthly Compliance List Flow:\*\* This flow uses all the data stored in the 'Employee Compliance' list and all the data in the 'Monthly Attendance Muster Report' the same report that the monthly attendance summary flow uses.  

&#x20;  <br/>The reason for using monthly attendance muster report with employee compliance is that the Employee Compliance list does not tell us about the leaves of employees which we extract from the Attendance Muster Report.  

&#x20;  <br/>Runs for the entire first week of every month at 06:00PM, runs every day so that the HR has an entire week to upload the Attendance Muster Report in SharePoint.\*\*  

&#x20;  <br/>\*\*Link to the Monthly Compliance List - \[LINK](https://legitbytes.sharepoint.com/sites/DevTeamDailyTracker/Lists/Monthly%20Compliance%20List/AllItems.aspx)


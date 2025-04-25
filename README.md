# Splunk - Windows Event Dashboard


## Forewords and Feedback
This write-up will reflect my thought process during the investigation and reasoning for actions taken during my **Home Lab - Splunk - Windows Event Dashboard**

Feel free to drop any feedback or point out anything I might’ve missed! I’m all about learning from others (and my own slip-ups). I really appreciate you taking the time to check out my project!

## Objective
We have been set the task to create a dashboard in Splunk using **Windows Events** telemetry which includes the following data:

-   Successful logins by user excluding System accounts
-   Successful logins with administrative privileges
-   Number of Failed Attempts by User

## Pre-requests
-   VM - _(WMware Workstation)_
-   Ubuntu Server - _(24.04.1 LTS)_
-   Splunk Enterprise _(9.3.1)_
-   Index file to use in Splunk _(5931 Events)_
-   Internet Browser

## Making sure we are using the correct index file for the lab
We are using the command **”index="mydfir-lab1”** to check we have **5931 events** as this will confirm we are working with the correct index file.

**INSERT IMAGE 1**

## Building our First query - Successful logins excluding System Accounts
For the dashboard for **Successful logins excluding System Accounts** and we are only interested in **Windows** event logs. We do this by selecting the Field name **host** and from here we select **winvm**.

**INSERT IMAGE 2**

Our query looks like this: **index="mydfir-lab1" host=winvm** which returns **993 Events**.

**INSERT IMAGE 3**

Next we want to get the Windows ID code for **Successful Windows logins** and to get this we will use the following query in Google: **windows successful logon event code** where we get the ID is **4624**.

**INSERT IMAGE 4**

With this info we go back to our Splunk query and click on the first event to look for the **field name** being used for the **Windows Event ID**.

**INSERT IMAGE 5**

**INSERT IMAGE 6**

We see the field name **EventCode 4672** and this is the field we will use to build further on our query. However we want to verify and confirm that we are working with the correct **Field name**. We do this with this Google search: **windows event id 4672** and get the code is for **Special privileges assigned to new logon**.

**INSERT IMAGE 7**

With the above info our query for the Event Code 4624 looks like this: **index="mydfir-lab1" host=winvm EventCode=4624** and now have **278 events**.

**INSERT IMAGE 8**

Next we want to look for a **field name** which can help us to identify the account name, and by this rule out **System Accounts**. We go back to our events and click on the **first event**.

**INSERT IMAGE 9**

We look through the field names and further down the list we see the field name **user** with the value **ADDC01$**. Why the field name user is interesting is because here we have the account name and when it ends with the $ sign it is an account with privileges. We click on the user to add it to our selected fields.

**INSERT IMAGE 10**

Now we can add to our query the user field and rule out any system accounts, our query looks like this: **index="mydfir-lab1" host=winvm EventCode=4624 user!=*$**

**INSERT IMAGE 10**

Now we have excluded any system accounts we want to see where the account logged into and how. We do this by taking note of the computer which is under the field **ComputerName**, at the same time we also take note of the **Logon_Type** field.

**INSERT IMAGE 11**

With the above information in mind we modify our query to include the field names **ComputerName, user** and **Logon_Type**. Our query now looks like this: **index=mydfir-lab1 host=winvm EventCode=4624 user!=*$ |stats count by ComputerName,user,Logon_Type**

**INSERT IMAGE 12**

From the result above we still see that a **SYSTEM** user still appears in our search. Though **keep in mind** that **attackers often abuse** this account and in production environments, you would want to baseline your **SYSTEM** account activities to filter out legitimate activity. However in our example we will **filter out** the **SYSTEM** account by adding another user field and entering SYSTEM as the value. Our query now looks like this: **index=mydfir-lab1 host=winvm EventCode=4624 user!=*$ user!=SYSTEM |stats count by ComputerName,user,Logon_Type**

**INSERT IMAGE 13**

We have now created our query for **Successful logins by user excluding System accounts** and move on to create our **Dashboard** with the following title and info.

**INSERT IMAGE 14**

**INSERT IMAGE 15**

# Building our Second query - Successful logins with administrative privileges
For our second query **Successful logins with administrative privileges**. We use the **Event ID 4672** which we checked and verified with the help of Google. Adapting and making changes to our previous query our new one looks like this: **index=mydfir-lab1 host=winvm EventCode=4672 user!=*$ user!=SYSTEM |stats count by ComputerName,user,Logon_Type**

**INSERT IMAGE 16**

**INSERT IMAGE 17**

**Worth noting** from the above results are that the Events tab shows 22 events while the Statistics has 0 events. The reason is that we are using **field names** which **exists**, **ComputerName** and **user**, though we are also using **Logon_Type** which does not exists. This is the reason why we have 0 in Statistics, because it is trying to look for **Logon_Type**. We can change this by changing our query to look like this: **index=mydfir-lab1 host=winvm EventCode=4672 user!=*$ user!=SYSTEM | table ComputerName,user,Logon_Type** and use **table** instead of **stats** command.

**INSERT IMAGE 18**

Again **worth noting**, we are getting events, and why? This is because table does not care if a field name exists or not. **Having this in mind when to use table or stats is important.** A scenario we will use table is when we are not certain if a field exists or not, and we will use stats to stack the events once I know what field names do exist. Since our second query is only looking for successful logins with administrative privileges, we can reuse our stats command and keep only the user field.

Our query now looks like this: **index=mydfir-lab1 host=winvm EventCode=4672 user!=*$ user!=SYSTEM |stats count by user**

**INSERT IMAGE 19**

From the search we see the only user is kporter. We save this to our existing dashboard Windows Activity and name our panel **Users with Administrative Privileges**.

**INSERT IMAGE 20**
**INSERT IMAGE 21**

# Building our Third query - New user accounts created
Our third query **New user accounts created** we use Google again and see the Event ID is 4720, and we try to adapt our query from 4672 to 4720. Our new query is: **index=mydfir-lab1 host=winvm EventCode=4720 user!=*$ user!=SYSTEM |stats count by user**

**INSERT IMAGE 22**

**INSERT IMAGE 23**

With our query we get **one event** and the user named **evil.** However, when we use the stats command, we will not know who created the account, **when** it happened, or **where** it occurred.

We head over to the **Events tab** and look at the first event. We see that this occurred on 2024-01-07 13:13:35 UTC and the Computer this action was performed on was **Desktop-IT**. Under the **Subject field**, we see a user called **kporter** who was responsible for creating a new account called **evil**.

**INSERT IMAGE 24**

From the available field names, we can use **ComputerName**. We do not want to use **Account_Name**, because it includes both subject & target account, which is not as useful when we want to perform more narrow searches in the future.

**INSERT IMAGE 25**

There is a field called **New_Account_Account_Name** that we can use for **evil** and **Subject_Account_Name for kporter**.

**INSERT IMAGE 26**

We have additional fields at the bottom such as user and **src_user**; these fields may not exist in your production environment because these were created by installing a 3rd party application called **Splunk Add-On for Microsoft Windows**. For more information about 3rd party application use this [link](https://splunkbase.splunk.com/) for **Splunk’s application portal**.

**INSERT IMAGE 27**

Use the stats command and include the field names **_time**, **ComputerName**, **Subject_Account_Name**, and **New_Account_Account_Name**.

Our query now look like this: **index=mydfir-lab1 host=winvm EventCode=4720 user!=*$ user!=SYSTEM |stats count by _time, ComputerName, Subject_Account_Name, New_Account_Account_Name**

**INSERT IMAGE 28**

We this query we get more information than we saw earlier which will help with analysis if a new user account was created. We add this onto our existing **Windows Activity dashboard** and title the panel as **New User Account Created**.

**INSERT IMAGE 29**

**INSERT IMAGE 30**

We now have our final dashboard with the data we were set out to collect and monitor, and now we can make changes to it for better readability if needed.

**INSERT IMAGE 31**


## Changelog

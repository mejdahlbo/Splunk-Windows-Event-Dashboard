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

![1 - Correct Index](https://github.com/user-attachments/assets/83c7cad6-a232-439f-ab3d-7074ebe08d77)

## Building our First query - Successful logins excluding System Accounts
For the dashboard for **Successful logins excluding System Accounts** and we are only interested in **Windows** event logs. We do this by selecting the Field name **host** and from here we select **winvm**.

![1 - host](https://github.com/user-attachments/assets/51bbfdf5-913c-4c28-b323-5137bd75c793)

Our query looks like this: **index="mydfir-lab1" host=winvm** which returns **993 Events**.

![2- winvm](https://github.com/user-attachments/assets/adb98427-fa29-43f7-9a77-f61f5edc2d4b)


Next we want to get the Windows ID code for **Successful Windows logins** and to get this we will use the following query in Google: **windows successful logon event code** where we get the ID is **4624**.

![3- Google](https://github.com/user-attachments/assets/920ecc47-f8ac-45bd-bc74-3c1ef307b7d5)

With this info we go back to our Splunk query and click on the first event to look for the **field name** being used for the **Windows Event ID**.

![4 - First event](https://github.com/user-attachments/assets/e22411b6-1b54-4f27-b007-dc52888f33a0)

![5 - Event Code](https://github.com/user-attachments/assets/ea8356c9-2372-4fb1-a03e-ba61d51fe04e)

We see the field name **EventCode 4672** and this is the field we will use to build further on our query. However we want to verify and confirm that we are working with the correct **Field name**. We do this with this Google search: **windows event id 4672** and get the code is for **Special privileges assigned to new logon**.

![6 - 4672](https://github.com/user-attachments/assets/86cc4e4c-f330-46b5-bf4a-8c1bed34de63)

With the above info our query for the Event Code 4624 looks like this: **index="mydfir-lab1" host=winvm EventCode=4624** and now have **278 events**.

![7 - 4624](https://github.com/user-attachments/assets/1afa6148-16f6-4e7a-a302-cb4651f65cbb)

Next we want to look for a **field name** which can help us to identify the account name, and by this rule out **System Accounts**. We go back to our events and click on the **first event**.

![8 - First Event](https://github.com/user-attachments/assets/5f825d54-31ad-417d-a428-56257eeb6338)

We look through the field names and further down the list we see the field name **user** with the value **ADDC01$**. Why the field name user is interesting is because here we have the account name and when it ends with the $ sign it is an account with privileges. We click on the user to add it to our selected fields.

![9 - User](https://github.com/user-attachments/assets/b6442039-a42b-4286-9de5-1368af29a436)

Now we can add to our query the user field and rule out any system accounts, our query looks like this: **index="mydfir-lab1" host=winvm EventCode=4624 user!=*$**

![10 - no system user](https://github.com/user-attachments/assets/9d2c4aee-5fa8-47c7-bf83-d77859ee7d8e)

Now we have excluded any system accounts we want to see where the account logged into and how. We do this by taking note of the computer which is under the field **ComputerName**, at the same time we also take note of the **Logon_Type** field.

![11 - Where logged in](https://github.com/user-attachments/assets/b4198c55-c9e0-40a5-854c-91c672b89b79)

With the above information in mind we modify our query to include the field names **ComputerName, user** and **Logon_Type**. Our query now looks like this: **index=mydfir-lab1 host=winvm EventCode=4624 user!=*$ |stats count by ComputerName,user,Logon_Type**

![12 - Computer Name user and Logon Type](https://github.com/user-attachments/assets/d83a2a2c-cb94-4d0d-89d6-9b7a6b21ae4b)

From the result above we still see that a **SYSTEM** user still appears in our search. Though **keep in mind** that **attackers often abuse** this account and in production environments, you would want to baseline your **SYSTEM** account activities to filter out legitimate activity. However in our example we will **filter out** the **SYSTEM** account by adding another user field and entering SYSTEM as the value. Our query now looks like this: **index=mydfir-lab1 host=winvm EventCode=4624 user!=*$ user!=SYSTEM |stats count by ComputerName,user,Logon_Type**

![13 - Filter](https://github.com/user-attachments/assets/f67546b0-adc4-44ae-8d9e-fe662acfadb9)

We have now created our query for **Successful logins by user excluding System accounts** and move on to create our **Dashboard** with the following title and info.

![14 - New Dashboard](https://github.com/user-attachments/assets/5f8d8d2f-5d11-45c5-9da8-49481d281274)

![15 - New Dashboard](https://github.com/user-attachments/assets/463f6c4d-d9fb-4c0e-8144-e61779df836e)

# Building our Second query - Successful logins with administrative privileges
For our second query **Successful logins with administrative privileges**. We use the **Event ID 4672** which we checked and verified with the help of Google. Adapting and making changes to our previous query our new one looks like this: **index=mydfir-lab1 host=winvm EventCode=4672 user!=*$ user!=SYSTEM |stats count by ComputerName,user,Logon_Type**

![16 - 4672](https://github.com/user-attachments/assets/8608e0aa-14e6-4d28-9632-570054ef9510)

![17 - Query](https://github.com/user-attachments/assets/44f253ae-ff03-4add-b86d-62210b3762ef)

**Worth noting** from the above results are that the Events tab shows 22 events while the Statistics has 0 events. The reason is that we are using **field names** which **exists**, **ComputerName** and **user**, though we are also using **Logon_Type** which does not exists. This is the reason why we have 0 in Statistics, because it is trying to look for **Logon_Type**. We can change this by changing our query to look like this: **index=mydfir-lab1 host=winvm EventCode=4672 user!=*$ user!=SYSTEM | table ComputerName,user,Logon_Type** and use **table** instead of **stats** command.

![18 - Stats](https://github.com/user-attachments/assets/e8cec544-10e5-4307-9a3b-843c2b6db4b6)

Again **worth noting**, we are getting events, and why? This is because table does not care if a field name exists or not. **Having this in mind when to use table or stats is important.** A scenario we will use table is when we are not certain if a field exists or not, and we will use stats to stack the events once I know what field names do exist. Since our second query is only looking for successful logins with administrative privileges, we can reuse our stats command and keep only the user field.

Our query now looks like this: **index=mydfir-lab1 host=winvm EventCode=4672 user!=*$ user!=SYSTEM |stats count by user**

![19 - User](https://github.com/user-attachments/assets/6db35873-b552-43ad-af91-366c1210e3a0)

From the search we see the only user is kporter. We save this to our existing dashboard Windows Activity and name our panel **Users with Administrative Privileges**.

![20 - Dashboard](https://github.com/user-attachments/assets/b986461b-1a74-45f2-8593-c8ed8e1df615)

![21 - Users with Administrative Privileges](https://github.com/user-attachments/assets/ebef9779-3ce8-4b53-ad86-e08746f9c90e)

# Building our Third query - New user accounts created
Our third query **New user accounts created** we use Google again and see the Event ID is 4720, and we try to adapt our query from 4672 to 4720. Our new query is: **index=mydfir-lab1 host=winvm EventCode=4720 user!=*$ user!=SYSTEM |stats count by user**

![22 - Event id 4720](https://github.com/user-attachments/assets/8e8d6aa4-0d61-4bb4-8dc4-ef0eefb3ceb3)

![23 - Evil](https://github.com/user-attachments/assets/46e32ea6-6578-42cc-9eec-919c5aee5b2a)

With our query we get **one event** and the user named **evil.** However, when we use the stats command, we will not know who created the account, **when** it happened, or **where** it occurred.

We head over to the **Events tab** and look at the first event. We see that this occurred on 2024-01-07 13:13:35 UTC and the Computer this action was performed on was **Desktop-IT**. Under the **Subject field**, we see a user called **kporter** who was responsible for creating a new account called **evil**.

![24 - User Evil](https://github.com/user-attachments/assets/fac68664-284b-4b69-af72-bf9b5b96fc70)

From the available field names, we can use **ComputerName**. We do not want to use **Account_Name**, because it includes both subject & target account, which is not as useful when we want to perform more narrow searches in the future.

![25 - Account_Name](https://github.com/user-attachments/assets/11c15e12-53d7-4cf3-b04e-de0e7a7c82ee)

There is a field called **New_Account_Account_Name** that we can use for **evil** and **Subject_Account_Name for kporter**.

![26 - Subject](https://github.com/user-attachments/assets/f531b545-a2f4-4024-8bcc-3d3af499166d)

We have additional fields at the bottom such as user and **src_user**; these fields may not exist in your production environment because these were created by installing a 3rd party application called **Splunk Add-On for Microsoft Windows**. For more information about 3rd party application use this [link](https://splunkbase.splunk.com/) for **Splunk’s application portal**.

![27 - src](https://github.com/user-attachments/assets/f4db788c-cc18-4560-b54c-a0336f7846ab)

Use the stats command and include the field names **_time**, **ComputerName**, **Subject_Account_Name**, and **New_Account_Account_Name**.

Our query now look like this: **index=mydfir-lab1 host=winvm EventCode=4720 user!=*$ user!=SYSTEM |stats count by _time, ComputerName, Subject_Account_Name, New_Account_Account_Name**

![28 - Evil](https://github.com/user-attachments/assets/b37298f2-2290-456c-8ae0-9448f24fdead)

We this query we get more information than we saw earlier which will help with analysis if a new user account was created. We add this onto our existing **Windows Activity dashboard** and title the panel as **New User Account Created**.

![29 - Existing](https://github.com/user-attachments/assets/37f9aaf2-ad51-4e03-b1b5-94d5d7957b66)

![30 - Title](https://github.com/user-attachments/assets/5318ec6d-fb01-4e82-8de8-12152896ca3f)

We now have our final dashboard with the data we were set out to collect and monitor, and now we can make changes to it for better readability if needed.

![31 - Dashboard](https://github.com/user-attachments/assets/835c2a24-1c62-4c36-baf1-619890b02ce4)

## Changelog

---
title: "Splunk 101"
date: 2022-03-28T14:03:06+02:00
draft: false
---

This write up refers to the [Splunk 101](https://tryhackme.com/room/splunk101) room on [TryHackMe](https://tryhackme.com/).

In this room we are familiarizing ourselves with [Splunk](https://www.splunk.com/), one of the best known and widely used SIEM (Security Information and Event Management) tool out there.

#### Task 1: Introduction to Splunk

##### Question 1)

Start the machine attached to this task and connect to it. Then open `http://127.0.0.1:8000` to connect to the Splunk instance. Click the `Completed` button to finish this task.

#### Task 2: Navigating Splunk

When accessing Splunk you are greeted by the default home screen. The top bar holds information about system-level messages, settings for configuring Splunk, job activity and miscellaneous information such as tutorial and a search feature.

On the left you can see the Apps panel. Here you can see all apps installed for the Splunk instance.

In the upper middle you will find the Explore Splunk panel with quick links for adding data to the Splunk instance, adding new apps or accessing Splunk's documentation.

Last but not least the lower middle represents the Splunk Dashboard. You can choose from a range of dashboards readily available or create and add them yourself.

To learn more about how to navigate Splunk review its [documentation](https://docs.splunk.com/Documentation/Splunk/8.1.2/SearchTutorial/NavigatingSplunk).

##### Question 2)

Click the `Completed` button.

#### Task 3: Splunk Apps

By default Splunk comes with the Search & Reporting app. This is where you can enter Splunk queries to search through the data ingested by Splunk.

To install more apps you need to click on "+ Find More Apps" in the Apps panel or "Splunk Apps" in the Explore Splunk panel. You can either install them directly from within Splunk or download them from the [Splunkbase](https://splunkbase.splunk.com/). Also make sure to refer to the Splunk [documentation](https://docs.splunk.com/Documentation/Splunk/8.1.2/Admin/Managingappobjects) on managing Splunk apps.

Now install the Splunk add-on from the desktop.

##### Question 3) What is the 'Folder Name' for the add-on?

In the Apps view look for "Microsoft Sysmon Add-on" after adding the add-on to Splunk.

**Solution:** TA-microsoft-sysmon

##### Question 4) What is the Version?

**Solution:** 10.6.2

#### Task 4: Adding Data

Splunk is able to ingest quite a lot of data from many different providers, which is then processed and transformed into a series of individual events. The data sources are then grouped into categories and presented to you for analysis.

|       Data source       |                              Description                              |
| :---------------------: | :-------------------------------------------------------------------: |
|  Files and directories  |              Data that comes from files and directories               |
|     Network events      | Remote data from any network port and SNMP events from remote devices |
|      IT Operations      |                 IT Ops such as Nagios, NetApp, Cisco                  |
|     Cloud services      |                Cloud services, such as AWS or Kinesis                 |
|    Database services    |                        Databases such as MySQL                        |
|    Security services    |     Security services such as McAfee, Microsoft Active Directory      |
| Virtualization services |                Virtualization services such as VMWare                 |
|   Application servers   |           Application servers such as WebLogic or WebSphere           |
|     Window sources      |        Windows Event Log, Registry, Performance and many more         |
|      Other sources      |              FIFO queues, scripted inputs from APIs etc               |

More information about data sources can be found [here](https://docs.splunk.com/Documentation/Splunk/8.1.2/Data/Getstartedwithgettingdatain#Use_apps_to_get_data_in).

This room focuses on Sysmon Logs.

Now click on Add Data in the home screen. Select Upload and upload the tutorial files from the Desktop.

##### Question 5) Upload the Splunk tutorial data on the Desktop. How many events are in this source?

Open the Search app and search for "source="tutorialdata.zip:*".

**Solution:** 109,864

#### Task 5: Splunk Queries

Splunk queries are how you are able to query data fed to Splunk through the Search interface.

To focus on a specific source you can specify `source=<target_source>`, e.g. `source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"` to query Sysmon Event Logs.

To show events with a specific ID try `source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventID=12` or filter by keywords like this `source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" GoggleUpdate.exe`.

Another useful aspect is that you can query for phrases like this `* "failed password for sneezy"`.

Refer to the [Splunk Quick Reference Guide](https://www.splunk.com/pdfs/solution-guides/splunk-quick-reference-guide.pdf) for more tips on searching and filtering.

##### Question 6) Use Splunk to Search for the phrase failed password using tutorialdata.zip as the source

Click the `Completed` button.

##### Question 7) What is the sourcetype?

This is part of the results when executing the previously mentioned query.

**Solution:** www1/secure

##### Question 8) In the search result, look at the Patterns tab.

Click the `Completed` button.

##### Question 9) What is the last username in this tab?

**Solution:** myuan

##### Question 10) Search for failed password events for this specific username. How many events are returned?

**Solution:** 16

#### Task 6: Sigma Rules

*Sigma is a generic and open signature format that allows you to describe relevant log events in a straightforward manner. The rule format is very flexible, easy to write and applicable to any type of log file. The main purpose of this project is to provide a structured form in which researchers or analysts can describe their once developed detection methods and make them shareable with others.*

Sigma rules are written in YAML and is a great format for exchanging queries between different SIEMs.

The Github [repo](https://github.com/Neo23x0/sigma) goes in depth about installation, examples and usage and an online [tool](https://uncoder.io/) can be used to do the conversion.

##### Question 11) Use the Select document feature. What is the Splunk query for 'sigma: APT29'?

**Solution:** `CommandLine="*-noni -ep bypass $$*"`

##### Question 12) Use the Github Sigma repo. What is the Splunk query for 'CACTURTORCH Remote Thread Creation'?

Search through the Repo for "CACTUSTORCH" and copy the Sigma rule to Uncoder.io.

**Solution:** `(source="WinEventLog:*" (SourceImage="*\\System32\\cscript.exe" OR SourceImage="*\\System32\\wscript.exe" OR SourceImage="*\\System32\\mshta.exe" OR SourceImage="*\\winword.exe" OR SourceImage="*\\excel.exe") TargetImage="*\\SysWOW64\\*" NOT StartModule="*")`

#### Task 7: Dashboards & Visualizations

Splunk Dashboards are a great tool to provide quick visualization of data.

Follow the instructions provided by the task to setup a dashboard for the Search app.

##### Question 13) What is the highest EventID?

Try executing this search query `source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" | top limit=5 EventID`

**Solution:** 11

#### Task 8: Alerts

Alerts is a feature in Splunk with which you are able to monitor and respond to specific events. They can be configured to use a saved search for monitoring events in real-time or on schedule. They trigger when a specific condition is met and then take the defined course of action.

Familiarize yourself with the [alerting workflow](https://docs.splunk.com/Documentation/SplunkCloud/8.1.2012/Alert/AlertWorkflowOverview) from the documentation.

##### Question 14)

Click the `Completed` button.

#### Task 9: Conclusion

Congratulations, you finished the [Splunk 101](https://tryhackme.com/room/splunk101) room on [TryHackMe](https://tryhackme.com/).

Though this was but a brief introduction to Splunk there is a lot more to learn, because Splunk is a very extensive tool that simply can not be covered by a single room.

##### Question 15)

Click the `Completed` button to finish this room.
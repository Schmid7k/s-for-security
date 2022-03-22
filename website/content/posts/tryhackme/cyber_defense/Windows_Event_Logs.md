---
title: "Windows Event Logs"
date: 2022-03-21T21:26:25+01:00
draft: false
---

This write up refers to the [Windows Event Logs](https://tryhackme.com/room/windowseventlogs) room on [TryHackMe](https://tryhackme.com/).

In this room we are familiarizing ourselves with the Windows Event Log system and the tools you can use to interact with them.

#### Task 1: What are event logs?

Taken from Wikipedia: _Event logs record events taking place in the execution of a system to provide an audit trail that can be used to understand the activity of the system and to diagnose problems. They are essential to understand the activities of complex systems, particularly in applications with little user interaction (such as server applications)_.

Basically what this tells you is that the event logs are the first place to query when a machine is experiencing issues, because they can provide clues about the problem's origin.

Given the information event logs provide it is just as well possible that the combination of log file entries from multiple machines in combination with statistical analysis may turn up correlations between events that seem unrelated at first glance.

This is one of the reasons why **SIEM** (**Security information and event management**) tools are so powerful. Instead of accessing every machine in an enterprise network by itself, to look at the logs, SIEMs are used to aggregate all that information.

##### Question 1)

Start the VM attached to this task and connect to it via Remmina or open split view. Then click the `Completed` button.

#### Task 2: Event Viewer

Since Windows Event Logs are not stored as text files but rather in .evt or .evtx format it is not possible to view them using a text editor. Instead you can access them through Windows built-in tools.

The first tool we are looking at is the GUI application **Event Viewer**.
Event Viewer is divided into 3 panes. The left pane holds a tree listing the event log providers. The middle pane display a general overview and summary or the events specified to the selected provider. The right is the actions pane.

Windows Events are categorized into 5 types:

|  Event type   |                                              Description                                               |
| :-----------: | :----------------------------------------------------------------------------------------------------: |
|     Error     |         Indicates loss of data/functionality, e.g. if a service fails to load during startup.          |
|    Warning    |       Not necessarily significant but indicates a possible future problem, e.g. low disk space.        |
|  Information  | Describes a successful operation of an application/driver/service, e.g. successfully loading a driver. |
| Success Audit |     Events that record successful, audited security access, e.g. when a user logs in successfully.     |
| Failure Audit |                                       Opposite of Success Audit.                                       |

Standard Windows logs are visible by selecting **Windows Logs** in the left pane. Each log has an explanation:

|     Log     |                                        Description                                        |
| :---------: | :---------------------------------------------------------------------------------------: |
| Application |     Events logged by applications, e.g. file error recorded by a database operation.      |
|  Security   | Valid/Invalid login attempts and resource events such as creating/opening/deleting files. |
|   System    |          Events logged by system components, e.g. driver failure during startup           |
|  CustomLog  |                  Events logged by applications that create custom logs.                   |

The next section in th left pane is **Applications and Services Logs**. Here you can go down `Microsoft > Windows > Powershell > Operational` to see log events created by using PowerShell.

Use this to explore how Event Logger displays event information and how to change and filter for logging properties.

##### Question 2)

Open the `Microsoft > Windows > PowerShell > Operational` log in Event Viewer.

##### Question 3) What is the Event ID for the first event?

Scroll through the list of events until you find the first event logged for PowerShell.

**Solution:** 40961

##### Question 4) Filter on Event ID 4104. What was the 2nd command executed in the PowerShell session?

From the Actions tab select _Filter Current Log_ and filter by Event ID 4104. Then take a look at the second entry.

**Solution:** whoami

##### Question 5) What is the Task Category for Event ID 4104?

Look at the _Task Category_ column in the middle pane.

**Solution:** Execute a Remote Command

##### Question 6)

Navigate to `Applications and Services Logs > Windows PowerShell` and click the `Completed` button.

##### Question 7) What is the Task Category for Event ID 800?

Look at the _Task Category_ colum in the middle pane.

**Solution:** Pipeline Execution Details

#### Task 3: wevtutil.exe

wevtutil.exe is a great tool for when you don't want to look through thousands of events by yourself but rather write scripts to do the work for you.

Take from Microsoft wevtutil.exe _enables you to retrieve information about event logs and publishers. You can also use this command to install and uninstall event manifests, to run queries, and to export, archive, and clear logs._

Access its help file by calling `wevtutil.exe /?` and try to understand how to use it.

##### Question 8) How many log names are in the machine?

You may need to research into how to count line numbers of command output.

**Solution:** Execute `wevtutil el | find /c /v ""`. This command prints out all log names and pipes the output to `find` which then counts the lines. The result is **1071**.

##### Question 9) What is the definition for the query-events command?

Look at the help section of the query-events command.

**Solution:** Read events from an event log, log file or using structured query.

##### Question 10) What option would you use to provide a path to a log file?

Again look at the help section of the query-events command.

**Solution:** /lf:true

##### Question 11) What is the **VALUE** for /q?

Look at the help section of the query-events command under the options header.

**Solution:** xpath query

##### Question 12)

All coming questions are based on the command `wevtutil qe Application /c:3 /rd:true /f:text`.

Click on the `Completed` button.

##### Question 13) What is the log name?

**Solution:** Application

##### Question 14) What is the /rd option for?

Look at the output of `wevtutil qe /h`.

**Solution:** Event Read Direction

##### Question 15) What is the /c option for?

Look at the output of `wevtutil qe /h`

**Solution:** Maximum number of events to read

#### Task 4: Get-WinEvent

Get-WinEvent is a PowerShell tool that _gets events from event logs and event tracing log files on local and remote computers._

Look at its Get-Help documentation to understand it better.

One incredibly useful command to keep in mind though is this:

```ps
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; ID=4104} | Select-Object -Property Message | Select-String -Pattern 'SecureString'
```

##### Question 16)

Use the [online](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/Get-WinEvent?view=powershell-7.1) help documentation for Get-WinEvent to answer the following question. Click the `Completed` button.

##### Question 17) Execute the command from Example 1. What are the names of the logs related to OpenSSH?

Execute the command from example 1 of the online documentation.

**Solution:** OpenSSH/Admin,OpenSSH/Operational

##### Question 17) Execute the command from Example 8. Instead of the string \*Policy\* search for \*PowerShell\*. What is the name of the 3rd log provider.

Execute the command from Example 8 and look at the output.

**Solution:** Windows-PowerShell-DesiredStateConfiguration-FileDownloadManager/...

##### Question 18) Execute the command from Example 9. Use Microsoft-Windows-PowerShell as the log provider. How many event ids are displayed for this event provider.

Execute the command from example 9 and pipe the output to `Measure-Object`.

**Solution:** 192

##### Question 19) How do you specify the number of events to display?

You can find the answer to this question in the documentation.

**Solution:** MaxEvents

##### Question 20) When using the FilterHashtable parameter and filtering by level, what is the value for Information?

Answer can also be found in the documentation.

**Solution:** 4

#### Task 5: XPath Queries

There is another way to filter event logs called **XPath**.

XPath (or XML Path Language) is supported by the Windows Event Log to query for events like so:

`XPath Query: *[System[(Level <= 3) and TimeCreated(timedeff(@SystemTime) <= 86400000]]]`

Both Get-WinEvent and wevtutil support XPath queries for filtering events.

To understand how to form a XPath query you should take a look at the [documentation](https://docs.microsoft.com/en-us/windows/win32/wes/consuming-events#xpath-10-limitations) but also at the middle pane of Event Viewer, when selecting the `Details` view for an event and setting it to `XML View`.

##### Question 21) Using Get-WinEvent and XPath, what is the query to find WLMS events with a System Time of 2020-12-15T01:09:08.940277500T?

Build up the XPath query by orienting yourself at the XML view of Event Viewer event logs.

**Solution:** `Get-WinEvent -LogName Application -FilterXPath '*/System/TimeCreated[@SystemTime="2020-12-15T01:09:08.940277500T"] and */System/Provider[@Name="WLMS"]'`

##### Question 22) Using Get-WinEvent and XPath, what is the query to find a user name Sam with an Logon Event ID of 4720?

Look at the example given for querying a specific user name event.

**Solution:** `Get-WinEvent -LogName Security -FilterXPath '*/System/EventID=4720 and */EventData/Data[@Name="TargetUserName"]="Sam"'`

##### Question 23) Based on the previous query, how many results are returned?

**Solution:** 2

##### Question 24) Based on the output from the question #2, what is Message?

**Solution:** A user account was created

##### Question 25) Still working with Sam as the user, what time was Event ID 4724 recorded? (MM/DD/YYYY H:MM:SS [AM/PM])

Execute the command from question #2 again but with EventID 4724.

**Solution:** 12/17/2020 1:57:14 PM

##### Question 26) What is the Provider Name?

This one can be answered from context. The Provider of Windows Security events, when it comes to account events, is always the same.

**Solution:** Microsoft-Windows-Security-Auditing

#### Task 6: Event IDs

This task introduces you to some additional resources, for querying Windows Event Logs, compiled from different providers.

Some noteable resources include:

- [The Windows Logging Cheat Sheet](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/580595db9f745688bc7477f6/1476761074992/Windows+Logging+Cheat+Sheet_ver_Oct_2016.pdf)
- [Spotting the Adversary with Windows Event Log Monitoring](https://apps.nsa.gov/iaarchive/library/reports/spotting-the-adversary-with-windows-event-log-monitoring.cfm)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Events to Monitor](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor)
- [The Windows 10 and Windows Server 2016 Security Auditing and Monitoring Reference](https://www.microsoft.com/en-us/download/confirmation.aspx?id=52630)
- [About Logging Windows](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.1)
- [Greater Visibility through PowerShell Logging](https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html)
- [Configure PowerShell logging to see PowerShell anomalies in Splunk UBA](https://docs.splunk.com/Documentation/UBA/5.0.4/GetDataIn/AddPowerShell)
- [Command-line Process Auditing](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing#try-this-explore-command-line-process-auditing)

##### Question 27)

Click the `Completed` button.

#### Task 7: Putting theory into practice

This is where the real fun begins. Everything is based on the `merged.evtx` file found on the Desktop of this room's virtual machine and there are different scenarios to play through:

_Scenario 1 (Quesions 1 & 2)_: The server admins have made numerous complaints to Management regarding PowerShell being blocked in the environment. Management finally approved the usage of PowerShell within the environment. Visibility is now needed to ensure there are no gaps in coverage. You researched this topic: what logs to look at, what event IDs to monitor, etc. You enabled PowerShell logging on a test machine and had a colleague execute various commands.

_Scenario 2 (Questions 3 & 4)_: The Security Team is using Event Logs more. They want to ensure they can monitor if event logs are cleared. You assigned a colleague to execute this action.

_Scenario 3 (Questions 5, 6, & 7)_: The threat intel team shared its research on Emotet. They advised searching for event ID 4104 and the text "ScriptBlockText" within the EventData element. Find the encoded PowerShell payload.

_Scenario 4 (Questions 8 & 9)_: A report came in that an intern was suspected of running unusual commands on her machine, such as enumerating members of the Administrators group. A senior analyst suggested searching for `C:\Windows\System32\net1.exe`. Confirm the suspicion.

##### Question 28) What event ID is to detect a PowerShell downgrade attack?

This question expects you to search for information online.

**Solution:** 400

##### Question 29) What is the Date and Time this attack took place? (MM/DD/YYYY H:MM:SS [AM/PM])

Take a look at the provided Log File and filter for Event ID 400.

**Solution:** 12/18/2020 7:50:33 AM

##### Question 30) A Log Clear event was recorded. What is the 'Event Record ID'?

Filtering by `Task Category` you can find a single `Log Clear` event.

**Solution:** 27736

##### Question 31) What is the name of the computer?

Look at the `Details` view in XML format when you found the `Log Clear` event.

**Solution:** PC01.example.corp

##### Question 32) What is the name of the first variable within the PowerShell command?

The intstruction for this scenario advised us to search for Event ID 4104 and the text "ScriptBlockText" within EventData. You can do so via the following XPath query:

```
Get-WinEvent -Path .\merged.evtx -FilterXPath '*/EventData/Data[@Name="ScriptBlockText"] and */System/EventID=4104" -Oldest -MaxEvents 1 | Format-List
```

**Solution:** $Va5w3n8

##### Question 33) What is the Date and Time this attack took place? (MM/DD/YYYY H:MM:SS [AM/PM])

**Solution:** 8/25/2020 10:09:28 PM

##### Question 34) What is the Execution Process ID?

Look at the XML view of this log event.

**Solution:** 6620

##### Question 35) What is the Group Security ID of the group she enumerated?

Find out the Event ID for Group Security Events by researching online.

The following query produces the solution:

```
Get-WinEvent -Path .\merged.evtx -FilterXPath '*/System/EventID=4799' -Oldest -MaxEvent 1 | Format-List
```

**Solution:** S-1-5-32-544

##### Question 36) What is the event ID?

Same command as previous question.

**Solution:** 4799

#### Task 8: Conclusion

Congratulations, you finished the [Windows Event Logs](https://tryhackme.com/room/windowseventlogs) room on [TryHackMe](https://tryhackme.com/).

Windows Event Logs are a great tool for Security Analysts and Incident Responders to search for clues and hints on what transpired on a compromised machine and they can give you useful insights, when queried/filtered correctly.

If you want to dive deeper into Event Monitoring take a look at these resources:

- [EVTX Attack Samples](https://github.com/sbousseaden/EVTX-ATTACK-SAMPLES)
- [PowerShell <3 the Blue Team](https://devblogs.microsoft.com/powershell/powershell-the-blue-team/)
- [Tampering with Windows Event Tracing: Background, Offense, and Defense](https://medium.com/palantir/tampering-with-windows-event-tracing-background-offense-and-defense-4be7ac62ac63)

##### Question 37)

Click on the `Completed` button to finish the room.

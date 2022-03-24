---
title: "Sysmon"
date: 2022-03-24T13:21:36+01:00
draft: false
---

This write up refers to the [Sysmon](https://tryhackme.com/room/sysmon) room on [TryHackMe](https://tryhackme.com/).

In this room we are familiarizing ourselves with **Sysmon**, a tool for monitoring and logging events on Windows systems.

#### Task 1: Introduction

Make sure you completed the rooms mentioned in the prerequisites.

##### Question 1)

Click the `Completed` button.

#### Task 2: Sysmon Overview

Per the official docs:  
_System Monitor (Sysmon) is a Windows system service and device driver that, once installed on a system, remains resident across system reboots to monitor and log system activity to the Windows event log. It provides detailed information about process creations, network connections, and changes to file creation time. By collecting the events it generates using Windows Event Collection or SIEM agents and subsequently analyzing them, you can identify malicious or anomalous activity and understand how intruders and malware operate on your network._

Sysmon takes care of gathering detailed logs as well as event tracing to assist you in identifying anomalies in your environment. To use it to the best of its abilities you would normally use in in conjunction with security information and event management (SIEM) systems for deeper analysis.

Events logged by Sysmon are stored in `Applications and Services Logs/Microsoft/Windows/Sysmon/Operational`

Before Sysmon is able to run it needs a config file. [Here](https://github.com/SwiftOnSecurity/sysmon-config) is an example config for identifying anomalies.

What follows is a list of important to know Event IDs when working with Sysmon:

1. Event ID 1
   1. Focus: Process creation events
   2. The following code snippet specifies the Event ID to pull from and the condition to look for.
      ```
      <RuleGroup name="" groupRelation="or">
          <ProcessCreate onmatch="exclude">
              <CommandLine condition="is">C:\Windows\system32\svchost.exe -k appmodel -p -s camsvc</CommandLine>
          </ProcessCreate>
      </RuleGroup>
      ```
2. Event ID 3
   1. Focus: Network Connection
   2. The following code snippet identifies files transmitted over open ports as well as open ports themselves and specifically port 4444, commonly used with Metasploit.
      ```
      <RuleGroup name="" groupRelation="or">
          <NetworkConnect onmatch="include">
              <Image condition="image">nmap.exe</Image>
              <DestinationPort name="Alert,Metasploit" condition="is">4444</DestinationPort>
          </NetworkConnect>
      </RuleGroup>
      ```
3. Event ID 7
   1. Focus: DLL loading
   2. The following code snippet looks for DLLs that have been loaded within the \Temp\ directory.
      ```
      <RuleGroup name="" groupRelation="or">
          <ImageLoad onmatch="include">
              <ImageLoaded condition="contains">\Temp\</ImageLoaded>
          </ImageLoad>
      </RuleGroup>
      ```
4. Event ID 8
   1. Focus: Processes injecting code into other processes
   2. The following code snippet looks at memory addresses for a specific condition indicating a Cobalt Strike beacon. Additionally it looks for injected processes that do not have a parent process.
      ```
      <RuleGroup name="" groupRelation="or">
          <CreateRemoteThread onmatch="include">
              <StartAddress name="Alert,Cobalt Strike" condition="end with">0B80</StartAddress>
              <SourceImage condition="contains">\</SourceImage>
          </CreateRemoteThread>
      </RuleGroup>
      ```
5. Event ID 11
   1. Focus: Overwritten/Created files
   2. The following code snippet is an example ransomware event monitory.
      ```
      <RuleGroup name="" groupRelation="or">
          <FileCreate onmatch="include">
              <TargetFilename name="Alert,Ransomware" condition="contains">HELP_TO_SAVE_FILES</TargetFilename>
          </FileCreate>
      </RuleGroup>
      ```
6. Event ID 12/13/14
   1. Focus: Registry modification
   2. The following code snippet looks for registry objects within _Windows\System\Scripts_, a common directory for adversaries to place scripts for persistence.
      ```
      <RuleGroup name="" groupRelation="or">
          <RegistryEvent onmatch="include">
              <TargetObject name="T1484" condition="contains">Windows\System\Scripts</TargetObject>
          </RegistryEvent>
      </RuleGroup>
      ```
7. Event ID 15
   1. Focus: Files created in alternate data stream
   2. The following code snippet looks for files with .hta extension within an alternate data stream.
      ```
      <RuleGroup name="" groupRelation="or">
          <FileCreateStreamHash onmatch="include">
              <TargetFilename condition="end with">.hta</TargetFilename>
          </FileCreateStreamHash>
      </RuleGroup>
      ```
8. Event ID 22
   1. Focus: DNS queries
   2. The following snippet will get all but those DNS events with target .microsoft.com.
      ```
      <RuleGroup name="" groupRelation="or">
          <DnsQuery onmatch="exclude">
              <QueryName condition="end with">.microsoft.com</QueryName>
          </DnsQuery>
      </RuleGroup>
      ```

##### Question 2)

Click the `Completed` button.

#### Task 3: Installing and Preparing Sysmon

Sysmon can easily be installed as a binary from [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon) or you can download the Sysinternals suite through the following PowerShell command:

`Download-SysInternalsTools C:\Sysinternals`

To start Sysmon open a PowerShell as Administrator and execute the following command:

`Sysmon.exe -accepteula -i sysmonconfig-export.xml`

Now that Sysmon is running you can look at Event Viewer to monitor events.

##### Question 3)

Deploy the machine attached to this task and click the `Completed` button.

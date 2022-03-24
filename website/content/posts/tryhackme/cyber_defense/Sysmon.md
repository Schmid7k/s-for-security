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

#### Task 4: Cutting out the Noise

Event monitoring will more often than not bombard your screen with thousands of events triggered by normal activity carried out by the monitored system. This is why it is important to know how to cut out the "noise", such that you, as an investigator, may focus on actually relevant events.

Here is a list of best practices regarding Sysmon.

- Exclude > Include
  - It is a lot easier to prioritize the exclusion of unnecessary events than the inclusion of interesting events
- CLI = Control
  - CLI gives you the ability to easily write scripts and code snippets that make your job easier, so make sure to incorporate tools like `Get-WinEvent` or `wevtutil.exe` into your workflow when analysing logs
- Know your environment
  - If you don't know the environment you are working in you don't know what is normal and what is suspicious.

##### Question 4)

Read through the task description, then click the `Completed` button.

##### Question 5) How many event ID 3 events are in C:\Users\THM-Analyst\Desktop\Scenarios\Practice\Filtering.evtx?

Open the file in Event viewer, then filter for all events with ID 3.

**Solution:** 73,591

##### Question 6) What is the UTC time created of the first network event in C:\Users\THM-Analyst\Desktop\Scenarios\Practice\Filtering.evtx?

In Event Viewer filter the events by "Date and Time", then look at the oldest entry.

**Solution:** 2021-01-06 01:35:50.464

#### Task 5: Hunting Metasploit

To hunt for Metasploit connections in your log files it is good practice to look for network connections originating from ports `4444` and `5555`, as `4444` is the default in Metasploit.

Use the following code snippet with Sysmon to identify active connections on port `4444` or `5555`

```
<RuleGroup name="" groupRelation="or">
	<NetworkConnect onmatch="include">
		<DestinationPort condition="is">4444</DestinationPort>
		<DestinationPort condition="is">5555</DestinationPort>
	</NetworkConnect>
</RuleGroup>
```

Or use the following PowerShell command to do the same using `Get-WinEvent`.

```
Get-WinEvent -Path <Path to Log> -FilterXPath '*/System/EventID=3 and */EventData/Data[@Name="DestinationPort"] and */EventData/Data=4444'
```

##### Question 7)

Read about how to hunt for Metasploit connections on a system. Then click the `Completed` button.

#### Task 6: Detecting Mimikatz

Mimikatz is used by Red Teamers and adversaries alike for Windows post-exploitation. To hunt for Mimikatz look for file creation, file execution from elevated processes, creation of remote threads or processes that Mimikatz creates.

The easiest way to hunt for Mimikatz is simply looking for files created with the name Mimikatz. The following code snippet does exactly that.

```
<RuleGroup name="" groupRelation="or">
	<FileCreate onmatch="include">
		<TargetFileName condition="contains">mimikatz</TargetFileName>
	</FileCreate>
</RuleGroup>
```

A more sophisticated way to hunt for Mimikatz is by looking for abnormal LSASS behaviour by using the ProcessAccess event ID. If LSASS is accessed by a process other than svchost.exe it should generally be considered suspicious. The following code snippet will help with that.

```
<RuleGroup name="" groupRelation="or">
	<ProcessAccess onmatch="exclude">
		<SourceImage condition="image">svchost.exe</SourceImage>
	</ProcessAccess>
	<ProcessAccess onmatch="include">
		<TargetImage condition="image">lsass.exe</TargetImage>
	</ProcessAccess>
</RuleGroup>
```

The same can be done using PowerShell and the `Get-WinEvent` command.

```
Get-WinEvent -Path <Path to Log> -FilterXPath '*/System/EventID=10 and */EventData/Data[@Name="TargetImage"] and */EventData/Data="C:\Windows\system32\lsass.exe"'
```

##### Question 8)

Read about how to hunt for Mimikatz traces on a system. Then click the `Completed` button.

#### Task 7: Hunting Malware

There are many types and forms of malware. This room focuses on RATs and backdoors.
Detecting and hunting for malware on a system is difficult if you don't know what to look for. That's why you need to identify the malware first before being able to search for clues regarding its behaviour.

One way to detect and hunt for malware is to filter for suspicious or well known malware ports, while at the same time excluding known network connections like OneDrive.

The following code snippet scans for ports `1034` and `1604`.

```
<RuleGroup name="" groupRelation="or">
	<NetworkConnect onmatch="include">
		<DestinationPort condition="is">1034</DestinationPort>
		<DestinationPort condition="is">1604</DestinationPort>
	</NetworkConnect>
	<NetworkConnect onmatch="exclude">
		<Image condition="image">OneDrive.exe</Image>
	</NetworkConnect>
</RuleGroup>
```

Or you can do something similar in PowerShell using `Get-WinEvent`.

```
Get-WinEvent -Path <Path to Log> -FilterXPath '*/System/EventID=3 and */EventData/Data[@Name="DestinationPort"] and */EventData/Data=<Port>'
```

##### Question 9)

Read about how to hunt for common malware types. Then click the `Completed` button.

#### Task 8: Hunting Persistence

Persistence is a technique used by attackers to establish a way that allows them to maintain access after compromising a system. Some common ways to do that include registry modification and startup scripts, which you can hunt for by using Sysmon with File Creation events and Registry Modification events.

The following code snippet monitors files created in the `\Startup\` or `\Start Menu` directories.

```
<RuleGroup name="" groupRelation="or">
	<FileCreate onmatch="include">
		<TargetFilename name="T1023" condition="contains">\Start Menu</TargetFilename>
		<TargetFilename name="T1165" condition="contains">\Startup\</TargetFilename>
	</FileCreate>
</RuleGroup>
```

The next code snippet monitors registry modification.

```
<RuleGroup name="" groupRelation="or">
	<RegistryEvent onmatch="include">
		<TargetObject name="T1060,RunKey" condition="contains">CurrentVersion\Run</TargetObject>
		<TargetObject name="T1484" condition="contains">Group Policy\Scripts</TargetObject>
		<TargetObject name="T1060" condition="contains">CurrentVersion\Windows\Run</TargetObject>
	</RegistryEvent>
</RuleGroup>
```

##### Question 10)

Read about how to hunt for persistency tools. Then click the `Completed` button.

#### Task 9: Detecting Evasion Techniques

Once an attacker was able to place malware on a system the next step would be to apply evasion techniques to avoid anti-virus or detection through similar software. There are a lot of evasion techniques that can be employed to hide malware from being detected but this room focus on Alternate Data Streams and Injections.

Alternate Data Streams are used by malware authors to hide malicious code INSIDE of other files as part of a different data stream apart from `$DATA`.
With Sysmon you are able to detect newly created streams to combat this exact situation.

Then there is injection, which comes in different types: Thread Hijacking, PE Injection, DLL Injection and more, though this room focuses on DLL Injection and backdooring DLLs. This is achieved through injecting malicious code into a DLL that is already in use by an application.

To hunt for Alternate Data Streams you can use the following code snippet.

```
<RuleGroup name="" groupRelation="or">
	<FileCreateStreamHash onmatch="include">
		<TargetFilename condition="contains">Downloads</TargetFilename>
		<TargetFilename condition="contains">Temp\7z</TargetFilename>
		<TargetFilename condition="ends with">.hta</TargetFilename>
		<TargetFilename condition="ends with">.bat</TargetFilename>
	</FileCreateStreamHash>
</RuleGroup>
```

To detect remote threads, which are often used in combination with other techniques, you can use this code snippet.

```
<RuleGroup name="" groupRelation="or">
	<CreateRemoteThread onmatch="exclude">
		<SourceImage condition="is">C:\Windows\system32\svchost.exe</SourceImage>
		<TargetImage condition="is">C:\Program Files (x86)\Google\Chrome\Application\chrome.exe</TargetImage>
	</CreateRemoteThread>
</RuleGroup>
```

Both can also be done in PowerShell with the following commands.

ADS detection:

```
Get-WinEvent -Path <Path to Log> -FilterXPath '*/System/EventID=15'
```

Remote Thread Creation detection:

```
Get-WinEvent -Path <Path to Log> -FilterXPath '*/System/EventID=8'
```

##### Question 11)

Read about how to hunt for evasion techniques employed by adversaries. Then click the `Completed` button.

#### Task 10: Practical Investigations

Time to put all that theory into practice!

_Investigation 1 - ugh, BILL THAT'S THE WRONG USB!_  
In this investigation, your team has received reports that a malicious file was dropped onto a host by a malicious USB. They have pulled the logs suspected and have tasked you with running the investigation for it.

Logs are located in C:\Users\THM-Analyst\Desktop\Scenarios\Investigations\Investigation-1.

_Investigation 2 - This isn't an HTML file?_  
Another suspicious file has appeared in your logs and has managed to execute code masking itself as an HTML file, evading your anti-virus detections. Open the logs and investigate the suspicious file.

Logs are located in C:\Users\THM-Analyst\Desktop\Scenarios\Investigations\Investigation-2.

_Investigation 3.1 - 3.2 - Where's the bounce when you need him?_  
Your team has informed you that the adversary has managed to set up persistence on your endpoints as they continue to move throughout your network. Find how the adversary managed to gain persistence using logs provided.

Logs are located in C:\Users\THM-Analyst\Desktop\Scenarios\Investigations\Investigation-3.1

and C:\Users\THM-Analyst\Desktop\Scenarios\Investigations\Investigation-3.2.

_Investigation 4 - Mom look! I built a botnet!_  
As the adversary has gained a solid foothold onto your network it has been brought to your attention that they may have been able to set up C2 communications on some of the endpoints. Collect the logs and continue your investigation.

Logs are located in C:\Users\THM-Analyst\Desktop\Scenarios\Investigations\Investigation-4.

##### Question 12) What is the full registry key of the USB device calling svchost.exe in Investigation 1?

Open the related event log in Event Viewer and look for an USB registry key.

**Solution:** HKLM\System\CurrentControlSet\Enum\WpdBusEnumRoot\UMB\2&37c186b&0&STORAGE#VOLUME#\_??\_USBSTOR#DISK&VEN_SANDISK&PROD_U3_CRUZER_MICRO&REV_8.01#4054910EF19005B3&0#\FriendlyName

##### Question 13) What is the device name when being called by RawAccessRead in Investigation 1?

Look at the Image description of RawAccessRead events.

**Solution:** \Device\HarddiskVolume3

##### Question 14) What is the first exe the process executes in Investigation 1?

This can be found in one of the Process Create events.

**Solution:** rundll32.exe

##### Question 15) What is the full path of the payload in Investigation 2?

This is listed as a parameter of OriginalFileName in one of the Process Create events.

**Solution:** C:\Users\IEUser\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5\S97WTYG7\update.hta

##### Question 16) What is the full path of the file the payload masked itself as in Investigation 2?

This is listed as ParentImage in one of the Process Create events.

**Solution:** C:\Users\IEUser\Downloads\update.html

##### Question 17) What signed binary executed the payload in Investigation 2?

This is listed as a parameter of OriginalFileName in one of the Process Create events.

**Solution:** C:\Windows\System32\mshta.exe

##### Question 18) What is the IP of the adversary in Investigation 2?

This is listed as DestinationIp in the Network connection detected event.

**Solution:** 10.0.2.18

##### Question 19) What back connect port is used in Investigation 2?

This is listed as DestinationPort in the Network connection detected event.

**Solution:** 4443

##### Question 20) What is the IP of the suspected adversary in Investigation 3.1?

This is listed as DestinationIP in the Network connection detected event.

**Solution:** 172.30.1.253

##### Question 21) What is the hostname of the affected endpoint in Investigation 3.1?

This is listed as Image in the Network connection detected event.

**Solution:** DESKTOP-O153T4R\q

##### Question 22) What is the hostname of the C2 server connecting to the endpoint in Investigation 3.1?

This is listed as DestinationIP in the Network connection detected event.

**Solution:** empirec2

##### Question 23) Where in the registry was the payload stored in Investigation 3.1?

This can be found in one of the Registry value set events.

**Solution:** HKLM\SOFTWARE\Microsoft\Network\debug

##### Question 24) What PowerShell launch code was used to launch the payload in Investigation 3.1?

This can be found in one of the Registry value set events.

**Solution:** "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -c "$x=$((gp HKLM:Software\Microsoft\Network debug).debug);start -Win Hidden -A \"-enc $x\" powershell";exit;

##### Question 25) What is the IP of the adversary in Investigation 3.2?

This can be found in one of the Network connection detected events.

**Solution:** 172.168.103.188

##### Question 26) What is the full path of the payload location in Investigation 3.2?

This can be found in one of the Process Created events.

**Solution:** c:\users\q\AppData:blah.txt'

##### Question 27) What was the full command used to create the scheduled task in Investigation 3.2?

This can be found in one of the Process Created events.

**Solution:** "C:\WINDOWS\system32\schtasks.exe" /Create /F /SC DAILY /ST 09:00 /TN Updater /TR "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String($(cmd /c ''more < c:\users\q\AppData:blah.txt'''))))\"

##### Question 28) What process was accessed by schtasks.exe that would be considered suspicious behaviour in Investigation 3.2?

This can be found in one of the RawAccessRead detected events when looking at the Details tab.

**Solution:** lsass.exe

##### Question 29) What is the IP of the adversary in Investigation 4?

This can be found in one of the Network connection detected events.

**Solution:** 172.30.1.253

##### Question 30) What port is the adversary operating on in Investigation 4?

This can be found in one of the Network connection detected events.

**Solution:** 80

##### Question 31) What C2 is the adversary utilizing in Investigation 4?

This can be easy to misunderstand but look at the DestinationIp option of some events and you will find the C2 name.

**Solution:** empire

#### Conclusion

Congratulations, you finished the [Sysmon](https://tryhackme.com/room/sysmon) room on [TryHackMe](https://tryhackme.com/).

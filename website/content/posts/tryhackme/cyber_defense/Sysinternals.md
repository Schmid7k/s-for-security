---
title: "Sysinternals"
date: 2022-03-15T12:51:01+01:00
draft: false
---

This write up refers to the [Sysinternals](https://tryhackme.com/room/btsysinternalssg) room on [TryHackMe](https://tryhackme.com/).

In this room we are familiarizing ourselves with the Windows-based **Sysinternals** tools.

#### Task 1: Introduction

Each tool falls into one of the following categories:

- File and Disk Utilities
- Networking Utilities
- Process Utilities
- Security Utilities
- System Information
- Miscellaneous

They were originally created in the '90s under the company name Wininternals Software. However, in 2006 Microsoft acquired Wininternals Software and the found of Wininternals joined Microsoft.

Sysinternals tools are extremely popular among IT professionals when it comes to managing Windows systems. They are that useful that even red teamers and adversaries came to use them in their operations.

##### Question 1) When did Microsoft acquire the Sysinternals tools?

This is explained in the summary of Sysinternals history.

**Solution:** 2006

##### Question 2)

Deploy the VM attached to this task and click the `Completed` button.

#### Task 2: Install the Sysinternals Suite

To download Sysinternals tools head over to https://docs.microsoft.com/en-us/sysinternals/downloads/ and download the tools you need. They are sorted alphabetically but you can also look for tools by category.

To download the entire Sysinternals suite, you can get it as a zip file from here https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite.

If you do not want to download the tools you can also use them from the Sysinternals Live URL https://live.sysinternals.com/.

##### Question 3) What is the last tool listed within the Sysinternals Suite?

Since the suite is sorted alphabetically you just need to scroll all the way down to the last tool in the list.

**Solution:** ZoomIt

#### Task 3: Using Sysinternals Live

Sysinternals Live service allows you to run Sysinternals tools directly from the command line or the Windows Explorer by providing the tools Live path e.g. **\\\live.sysinternals.com\tools\\\<toolname\>** or **live.sysinternals.com/\<toolname\>**.

However to make this work you need to install and run a WebDAV client on your machine. WebDAV allows local machine to access a remote machine running a WebDAV share.

On Windows 10 you need to manually start the WebDAV client via the command line and you have to enable **Network Discovery** from the **Network and Sharing Center**.

```ps
start-service webclient <-- Starts the WebDAV client
control.exe /name Microsoft.NetworkAndSharingCenter <-- Opens the Network and Sharing Center GUI
```

In the Network and Sharing Center go to `Change advanced sharing settings` and enable `Turn on network discovery`.

However, the attached VM is a Windows Server 2019 edition for which you firstly have to install the WebDAV client. Just call this from PowerShell:

```ps
Install-WindowsFeature WebDAV-Redirector -Restart
```

Now you can run Sysinternals tools live from the website.

##### Question 4) What service needs to be enabled on the local host to interact with live.sysinternals.com?

This is related to the explanation of why you need an active WebDAV client, if you want to use Sysinternals Live.

**Solution:** webclient

#### Task 4: File and Disk Utilities

Let's start with taking a look at some Sysinternals tool!

**Sigcheck**  
_**Sigcheck** is a command-line utility that shows file version number, timestamp information, and digital signature details, including certificate chains. It also includes an option to check a file’s status on VirusTotal, a site that performs automated file scanning against over 40 antivirus engines, and an option to upload a file for scanning._

An example use case would be to check for unsigned file in C:\\Windows\\System32 like so:

```ps
sigcheck -u -e C:\Windows\System32
```

`-u` Shows files that are unknown to VirusTotal  
`-e` Scan only executables

**Streams**  
_The NTFS file system provides applications the ability to create **alternate data streams** of information. By default, all data is stored in a file's main unnamed data stream, but by using the syntax 'file:stream', you are able to read and write to alternates._

Alternate Data Streams are a concept specific to NTFS that allow file to contain more than one stream of data. Windows Explorer is not able to view ADS natively but you can view this data via third party executables or through PowerShell.

Malware authors like to use ADS to hide malicious code behind the curtain of a legit application.

**Stream** can be used to view ADS like so:

```ps
streams .\SysinternalsSuite.zip
```

You can read more about stream [here](https://docs.microsoft.com/en-us/sysinternals/downloads/streams).

**SDelete**  
_**SDelete** is a command line utility that takes a number of options. In any given use, it allows you to delete one or more files and/or directories, or to cleanse the free space on a logical disk._

It has been used by adversaries due to its implementation of **DOD 5220.22-M** (Department of Defense clearing and sanitizing protocol), because of which it is able to wipe data from a disk with the following methodology:

1. Write a zero and verify
2. Write a one and verify
3. Write a random character and verify

##### Question 5) There is a txt file on the desktop named file.txt. Using one of the three discussed tools in this task, what is the text within the ADS?

Use streams to get the ADS name, then open the file from PowerShell like so:

```ps
.\file.txt:<stream_name>
```

**Solution:** I am hiding in the stream.

#### Task 5: Networking Utilities

**TCPView**  
_**TCPView** is a Windows program that will show you detailed listings of all TCP and UDP endpoints on your system, including the local and remote addresses and state of TCP connections. On Windows Server 2008, Vista, and XP, TCPView also reports the name of the process that owns the endpoint. TCPView provides a more informative and conveniently presented subset of the Netstat program that ships with Windows. The TCPView download includes Tcpvcon, a command-line version with the same functionality._

The Windows native equivalent to TCPView is **Resource Monitor**. You can open it from the command line using `resmon` or through the Performance tab within Task Manager.

##### Question 6) Using WHOIS tools, what is the ISP/Organization for the remote address in the screenshots above?

Try https://www.whois.com/whois/

**Solution:** Microsoft Corporation

#### Task 6: Process Utilities

**Autoruns**  
_This utility, which has the most comprehensive knowledge of auto-starting locations of any startup monitor, shows you what programs are configured to run during system bootup or login, and when you start various built-in Windows applications like Internet Explorer, Explorer and media players. These programs and drivers include ones in your startup folder, Run, RunOnce, and other Registry keys. Autoruns reports Explorer shell extensions, toolbars, browser helper objects, Winlogon notifications, auto-start services, and much more. Autoruns goes way beyond other autostart utilities._

Autoruns is great to search for malicious programs trying to establish Persistence on the local machine, because it checks for auto-starting applications.

**ProcDump**  
_**ProcDump** is a command-line utility whose primary purpose is monitoring an application for CPU spikes and generating crash dumps during a spike that an administrator or developer can use to determine the cause of the spike._

To learn more about ProcDump visit this [page](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump).

**Process Explorer**  
_The **Process Explorer** display consists of two sub-windows. The top window always shows a list of the currently active processes, including the names of their owning accounts, whereas the information displayed in the bottom window depends on the mode that Process Explorer is in: if it is in handle mode you'll see the handles that the process selected in the top window has opened; if Process Explorer is in DLL mode you'll see the DLLs and memory-mapped files that the process has loaded._

**Process Monitor**  
_**Process Monitor** is an advanced monitoring tool for Windows that shows real-time file system, Registry and process/thread activity. It combines the features of two legacy Sysinternals utilities, Filemon and Regmon, and adds an extensive list of enhancements including rich and non-destructive filtering, comprehensive event properties such as session IDs and user names, reliable process information, full thread stacks with integrated symbol support for each operation, simultaneous logging to a file, and much more. Its uniquely powerful features will make Process Monitor a core utility in your system troubleshooting and malware hunting toolkit._

ProcMon captures events and activity on the local machine to a **VERY** extensive range. To use it effectively you have to use it with Filters and configure it properly or you will be overwhelmed by the sheer mountain of information generated.

[Here](https://adamtheautomator.com/procmon/) is a proper guide to ProcMon.

**PsExec**  
_**PsExec** is a light-weight telnet-replacement that lets you execute processes on other systems, complete with full interactivity for console applications, without having to manually install client software. PsExec's most powerful uses include launching interactive command-prompts on remote systems and remote-enabling tools like IpConfig that otherwise do not have the ability to show information about remote systems._

Due to its features PsExec is a tool adversaries love to use to get a hold on a system. Read more about it [here](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec)

##### Question 7) Run Autoruns and inspect what are the new entries in the Image Hijacks tab compared to the screenshots above.

Call AutoRuns from PowerShell and select the Image Hijacks tab. Then click the `Completed` button.

##### Question 8) What entry was updated?

Look through the Image Hijacks list and compare it to the screenshot attached to the task description.

**Solution:** taskmgr.exe

##### Question 9) What is the updated value?

This is listed under the Image Path column.

**Solution:** c:\tools\sysint\procexp.exe

#### Task 7: Security Utilities

**Sysmon**  
_**System Monitor (Sysmon)** is a Windows system service and device driver that, once installed on a system, remains resident across system reboots to monitor and log system activity to the Windows event log. It provides detailed information about process creations, network connections, and changes to file creation time. By collecting the events it generates using Windows Event Collection or SIEM agents and subsequently analyzing them, you can identify malicious or anomalous activity and understand how intruders and malware operate on your network._

This tool is **very** comprehensive. Take a look at the Sysmon [room](https://tryhackme.com/room/sysmon) to learn how to use it.

##### Question 10)

Click on the `Completed` button.

#### Task 8: System Information

**WinObj**  
_**WinObj** is a 32-bit Windows NT program that uses the native Windows NT API (provided by NTDLL.DLL) to access and display information on the NT Object Manager's name space._

Remember the concept of Session 0 and Session 1 from the Core Windows Processes room? With WinObj you can take a deeper look at the different sessions.

##### Question 11)

Click on the `Completed` button.

#### Task 9: Miscellaneous

**BgInfo**  
_It automatically displays relevant information about a Windows computer on the desktop's background, such as the computer name, IP address, service pack version, and more._

Great in combination with servers. When someone RDPs into a server BgInfo can be used to display system information on the wallpaper.

Learn more about it [here](https://docs.microsoft.com/en-us/sysinternals/downloads/bginfo)

**RegJump**  
_This little command-line applet takes a registry path and makes Regedit open to that path. It accepts root keys in standard (e.g. HKEY_LOCAL_MACHINE) and abbreviated form (e.g. HKLM)._

Handy little command line tool that allows you to jump directly to a specific key and depth in the registry.

**Strings**  
_**Strings** just scans the file you pass it for UNICODE (or ASCII) strings of a default length of 3 or more UNICODE (or ASCII) characters. Note that it works under Windows 95 as well._

Strings is an amazing tool to quickly check for human readable text in malicious binaries which could give away a lot about the intentions of the application.

##### Question 12) Run the Strings tool on ZoomIt.exe. What is the full path to the .pdb file?

Try

```ps
strings .\ZoomIt.exe | findstr pdb
```

**Solution:** C:\agent_work\112\s\Win32\Release\ZoomIt.pdb

#### Task 10: Conclusion

Congratulations, you finished the [Sysinternals](https://tryhackme.com/room/btsysinternalssg) room on [TryHackMe](https://tryhackme.com/).

Sysinternals tools host a great combination of useful analysis tools everyone should be familiar with.

If you want to learn more about them take a look at the following resources:

- https://docs.microsoft.com/en-us/archive/blogs/markrussinovich/
- https://techcommunity.microsoft.com/t5/windows-blog-archive/bg-p/Windows-Blog-Archive/label-name/Mark%20Russinovich
- https://www.youtube.com/watch?v=A_TPZxuTzBU
- https://www.youtube.com/watch?v=vW8eAqZyWeo

##### Question 13)

Click on the `Completed` button to finish the room.

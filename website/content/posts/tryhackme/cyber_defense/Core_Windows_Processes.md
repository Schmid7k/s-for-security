---
title: "Core Windows Processes"
date: 2022-03-14T13:43:31+01:00
draft: false
---

This write up refers to the [Core Windows Processes](https://tryhackme.com/room/btwindowsinternals) room on [TryHackMe](https://tryhackme.com/).

In this room we are familiarizing ourselves with core processes within a Windows system, how they operate under normal conditions but also how they can help us identify malicious processes.

#### Task 1: Introduction

Windows is the most used OS in the world, while at the same time most of its users have next to no understanding of its interworkings. Most people are content with the knowledge that their computers "just work".

Antivirus software was the first kind of protection software that was broadly adapted by end-users, though it is no longer enough to keep malware and attacks at bay. Modern tools such as **EDR (Endpoint Detection and Response)** have been developed to help with malware detection. But even with these layers of defense the Cyber Security world still needs dedicated human beings that investigate suspicious binaries or processes reported by the above mentioned tools. For this reason we need to understand the normal behaviour of the system we have to defend, which is the focus of this room.

##### Question 1)

Read the introductory text and deploy the attached virtual machine. Then click the `Completed` button.

#### Task 2: Task Manager

The **Task Manager** comes built-in with every Windows installation. It has a GUI through which you can see and monitor everything that is currently running on a Windows system. On top of that it provides you with information on resource usage and the ability to "kill" programs that are not responding.

Open the Task Manager through a right-click on the taskbar, then select `Task Manager` in the context menu.

Once opened, click on `More details`. This should expand the GUI and show a lot more information, exactly what we need.

The `Processes` tab should present a list of currently running processes. These are categorized into `Apps`, `Background processes` and `Windows processes`.

Each row is equipped with certain columns that provide additional information about the process. Expand the columns by right-clicking on any column and selecting additional options. This is great for performing an initial system analysis!

The `Details` tab provides information about core processes running on the system. Some really helpful columns to add are `Image path name` and `Command line`. With these two columns a skilled analyst can quickly identify outliers or suspicious behaviour, e.g. when a process is named after a Windows process but the Image path name or Command line is not what it is supposed to be.

However, there is one thing Task Manager lacks and that is information about **Parent-Child** relationships in processes. This is where tools like **Process Hacker** or **Process Explorer** come into play.

##### Question 2)

Familiarize yourself with the Task Manager. Then click the `Completed` button.

#### Task 3: System

**System** is the first windows process we are going to take a look at. Normally PIDs (Process Identifiers) are given at random to new process, this is not the case for the System process. The PID for System is always 4. Here is its official definition from _Windows Internals 6th Edition_:

"_The System process (process ID 4) is the home for a special kind of thread that runs only in kernel mode a kernel-mode system thread. System threads have all the attributes and contexts of regular user-mode threads (such as a hardware context, priority, and so on) but are different in that they run only in kernel-mode executing code loaded in system space, whether that is in Ntoskrnl.exe or in any other loaded device driver. In addition, system threads don't have a user process address space and hence must allocate any dynamic storage from operating system memory heaps, such as a paged or nonpaged pool._"

To understand the differences between user mode and Kernel-mode read through [this](https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode).

Time to explore the normal behaviour of the System process. Start `Process Explorer` and view the properties of System.

Process Explorer gives the following information:

- **Image Path:** N/A
- **Parent Process:** None
- **Number of Instances:** One
- **User Account:** Local System
- **Start Time:** At boot time

While Process Hacker shows this:

- **Image Path:** C:\Windows\system32\ntoskrnl.exe (NT OS Kernel)
- **Parent Process:** System Idle Process (0)

Process Hacker will also confirm that this is "Verified" Microsoft Windows behaviour.

So, what is unusual behaviour for System?

- A parent process (aside from System Idle Process (0))
- Multiple instances of System
- A different PID than 4
- Not running in Session 0

##### Question 3) What PID should System always be?

The solution can be found multiple times in the text about System.

**Solution:** 4

#### Task 4: System > smss.exe

**smss.exe (Session Manager Subsystem)**, or **Windows Session Manager**, is responsible for creating new sessions. It is the first user-mode process started by the kernel.

This process starts the kernel and user mode of the Windows subsystem (more [here](https://en.wikipedia.org/wiki/Architecture_of_Windows_NT)), including `win32k.sys (kernel mode)`, `winsrv.dll (user mode)` and `csrss.exe (user mode)`.

`csrss.exe (Windows subsystem)` and `wininit.exe` are started in Session 0, which is an isolated Windows session for the OS. `csrss.exe` and `winlogon.exe` are started in Session 1, the user session. The first child instance then creates child instances in new sessions by copying itself into the new session and self-terminating. (more [here](https://en.wikipedia.org/wiki/Session_Manager_Subsystem))

Additionally any subsystem listed in the `Required` value of `HKLM\System\CurrentControlSet\Control\Session Manager\Subsystems` in the registry is also launched.

On top of that SMSS is responsible for creating environment variables, virtual memory paging files and it starts winlogon.exe (Windows Logon Manager).

Normal behaviour of smss.exe looks like this:

- **Image Path:** %SystemRoot%\System32\smss.exe
- **Parent Process:** System
- **Number of Instances:** One master instance and child instance **per sessions**. The child instance exits after creating the session
- **User Account:** Local System
- **Start Time:** Within seconds of boot time for the master instance

So, what is unusual behaviour for smss.exe?

- A parent process other than System(4)
- Image path is different from C:\Windows\System32
- More than 1 running process
- User is not SYSTEM
- Unexpected registry entries for Subsystem

##### Question 4) What other two processes does smss.exe start in Session 1? (answer format: process1,process2)

The solution can be found in the third paragraph.

**Solution:** csrss.exe,winlogon.exe

#### Task 5: csrss.exe

**csrss.exe (Client Server Runtim Process)** is the user-mode side of the Windows subsystem. It is always running and critical to system operation. Termination will result in system failure! Its responsibilities lie in the Win32 console window and process thread creation and deletion. Each instance loads `csrsrv.dll`, `basesrv.dll` and `winsrv.dll` among others.

It is also responsible for making the Windows API available to other processes, mapping drive letters and handling the Windows shutdown process. (more [here](https://en.wikipedia.org/wiki/Client/Server_Runtime_Subsystem))

Normal behaviour of csrss.exe looks like this:

- **Image Path:** %SystemRoot%\System32\csrss.exe
- **Parent Process:** Created by an instance of smss.exe
- **Number of Instances:** Two or more
- **User Account:** Local System
- **Start Time:** Within seconds of boot time for the first 2 instances (for Session 0 and 1). Start times for additional instances occur as new sessions are created.

So, what is unusual behaviour for csrss.exe?

- An actual parent process (remember, smss.exe calls csrss.exe and then self-terminates)
- Image file path other then C:\Windows\System32
- Subtle misspellings to hide rogue process masquerading as csrss.exe in plain sight
- User is not SYSTEM

##### Question 5) What was the process which had PID 384 and PID 488?

Think about which process spawns csrss.exe

**Solution:** smss.exe

#### Task 6: wininit.exe

The **Windows Initialization Process (wininit.exe)** is responsible for launching services.exe (Service Control Manager), lsass.exe (Local Security Authority) and lsaiso.exe within Session 0. It is another critical Windows process that run in the background along with its child processes.

Note: _lsaiso.exe_ can only be seen when **Credential Guard and Key Guard** is enabled.

Normal behaviour of wininit.exe looks like this:

- **Image Path:** %SystemRoot%\System32\wininit.exe
- **Parent Process:** Created by an instance of smss.exe

So, what is unusual behaviour for wininit.exe?

- Subtle misspellings to hide rogue process in plain sight
- Multiple running instances
- Not running as SYSTEM

##### Question 6) Which process you might not see running if Credential Guard is not enabled?

The solution can be found in a small side note.

**Solution:** lsaiso.exe

#### Task 7: wininit.exe > services.exe

The **Service Control Manager (SCM)** or **services.exe** is responsible for handling loading, interacting and starting/ending of system services. It maintains a database of services that can be queried with the built-in utility `sc.exe`

Information regarding services is stored in the registry, `HKLM\System\CurrentControlSet\Services`.

It also loads device drivers marked as auto-start in memory.

Once a user is logged into a machine this process is also responsible for setting the value of the _Last Known Good control set_, `HKLM\System\Select\LastKnownGood` to that of `CurrentControlSet`.

Additionally it is the parent to processes such as `svchost.exe`, `spoolsv.exe`, `msmpeng.exe` and `dllhost.exe`. (more [here](https://en.wikipedia.org/wiki/Service_Control_Manager))

Normal behaviour of services.exe looks like this:

- **Image Path:** %SystemRoot%\System32\services.exe
- **Parent Process:** wininit.exe
- **Number of Instances:** One
- **User Account:** Local System
- **Start Time:** Within seconds of boot time

So, what is unusual behaviour for services.exe?

- Parent process other than wininit.exe
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue process in plain sight
- Multiple running instances
- Not running as SYSTEM

##### Question 7) How many instances of services.exe should be running on a Windows system?

The solution can be found in the normal behaviour section.

**Solution:** 1

#### Task 8: wininit.exe > services.exe > svchost.exe

The **Service Host (Host Process for Windows Services)**, or **svchost.exe**, is responsible for hosting and managing Windows services.

Services ran by this process are implemented as DLLs. The DLL itself is stored in the registry for the service under the `Parameters` subkey in `ServiceDLL` with the full path `HKLM\System\CurrentControlSet\Services\SERVICE NAME\Parameters`.

It is important to note that the binary path to svchost.exe from the perspective of a service must always include a `-k` identifier, e.g. `C:\Windows\system32\svchost.exe -k exampleservice.dll`. This is how a legitimate svchost.exe process is called.

This parameter is for grouping similar services to share the same process for the benefit of reducing resource consumption. Though since Windows 10 Version 1703 machines with more than 3.5 GB of memory will run each service in its own process. (more [here](https://en.wikipedia.org/wiki/Svchost.exe))

Since svchost.exe will always have multiple processes running on a windows machine it has often been a target for malicious use.

Normal behaviour of svchost.exe looks like this:

- **Image Path:** %SystemRoot%\System32\svchost.exe
- **Parent Process:** services.exe
- **Number of Instances:** Many
- **User Account:** Varies (SYSTEM, Network Service, Local Service) depending on the svchost.exe instance. In Windows 10 some instances can run as the logged-in user.
- **Start Time:** Typically within seconds of boot time. Other instances can be started after boot

So, what is unusual behaviour for svchost.exe?

- A parent process other than services.exe
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue process in plain sight
- The absence of the -k parameter

##### Question 8) What single letter parameter should always be visible in the Command line or Binary path?

The solution can be found in the unusual behaviour section.

**Solution:** k

#### Task 9: lsass.exe

Per Wikipedia, "_Local Security Authority Subsystem Service (LSASS) is a process in Microsoft Windows operating systems that is responsible for enforcing the security policy on the system. It verifies users logging on to a Windows computer or server, handles password changes, and creates access tokens. It also writes to the Windows Security Log._"

It creates security tokens for SAM (Security Account Manager), AD (Active Directory), and NETLOGON. Authentication packages are specified in `HKLM\System\CurrentControlSet\Control\Lsa`.

This process is a prominent target for tools such as **mimikatz** to dump credentials or to mimic in plain sight.

Normal behaviour of lsass.exe looks like this:

- **Image Path:** %SystemRoot%\System32\lsass.exe
- **Parent Process:** wininit.exe
- **Number of Instances:** One
- **User Account:** Local System
- **Start Time:** Within seconds of boot time

So, what is unusual behaviour for lsass.exe?

- A parent process other than wininit.exe
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue process in plain sight
- Multiple running instances
- Not running as SYSTEM

##### Question 9) What is the parent process for LSASS?

The solution can be found in the normal behaviour section.

**Solution:** wininit.exe

#### Task 10: winlogon.exe

**winlogon.exe**, or **Windows Logon**, is responsible for handling the **Secure Attention Sequence (SAS)**. It is the ALT+CTRL+DELETE key combination users press to enter their username & password.

It is also responsible for loading the user profile by loading the user's `NTUSER.DAT` into `HKCU` and loading the user's shell via `userinit.exe`. (more [here](<https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc939862(v=technet.10)?redirectedfrom=MSDN>))

Additionally it takes care of locking the screen and running the user's screensaver, among other functions. It is launched by smss.exe along with a copy of csrss.exe within Session 1.

Normal behaviour of winlogon.exe looks like this:

- **Image Path:** %SystemRoot%\System32\winlogon.exe
- **Parent Process:** Created by an instance of smss.exe that exits, so analysis tools usually do not provide the parent process name.
- **Number of Instances:** One or more
- **User Account:** Local System
- **Start Time:** Within seconds of boot time for the first instance (for Session 1). Additional instances occur as new sessions are created, typically through Remote Desktop or Fast User Switching logons.

So, what is unusual behaviour for winlogon.exe?

- An actual parent process (remember, smss.exe self-terminates after calling it)
- Image file path other than C:\Windows\System32
- Subtle misspellings to hide rogue process in plain sight
- Not running as SYSTEM
- Shell value in the registry other than explorer.exe

##### Question 10) What is the non-existent parent process for winlogon.exe?

The solution can be found in the normal behaviour section.

**Solution:** smss.exe

#### Task 11: explorer.exe

**explorer.exe**, or **Windows Explorer**, is the process that gives users access to their folders and files, while at the same time taking care of the Start Menu, Taskbar etc.

Winlogon runs `userinit.exe`, which launches the value in `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell`. Userinit.exe exits after spawning explorer.exe, which is why explorer.exe does not have a parent process though it will have **many** child processes.

Normal behaviour of explorer.exe looks like this:

- **Image Path:** %SystemRoot%\explorer.exe
- **Parent Process:** Created by userinit.exe which exits after spawning
- **Number of Instances:** One ore more per interactively logged-in user
- **User Account:** Logged-in user(s)
- **Start Time:** First instance when the first interactive user logon session begins

So, what is unusual behaviour for explorer.exe?

- An actual parent process
- Image file path other than C:\Windows
- Running as an unknown user
- Subtle misspellings to hide rogue process in plain sight
- Outbound TCP/IP connections

##### Question 11) What is the non-existent process for explorer.exe?

The solution can be found in the normal behaviour section.

**Solution:** userinit.exe

#### Task 12: Conclusion

Congratulations, you finished the [Core Windows Processes](https://tryhackme.com/room/btwindowsinternals) room on [TryHackMe](https://tryhackme.com/).

Since Windows is a dynamic landscape the list of core process to look out for when analysing a compromised system keeps expanding. Some additional processes to look out for meanwhile are `RuntimeBroker.exe` and `taskhostw.exe`. Feel free to research these processes yourself to further progress your understanding of Windows internal behaviour.

If you want to dig deeper take a look at the resources used for this room:

- https://www.threathunting.se/tag/windows-process/
- https://www.sans.org/posters/hunt-evil/
- https://docs.microsoft.com/en-us/sysinternals/resources/windows-internals

##### Question 12)

Click on the `Completed` button to finish the room.

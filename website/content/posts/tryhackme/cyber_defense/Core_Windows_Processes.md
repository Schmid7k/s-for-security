---
title: "Core Windows Processes"
date: 2022-03-14T13:43:31+01:00
draft: false
---

In this room we are familiarizing ourselves with core processes within a Windows system, how they operate under normal conditions but also how they can help you identify malicious processes.

So let's get into it!

#### Task 1: Introduction

The introductory text talks about how Windows is the most used OS in the world, while at the same time most of its users have next to no understandings of its interworkings. Most people are content with the knowledge that their computers "just work".

It then goes on about how antivirus software was the first kind of protection software that was broadly adapted by end-users, though it is no longer enough to keep malware and attacks at bay. Modern tools such as **EDR (Endpoint Detection and Response)** have been developed to aid antivirus software. But even with these layers of defense the Cyber Security world still needs dedicated human beings that investigate suspicious binaries or processes reported by the above mentioned tools. For this reason we need to understand the normal behaviour of the system we have to defend, which is the focus of this room.

##### Question 1)

Read the introductory text and deploy the attached virtual machine. Then click the `Completed` button.

#### Task 2: Task Manager

The **Task Manager** comes built-in with every Windows installation. It has a GUI through which you can see and monitor everything that is currently running on a Windows system. On top of that it provides you with information on resource usage and whether a program is not responding, which you can then "kill" via the Task Manager.

Open the Task Manager through a right-click on the taskbar, then select `Task Manager` in the context menu.

Once it opened, click on `More details`. This should expand the GUI and show a lot more information, exactly what we need.

The `Processes` tab should present a list of currently running processes to you. These are categorized into `Apps`, `Background processes` and `Windows processes`.

Each row is equipped with certain columns that provide additional information about the process. You can expand the columns by right-clicking on any column and selecting additional options. This is great for performing your first analysis on a system!

The `Details` tab provides information about core processes running on the system. Here some really helpful columns to add are `Image path name` and `Command line`. With these two columns a skilled analyst can quickly identify outliers or suspicious behaviour, e.g. when a process is named after a Windows process but the Image path name or Command line is not what it is supposed to be.

However there is one thing Task Manager lacks and that is information about **Parent-Child** relationships in processes. This is where tools like **Process Hacker** or **Process Explorer** come into play.

##### Question 2)

Familiarize yourself with the Task Manager. The click the `Completed` button.

#### Task 3: System

**System** is the first windows process we are going to take a look at. Normally PIDs are given at random to new process, this is not the case for the System process. The PID for System is always 4. Here is its official definition from _Windows Internals 6th Edition_:

"_The System process (process ID 4) is the home for a special kind of thread that runs only in kernel mode a kernel-mode system thread. System threads have all the attributes and contexts of regular user-mode threads (such as a hardware context, priority, and so on) but are different in that they run only in kernel-mode executing code loaded in system space, whether that is in Ntoskrnl.exe or in any other loaded device driver. In addition, system threads don't have a user process address space and hence must allocate any dynamic storage from operating system memory heaps, such as a paged or nonpaged pool._"

To understand the differences between user mode and Kernel-mode read through [this](https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode).

Time to explore the normal behaviour of the System process. Start `Process Explorer` and view the properties of System.

Process Explorer should give you the following information:

- **Image Path:** N/A
- **Parent Process:** None
- **Number of Instances:** One
- **User Account:** Local System
- **Start Time:** At boot time

While Process Hacker should show you this:

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

Now we take a look at **smss.exe (Session Manager Subsystem)**. This process is also known as **Windows Session Manager** and is responsible for creating new sessions. It is the first user-mode process started by the kernel.

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

So, what is unusual behaviour for SMSS?

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

So, what is unusual behaviour for CSRSS?

- An actual parent process (remember, smss.exe calls csrss.exe and then self-terminates)
- Image file path other then C:\Windows\System32
- Subtle misspellings to hide rogue process masquerading as csrss.exe in plain sight
- User is not SYSTEM

##### Question 5) What was the process which had PID 384 and PID 488?

Think about which process spawns csrss.exe

**Solution:** smss.exe

#### Task 6: wininit.exe

The **Windows Initialization Process (wininit.exe)** is responsible for launching services.exe (Service Control Manager), lsass.exe (Local Security Authority) and lsaiso.exe within Session 0. It is another critical Windows process that run in the background along with its child processes.

_lsaiso.exe_ can only be seen when **Credential Guard and Key Guard** is enabled.

Normal behaviours of wininit.exe looks like this:

- **Image Path:** %SystemRoot%\System32\wininit.exe
- **Parent Process:** Created by an instance of smss.exe
- Subtle misspellings to hide rogue process in plain sight
- Multiple running instances
- Not running as SYSTEM

##### Question 6) Which process you might not see running if Credential Guard is not enabled?

It is explained in a small side note.

**Solution:** lsaiso.exe

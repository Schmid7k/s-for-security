---
title: "OpenVAS"
date: 2022-03-09T19:23:33+01:00
---

In this room we are familiarizing ourselves with the [OpenVAS](https://openvas.org/) vulnerability scanner.

So let's get into it!

#### Task 1: Introduction

The official documentation states:  
_OpenVAS is an application used to scan endpoints and web applications to identify and detect vulnerabilities. It is commonly used by corporations as part of their mitigation solutions to quickly identify any gaps in their production or even development servers or applications. This is not an end all be all solution but can help to get rid of any common vulnerabilities that may have slipped through the cracks._

##### Question 1)

Read the introductory text and click the `Completed` button.

#### Task 2: GVM Framework Architecture

This task talks about the structure of the _Greenbone Vulnerability Management (GVM)_ framework and how _OpenVAS_ fits into it.

![GVM structure](../../../../images/content/tryhackme/cyber_defense/openvas/GVM_breakdown.png "GVM structure")

The _GVM framework_ can be broken down into distinct sub-section: Front-End, Back-End, and Vulnerability/Information feed. Each part has different obligations.

**Front-End (GSA, Web Interfaces)**  
The front-end provides you with an accessible GUI via the web interface you can navigate to in your browser.

**Back-End (OSP, OpenVAS, Targets)**  
The back-end infrastructure conducts the actual vulnerability scans and data processing through _OpenVAS_ and _GVM_. The _Greenbone Vulnerability Manager_ acts like a middle man between scanners and the front-end user interface

**Vulnerability/Information Feed (NVT, SCAP CERT, User Data, Community Feed)**  
This part of _GVM_ contains information and vulnerability tests provided by that _Greenbone Community Feed_, which is the baseline for testing against systems. Here you may also provide user data in place of _NVTs_ and _SCAP CERTs_.

##### Question 2)

Read about _GVM_ architecture and click the `Completed` button.

#### Task 3: Installing OpenVAS

The by far easiest way to install _OpenVAS_ is through docker. Open a terminal and type in:

```bash
sudo apt install docker.io
sudo docker run -d -p 443:443 --name openvas mikesplain/openvas
```

and after a few minutes of installing and set up you should be able to navigate to [https://127.0.0.1](https://127.0.0.1) in your browser and be greeted by OpenVAS' login window.

Type in the default credentials `admin/admin` and you should see the dashboard.

##### Question 3)

Install and setup OpenVAS, then click the `Completed` button.

#### Task 4: Initial Configuration

Make sure that _OpenVAS_ is properly configured and ready to run by going to _Scans > Tasks_, click on the purple magic wand icon in the upper left corner and start the basic configuration wizard. Initiate a scan on `127.0.0.1` and wait until the scan finishes.

_This task can take **A LOT** of time and your browser may not be responsible while the scan is running. Take a break, stretch, get some tea or coffee and wait until it finishes._

After it is done you should be taken to a new dashboard for analyzing your completed and ongoing scans. Now you should also be able to navigate to _Scans > Reports_ and click on the newly created report about the scan on your localhost.

##### Question 4)

Complete the first scan and click the `Completed` button.

#### Task 5: Scanning Infrastructure

Now the real fun begins. Deploy the virtual machine attached to this task and make sure you are connected to the TryHackMe network via OpenVPN or the AttackBox.

Click on the star icon in the upper left corner of the _Scans_ dashboard to open the _New Task_ window. Here you can create and modify settings for a new _OpenVAS_ task. Begin by giving it a recognizable name.

Now you want to configure a new target by clicking the star icon next to the _Scan Targets_ option. Give your target a name and set the IP address under the _Hosts_ option to the address of the target VM. You can safely ignore all other options, they are only used for advanced vulnerability management solutions.

When you are done configuring the target finish task creation and you are brought back to the _Scans_ dashboard with the newly created task in the list of tasks.

##### Question 5)

Start the newly created scan by clicking the green arrow icon below the _Actions_ tab and click the `Completed` button.

#### Task 6: Reporting and Continuous Monitoring

Now we familiarize ourselves with _OpenVAS'_ reporting and monitoring capabilities. Download the report attached to the task and open it in your browser.

The report begins with some general information on when the scan was started, when it ended, the name of the task that ran, as well as some host specific information such as target IPs, how many vulnerabilities were found and how severe they were.

Afterwards _OpenVAS_ goes into detail about every vulnerability. How it was detected, why it was detected and how to mitigate it.

##### Question 6)

Read through the task description on how to read _OpenVAS_ reports and practice reporting and monitoring. Click the `Completed` button to finish the task.

#### Task 7: Practical Vulnerability Management

Challenge time! Download the report attached to the task and open it in your browser.

##### Question 7) When did the scan start in Case 001?

Look at the _Start_ tab in the _Host Summary_ table.

**Solution:** Feb 28, 00:04:46

##### Question 8) When did the scan end in Case 001?

Look at the _End_ tab in the _Host Summary_ table.

**Solution:** Feb 28, 00:21:02

##### Question 9) How many ports are open in Case 001?

Look at the _Port Summary_ table. (Hint: _general/tcp_ is not a port)

**Solution:** 3

##### Question 10) How many vulnerabilities were found in Case 001?

Look at the _High_, _Medium_, and _Low_ tabs in the _Host Summary_ table.

**Solution:** 5

##### Question 11) What is the highest severity vulnerability found? (MSxx-xxx)

_OpenVAS_ presents vulnerabilities in its reports sorted from highest to lowest severity. Therefore the highest severity vulnerability should be at the very top of the _Security Issues_ section.

**Solution:** MS17-010

##### Question 12) What is the first affected OS to this vulnerability?

Information on affected OS' can be found below the _Affected Software/OS_ tab.

**Solution:** Microsoft Windows 10 x32/x64 Edition

##### Question 13) What is the recommended vulnerability detection method?

Information on how the vulnerability was detected can be found below the _Vulnerability Detection Method_ tab.

**Solution:** Send the crafted SMB transaction request with fid = 0 and check the response to confirm the vulnerability.

#### Task 8: Conclusion

Congratulations, you finished the _OpenVAS_ room! If you want to learn more about _OpenVAS_ and _GVM_ take a look at the technical [documentation](https://docs.greenbone.net/) or head over to the [community portal](https://community.greenbone.net/).

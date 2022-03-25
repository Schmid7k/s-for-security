---
title: "Osquery"
date: 2022-03-25T14:52:32+01:00
draft: false
---

This write up refers to the [Osquery](https://tryhackme.com/room/osqueryf8) room on [TryHackMe](https://tryhackme.com/).

In this room we are familiarizing ourselves with [Osquery](https://osquery.io/), an open-source tool developed by Facebook for querying endpoints using SQL syntax.

#### Task 1: Introduction

Osquery is a popular tool for host and network level detection used by well-known companies like Facebook, Github or AT&T, which is why it is a good idea to familiarize yourself with it, if you are looking to enter the field or just want to up your knowledge.

##### Question 1)

Read the introductory text and click the `Completed` button.

#### Task 2: Installation

To install Osquery on your local machine follow the installation instructions.

- [Windows](https://osquery.readthedocs.io/en/stable/installation/install-windows/)
- [Linux](https://osquery.readthedocs.io/en/stable/installation/install-linux/)
- [macOS](https://osquery.readthedocs.io/en/stable/installation/install-macos/)
- [FreeBSD](https://osquery.readthedocs.io/en/stable/installation/install-freebsd/)

Also make sure to look into the [documentation](https://osquery.readthedocs.io/en/latest/installation/cli-flags/).

##### Question 2)

Start the attached VM. Then click the `Completed` button.

#### Task 3: Interacting with the Osquery Shell

To interact with Osquery open CMD or PowerShell and run `osqueryi`.

Make sure to check out the help menu by calling `.help`.

To list all available tables use the `.tables` command e.g. `.tables process` queries all tables associated with processes.

Once you found a table you want to examine call `.schema table_name` to list the table's schema.

##### Question 3) What is the Osquery version?

Execute `.version`.

**Solution:** 4.6.0.2

##### Question 4) What is the SQLite version?

Can also be found in the output of `.version`.

**Solution:** 3.34.0

##### Question 5) What is the default output mode?

Execute `.help`.

**Solution:** pretty

##### Question 6) What is the meta-command to set the output to show one value per line?

Can also be found in the output of `.help`.

**Solution:** .mode line

##### Question 7) What are the 2 meta-commands to exit osqueryi?

Can also be found in the output of `.help`.

**Solution:** .quit,.exit

#### Task 4: Schema Documentation

The schema documentation needed to solve the next questions can be found [here](https://osquery.io/schema/4.7.0/).

##### Question 8) What table would you query to get the version of Osquery installed on the Windows endpoint?

**Solution:** osquery_info

##### Question 9) How many tables are there for this version of Osquery?

**Solution:** 266

##### Question 10) How many of the tables for this version are compatible with Windows?

**Solution:** 96

##### Question 11) How many tables are compatible with Linux?

**Solution:** 155

##### Question 12) What is the first table listed that is compatible with both Linux and Windows?

**Solution:** arp_cache

#### Task 5: Creating Queries

Osquery can be queried using a simpler version of SQL. Most statements will be SELECT statements as you will rarely ever need UPDATE or DELETE statements unless you are dealing with extensions or run-time tables.

Other than that querying Osquery is as easy as writing SQL statements for a database.

- `SELECT * FROM processes;` Retrieves all information from the processes table.
- `SELECT pid, name, path FROM processes;` only retrieves the pid, name and path columns.
- You can use aggregation statements like `SELECT count(*) FROM processes`
- Or WHERE clauses to be more specific `SELECT pid, name, path FROM processes WHERE name='lsass.exe'`

Refer to the [documentation](https://osquery.readthedocs.io/en/stable/introduction/sql/) to learn more about Osquery's SQL syntax.

##### Question 13) What is the query to show the username field from the users table where the username is 3 characters long and ends with 'en'? (use single quotes in your answer)

**Solution:** `SELECT username FROM users WHERE username LIKE '%en';`

#### Task 6: Using Kolide Fleet

Kolide Fleet is an open-source Osquery Fleet Manager for querying multiple endpoints from Kolide Fleet's UI.

To start Kolide on the VM execute the following commands in an Ubuntu shell (sudo password is `tryhackme`):

1. `sudo redis-server --daemonize yes`
2. `sudo service mysql start`
3. `/usr/bin/fleet serve \--mysql_address=127.0.0.1:3306 \--mysql_database=kolide \--mysql_username=root \--mysql_password=tryhackme \--redis_address=127.0.0.1:6379 \--server_cert=/home/tryhackme/server.cert \--server_key=/home/tryhackme/server.key \--auth_jwt_key=JB+wEDR4V3bbhU4OlIMcXpcBQAaZc+4r \--logging_json`

Then you can navigate to [https://127.0.0.1:8080](https://127.0.0.1:8080) and log into Kolide Fleet with username `thmosquery` and password `tryhackme1!`.

Click on `Add New Host` and note down the Osquery secret. You will need it for the following command to add the VM to Kolide.

`launcher.exe --hostname=127.0.0.1:8080 --enroll_secret=<SECRET_KEY> --insecure`

Once you refresh the page that new machine should be presented to you. Now you can click on `Query > Create New Query` to launch an Osquery query directly through Kolide with additional options, such as being able to save a query, what targets to run it against and a description of what the query does.

##### Question 14) What is the Osquery Enroll Secret?

**Solution:** k3hFh30bUrU7nAC3DmsCCyb1mT8HoDkt

##### Question 15) What is the Osquery version?

You can find the answer in Kolide's dashboard.

**Solution:** 4.2.0

##### Question 16) What is the path for the running osqueryd.exe process?

Execute `SELECT * FROM processes WHERE path LIKE '%osquery%';`

**Solution:** C:\Users\Administrator\Desktop\launcher\windows\osqueryd.exe

#### Task 7: Osquery extensions

You can add additional functionality to Osquery by making use of its many extensions. Check out the [docs](https://osquery.readthedocs.io/en/stable/deployment/extensions/) to learn more.

Also here are 2 repos of extensions to play around with:

- [https://github.com/trailofbits/osquery-extensions](https://github.com/trailofbits/osquery-extensions)
- [https://github.com/polylogyx/osq-ext-bin](https://github.com/polylogyx/osq-ext-bin)

##### Question 17) According to the polylogyx readme, how many features does the plug-in add to the Osquery core?

When I did this room the amount of features was higher than when the room was created, so if you don't get the right solution at first try decrementing your number until you get it right.

**Solution:** 23

#### Task 8: Linux and Osquery

Use the Ubuntu terminal in the machine and launch Osquery

Also make sure to look at the On-Demand YARA scanning [here](https://osquery.readthedocs.io/en/stable/deployment/yara/).

##### Question 18) What is the current_value for kernel.osrelease?

Execute `SELECT * FROM kernel_info;`.

**Solution:** 4.4.0-17763-Microsoft

##### Question 19) WHat is the uid for the bravo user?

Execute `SELECT * FROM users WHERE username='bravo';`.

**Solution:** 1002

##### Question 20) One of the users performed a Binary Padding attack. What was the target file in the attack?

Definition of [Binary Padding](https://attack.mitre.org/techniques/T1027/001/).

Execute `SELECT * FROM shell_history;`. This will print out the shell history for you to analyze.

**Solution:** notsus

##### Question 21) What is the hash value for this file?

Call `md5sum` with the file as parameter.

**Solution:** 3df6a21c6d0c554719cffa6ee2ae0df7

##### Question 22) Check all file hashes in the home directory for each user. One file will not show any hashes. Which file is that?

Execute `SELECT * FROM hash WHERE path LIKE '/home/%/%';`

**Solution:** fleet.zip

##### Question 23) There is a file that is categorized as malicious in one of the home directories. QUery the Yara table to find this file. use the sigfile which is save in /var/osquery/yara/scanner.yara. Which file is it?

Execute `SELECT * FROM yara WHERE path LIKE '/home/%/%' AND sigfile='/var/osquery/yara/scanner.yara';`.

**Solution:** notes

##### Question 24) What were the matches?

**Solution:** eicar_av_test,eicar_substring_test

##### Question 25) Scan the file from Q#3 with the same Yara file. What is the entry for strings?

**Solution:** $eicar_substring:1b

#### Task 9: Windows and Osquery

Make sure to load the polylogyx extension before attempting the questions.

`osqueryi --allow-unsafe --extension "C:\Program Files\osquery\extensions\osq-ext-bin\plgx_win_extension.ext.exe`

##### Question 26) What is the description for the Windows Defender Service?

The Windows Defender Service is called _WinDefend_ in Windows.

Execute `SELECT * FROM services WHERE name LIKE 'WinD%';`.

**Solution:** Helps protect users from malware and other potentially unwanted software

##### Question 27) There is another security agent on the Windows endpoints. What is the name of this agent?

Try `SELECT * FROM programs WHERE name LIKE '%Agent%';`.

**Solution:** AlienVault Agent

##### Question 28) What is required with win_event_log_data?

Executing `SELECT * FROM win_event_log_data;` will produce an error with the solution.

**Solution:** source

##### Question 29) How many sources are returned for win_event_log_channels?

Execute `SELECT COUNT(source) FROM win_event_log_channels;` to automatically count all entries.

**Solution:** 1076

##### Question 30) What is the schema for win_event_log_data?

Execute `.schema win_event_log_data` and copy the output.

**Solution:**

    CREATE TABLE win_event_log_data(`time`BIGINT,`datetime`TEXT,`source`TEXT,`provider_name`TEXT,`provider_guid`TEXT,`eventid`INTEGER,`task`INTEGER,`level`INTEGER,`keywords`BIGINT,`data`TEXT,`eid` TEXT HIDDEN);

##### Question 31) The previous file scanned on the Linux endpoint with Yara is on the Windows endpoint. What date/time was this file first detected? (Answer format: YYYY-MM-DD HH:MM:SS)

By querying win_event_log_channels like this

`SELECT source FROM win_event_log_channels WHERE source LIKE '%DEF%';`

you are able to find out how Microsoft Defender is listed as a source for querying win_event_log_data.

Now you need to head over to [here](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide) and search for the Event ID related to Windows Defender detecting malware. (Hint: It's 1116).

With this information you can build up the query to solve this challenge as follows:

`SELECT datetime FROM win_event_log_data WHERE source='Microsoft-Windows-Windows Defender/Operational' AND eventid=116 ORDER BY datetime LIMIT 1;`

**Solution:** 2021-04-01 00:50:44

##### Question 32) What is the query to find the first Sysmon event? Select only the event id, order by date/time, and limit the output to only 1 entry.

Again look for how Sysmon is called in the win_event_log_channels table, then apply the logic to the win_event_log_data table.

**Solution:** `SELECT eventid FROM win_event_log_data WHERE source='Microsoft-Windows-Sysmon/Operation' ORDER BY datetime LIMIT 1;`

##### Question 33) What is the Sysmon event id?

**Solution:** 16

#### Task 10: Conclusion

Congratulations, you finished the [Osquery](https://tryhackme.com/room/osqueryf8) room on [TryHackMe](https://tryhackme.com/).

To learn more about this very useful tool take a look at the following sources.

- File Integrity Monitoring [https://osquery.readthedocs.io/en/latest/deployment/file-integrity-monitoring/](https://osquery.readthedocs.io/en/latest/deployment/file-integrity-monitoring/)
- Process Auditing [https://osquery.readthedocs.io/en/latest/deployment/process-auditing/](https://osquery.readthedocs.io/en/latest/deployment/process-auditing/)
- Syslog Consumption [https://osquery.readthedocs.io/en/latest/deployment/syslog/](https://osquery.readthedocs.io/en/latest/deployment/syslog/)
- Community projects [https://osquery.io/](https://osquery.io/)
- Enterprise threat hunting [https://github.com/teoseller/osquery-attck](https://github.com/teoseller/osquery-attck)

If you are going to dive deeper into the field of incident response and security analysing you will see quickly that Osquery is often part of bigger frameworks and SIEMs like ELK or Splunk so it's good to know how to deal with this tool.

##### Question 34)

Click the `Completed` button to finish this room.

---
title: "Attacking Kerberos"
date: 2022-04-07T16:18:23+02:00
draft: false
---

This write up refers to the [Attacking Kerberos](https://tryhackme.com/room/attackingkerberos) room on [TryHackMe](https://tryhackme.com/).

#### Task 1: Introduction

In this room we are familiarizing ourselves with Kerberos, the windows ticket-granting service.

The task description gives a summary on Kerberos' components, how its ticket system works, common terminology when working with Kerberos as well as what requirements are needed for different attacks.

All task related questions can be answered by reading the summary on Kerberos.

##### Question 1) What does TGT stand for?

**Solution:** Ticket Granting Ticket

##### Question 2) What does SPN stand for?

**Solution:** Service Principal Name

##### Question 3) What does PAC stand for?

**Solution:** Privilege Attribute Certificate

##### Question 4) What two services make up the KDC?

**Solution:** AS, TGS

##### Question 5)

Deploy the machine and click the `Completed` button.

#### Task 2: Enumeration w/ Kerbrute

Before we can tackle the questions attached to this task we need to take care of some pre-requisites. First of all we need to add the DNS domain name and the target machine IP to `/etc/hosts`. Then we can download a precompiled Kerbrute binary from [GitHub](https://github.com/ropnop/kerbrute/releases), rename it to just `kerbrute` and make it executable `chmod +x kerbrute`. Now qw download the wordlist attached to the task and we can start brute forcing user accounts on the target machine with this command:

```
./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local <wordlist>
```

This gives us enough information to answer the questions.

##### Question 6) How many totals users do we enumerate?

**Solution:** 10

##### Question 7) What is the SQL service account name?

**Solution:** SQLService

##### Question 8) What is the second "machine" account name?

**Solution:** Machine2

##### Question 9) What is the third "user" account name?

**Solution:** User3

#### Task 3: Harvesting & Brute-Forcing Tickets w/ Rubeus

For this task we need to SSH into the machine with the given credentials. Once we are on the machine we can `cd Downloads` to navigate to the directory where a precompiled `Rubeus.exe` was left for us. Using this executable we can begin harvesting Ticket Granting Tickets (TGTs).

```
Rubeus.exe harvest /interval:30
```

This is enough to answers this task's questions.

##### Question 10) Which domain admin do we get a ticket for when harvesting tickets?

**Solution:** Administrator
##### Question 11) Which domain controller do we get a ticket for when harvesting tickets?

**Solution:** CONTROLLER-1

#### Task 4: Kerberoasting w/ Rubeus & Impacket

This rooms task description summarizes one of the most popular Kerberos attacks called Kerberoasting. It explains how to take advantage of it and how to mitigate against this kind of attack.

Luckily the `Rubeus.exe` on the target machine is enough to carry out this attack, so we can navigate into the `Downloads` directory again and execute

```
Rubeus.exe kerberoast
```

to dump the Kerberos hash of all users vulnerable to Kerberoasting. Copying these hashes over to our own machine allows us to crack them using `hashcat` and the modified rockyou wordlist that comes with the task.

```
hashcat -m 13100 -a 0 hash.txt wordlist.txt
```

This will be all to answers this task's questions.

##### Question 12) What is the HTTPService Password?

**Solution:** Su*******0

##### Question 13) What is the SQLService Password?

**Solution:** My***********#

#### Task 5: AS-REP Roasting w/ Rubeus

Much like Kerberoasting AS-REP roasting dumps the krbasrep5 hashes of user accounts that have Kerberos pre-authentication disabled. We can use this to our advantage by again using `Rubeus`.

```
Rubeus.exe asreproast
```

Again we get a list of hashes from vulnerable users, we can copy them over to our machine and put them into a `hashes.txt`. However before we can crack them with `hashcat` we need to insert a *23$* after *$krb5asrep$* for every hash.

Then we can brute force them with the same wordlist as in task 4.

```
hashcat -m 18200 hashes.txt wordlist.txt
```

##### Question 14) What hash type does AS-REP Roasting use?

Looking through the `hashcat` help menu gives us the answer to this question.

**Solution:** Kerberos 5 AS-REP etype 23

##### Question 15) Which User is vulnerable to AS-REP roasting?

**Solution:** User3

##### Question 16) What is the User's Password?

**Solution:** P*******3

##### Question 17) Which Admin is vulnerable to AS-REP roasting?

**Solution:** Admin2

##### Question 18) What is the Admin's Password?

**Solution:** P*******2

#### Task 6: Pass the Ticket w/ mimikatz

The next attack we get to take a look at is the pass the ticket attack. This attack dumps the Ticket Granting Ticket (TGT) from the Local Security Authority Subsystem Service (LSASS) memory of the machine. The dumped ticket can then be used to gain domain admin privileges, if a domain admin ticket was in the LSASS memory.

This attack can be executed on the target machine using mimikatz.

```js
mimikatz.exe
privilege::debug // Check if we have administrator privileges
sekurlsa::tickets /export // Export all .kirbi tickets into the current directory
```

Check through the extracted tickets and look for an administrator ticket from krbtgt.

With the ticket we can finally execute the actual Pass the Ticket attack with mimikatz.

```js
kerberos::ptt <ticket> // Replace <ticket> with the ticket you found earlier
```

We can verify that the attack was successful by exiting mimikatz and executing `klist`. The output should tell us that we are that user we were trying to impersonate.

##### Question 19)

Click the `Completed` button and continue.

#### Task 7: Golden/Silver Ticket Attacks w/ mimikatz

This task explains the Golden/Silver ticket attack against Kerberos by using mimikatz. This attack works by dumping the ticket-granting ticket of a user on the domain, preferably a domain admin (for a silver ticket) or the krbtgt ticket (for a golden ticket).

If we wanted to create a golden ticket we would execute the following steps.

```
mimikatz.exe
privilege::debug // Check for administrator privileges
lsadump::lsa /inject /name:krbtgt // Dump the has and security identifier needed to create a Golden Ticket. Change /name: to the name of a service or domain admin account if you want a Silver Ticket instead
Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:500 // Command for creating a golden ticket.
misc::cmd // Open a new elevated command prompt with the given ticket in mimikatz
```

##### Question 20) What is the SQLService NTLM Hash?

**Solution:** c******************************a

##### Question 21) What is the Administrator NTLM Hash?

**Solution:** 2******************************6

#### Task 8: Kerberos Backdoors w/ mimikatz

This task teaches us how to create a skeleton key for persistent access to a Kerberos domain. There is no real question attached to this task so we can learn about skeleton keys when we want to.

##### Question 22)

Click the `Completed` button and continue.

#### Task 9: Conclusion

Congratulations, you finished the [Attacking Kerberos](https://tryhackme.com/room/attackingkerberos) room on [TryHackMe](https://tryhackme.com/)!

If you want to learn more about Kerberos and how to exploit it take a look at the following resources:

- https://medium.com/@t0pazg3m/pass-the-ticket-ptt-attack-in-mimikatz-and-a-gotcha-96a5805e257a
- https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat
- https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1
- https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/
- https://www.varonis.com/blog/kerberos-authentication-explained/
- https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It-wp.pdf
- https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf
- https://www.redsiege.com/wp-content/uploads/2020/04/20200430-kerb101.pdf

##### Question 23)

Click on the `Completed` button to finish the room.
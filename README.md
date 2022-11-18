# SCDRM - Security Change Disaster-Recovery Manager		
-----------------------

0.1 What is SCDRM?
-----------------------------------------
It is a wrapper that acts as Ansible local agent and disaster recovery daemon for enterprise environments currently only for Red Hat Enterprise Linux.

Software stack:
 - AIDE: Intrusion detection system
 - Ansible: provided as Ansible roles/collection, intended as Ansible local agent
 - Git: change tracking and reverting
 - script/tlog: terminal session logging

It enforces secure change process, does security reporting, change tracking and management. 

Can coexist with existing change management systems like Puppet, CFEngine, Ansible...


0.2 Why would someone run it?
-----------------------------------------
Because people make mistakes!

Because managing N(N)k machines in a heavily segmented infrastructure with Ansible is fun and easy!



0.2.1 Answer this question! 
-----------------------------------------
If a junior sysadmin makes a typo in a playbook which got executed on ENTIRE inventory, and the typo resulted in /etc/passwd file getting deleted/overwritten, how long would it take you to recover from this?


How bad would it hurt?


0.3 How to use it?
-----------------------------------------
Get the project from Github or Ansible Galaxy.

Install and adjust settings to your needs - change configuration variables, update .gitignore and aide.conf.

By default your systems are actively set for disaster recovery, few most common scenarios.

Spin it up from 'dryrun' mode to active system critical configuration protection.

Embrace new change process into your environment by starting using 'adminmode/adminstart(-auto)' for all and any system changes, both interactive and automated.

Get daily reports and make unauthorized changes impossible.


0.4 Targeted audience
-----------------------------------------
Anyone looking for a free Ansible local agent.

Anyone interested in running a free local disaster recovery agent for RHEL and up the uptime.

Enterprise environments requiring hard and heavy security posture and compliance.

Security managers looking for a free compliance/reporting/audit tool.


0.5 What does it do / how does it work
-----------------------------------------
By default it will check for few most common man made disaster scenarios, like stopped network/SSH, missing default route, bad permissions, etc. When found it will revert to defaults, i.e. restart network/SSH or restore previously known default route.

When set with 'dryrun=0' it will actively protect /etc, /usr/lib/systemd/ and /usr/lib64/security, by using local Git instance for every directory. It uses Git to revert any unauthorized change.

It will record terminal user session for debugging/accounting purposes.

All changes are committed with local Git instances and AIDE update is ran after EVERY exit from 'adminmode'.

A global Git instance is taking care of commit metadata for every machine in your inventory, so the management node should always know about the current state in the infrastructure.

This makes possible to revert settings for entire infrastructure in a safe and automated manner or just hunt for any unauthorized changes.


Adminmode allows ONLY ONE user/script/playbook at a time to be doing changes to a machine. No more stepping over toes.


0.6 LICENCE
-----------------------------------------
GNU GPL v3 - see LICENSE for more details.


0.7 VIDEO DEMO - v1.4.7
-----------------------------------------
You can watch the demo at:

https://ibm.webex.com/ibm/ldr.php?RCID=370447dfd337cc651036af46684c0ae1

Enter with iPNngwP8
											

0.8 CONTRIBUTE
-----------------------------------------
There are many ways you could contribute to this project:


First, I would like to hear your disaster stories, that happened to you or someone you know.

One of my favorites is dropping the database on the wrong server, standalone with no backups of course.

What I am looking for are easy wins, stuff that can be easily checked and fixed. 

You can hardly protect against bad rm -rf without a proper backup...


Secondly, any good idea is taken into consideration, regarding approach, extensions and features or methodology.


Most importantly PLEASE do report all and any bugs, errors or weird stuff SCDRM might spit out.

Only you and your feedback can bring up SCDRM to a truly stable thing.

This has been tested working on RHEL 7/8/9 but still needs a proper test in Nk environment.

-----------------------------------------------------------------------------------------------------------------------------------------------------------




P.S. For those willing to read more, here we go in more detail.




1.0 INTRODUCTION
-----------------------------------------

While working with Nk RHEL infrastructure managed by CFEngine, which implementation was insufficient if you ask me, I've immediately started to write Ansible plays to deal with the infrastructure.

Been Ansible user for years and loving it, for multiple reasons.


Even after tunning the Ansible and playbooks to get the best performance, some things took longer than required. 

Control node used had 12 vCPUs running Ansible 2.9.x with 12 forks and 96 serial using 'free' strategy over ten gigabit ethernet. This was my best working setup.

For a big environmnt it is not always feasible to wait for a playbook running across Nk machines and reporting back. 

When there is no callback available due to heavy segmentation and security posture, you can rely only on SSH.


Puppet or CFEngine, for example, run local agents to maintain configuration of huge fleets of machines in short timeframe.

That is why a local agent for Ansible was written.


This has been implemented in a simple manner using local Git to monitor for changes. Currently works ONLY for: /etc /usr/lib/systemd /usr/lib64/security.

Git is fast, stable, lightweight and a standard.


Working in environment with high employee fluctuation brings in extra challenges.

Having rookies running playbooks across Nk machines can result in disaster. I have personaly witnessed this several times.

People can do mistakes because they are: stressed, tired, sick, shortsighted, fat fingered, distracted, in a hurry, ignorant, mean, out of luck...


When a 'mistake' happens on Nk machines, it is usually a DISASTER. When there is no fast disaster recovery process established, this can easily turn into 'DISASTER FROM HELL'.

In order to protect against these scenarios, a collection of known disaster sanity checks are performed.

The idea is to (at least) keep system reachable over SSH.

Recovering Nk machines over console is not something a sane person would enjoy nor is doable in meaningful timeframe.



1.1 APPROACH
-----------------------------------------

The idea is to check for changed/deleted critical system configuration files and revert.

Check for common connection problems and known disaster scenarios and try to recover/prevent.

Use AIDE for checking the rest of the system and send summary reporting to Security Management.

Enforce a standard change process, that is safe and fast to revert, transparent, traceable and easy to manage.

Keep up the uptime.



SCDRM does not care about your configuration posture or how many tasks you need to execute to get the 'end results'.

It will record the current state and monitor for changes upon which it acts accordingly.

Simple change revert can bring you system back to operational state.



2.0 DESCRIPTION
-----------------------------------------

So, what does SCDRM exactly do? In detail.


It's a stack made of few components:

	1. Human caused disaster recovery

	2. Change management for /etc, /usr/lib/systemd, /usr/lib64/security

	3. IDS reporting

	4. User terminal session logging

	5. Self protection


When installing by default SCDRM runs in DRYRUN, meaning it will not do any kind of reverting for files under /etc, /usr/lib/systemd and /usr/lib64/security. But it will do DR checks.

DR checks are done partially, every 5 minutes and every hour.

Checks done every 5 minutes (by default) will do DR checks and only when dryrun=0, revert any unauthorized changes and deletions under /etc, /usr/lib/systemd and /usr/lib64/security.

More intensive checks are done every hour - RPM verification.

AIDE check is done once a day starting at 20:00 randomizing start time up to 8 hours for the sake of infrastructure.

Actions are logged under /var/log/scdrm/ and a report file is updated after every check.

This report file can be used for example reporting with playbooks/reporting.yml.


In my case we had CFEngine taking control of configuration management, however not completely.

This left a 'loophole' for corner cases and caused several 'disaster' scenarios.

SCDRM implementation can co-exist with different change management systems.

Where CFEngine is managing only SOME files in /etc, SCDRM fills in the gap.

In combination with AIDE, no unauthorized change can pass under the radar.


Meaning you could use SCDRM along with any configuration management tools safely, without conflict.

This is by presuming your change management system writes a standard file header into EVERY managed file, 

which is how SCDRM determines which files to exclude.



3.0 INSTALLATION
-----------------------------------------

You should use the 'installation_wizard'.

It will setup some variable and invoke two plays:

	playbooks/scdrm.yml    (installs fleet of hosts)
	playbooks/adminssh.yml (installs only localhost)

Default install will set SCDRM in DRYRUN mode, meaning it will do only DR checks but will NOT revert any changes/deletions in given directories.

This allows you to run if for some time and collect enough intelligence about changes in your environment.

Then you would most probably want to update your aide.conf and .gitignore to reflect your environment needs.

Once updated 'DRYRUN=0' in /etc/scdrm/scdrm.conf you have activated change/deletion recovery for given directories for that machine(s).

To do this for you inventory you can simply execute:

	ansible-playbook playbooks/scdrm.yml -e "update=yes dryrun=0"
			or
	scdrm-config-update--dryrun-off
 
3.1 DEINSTALLATION
-----------------------------------------

It is a two part procedure. 

1. Remove SCDRM from your inventory:

        ansible-playbook playbooks/scdrm.yml -e "remove=yes"


2. Remove SCDRM and 'adminssh' from central management node.

        ansible-playbook playbooks/adminssh-remove.yml -e "myuser=$USER"


4.0 CHANGE PROCESS
-----------------------------------------

Your existing change process must adapt. Meet 'adminssh' and 'adminstart'.

Changes can be done via interactive shell or program/playbook/task.

Use 'adminssh' for any change related SSH session and interactive shell. 

For read only/investigation purpose, feel free to use 'defaul' ssh command.


When a script making a change is ran, it should invoke '/usr/local/sbin/adminstart-auto',

before making any changes to the system, and then after all works is done run:

'/usr/local/sbin/adminstop-auto' to commit the changes.

When connected as user, program '/usr/local/bin/adminstart' is started

Once done with the changes, exiting the shell (ctrl+d) will commit your changes, AND

before exiting, AIDE is updated to make sure all authorized changes are recorded for intrusion detection.

Finally, the Git commit IDs are stored in local host vars files for change tracking.



These will ensure the change process is respected and will safely close the changes to the system with Git and AIDE.

The two have been provided as roles and binaries you can easily include into your existing playbooks, i.e.:

    ---
    - name: Test SCDRM change process
      hosts: all
      gather_facts: False
      serial: 96
      roles:
        - role: adminstart
        - role: install_httpd
        - role: adminstop
    ...

Or you could call the binaries directly, i.e. 

    ---
    - name: Adminstart
      become: yes
      shell: /usr/local/sbin/adminstart-auto

    - name: Update system
      yum: 
        name: "*"
        update_only: yes
        state: latest

    - name: Adminstop
      become: yes
      shell: /usr/local/sbin/adminstop-auto
...

The only difference between 'adminstop/adminstart' and 'adminstart-auto/adminstop-auto' is user session logging.

Scripted/automated sessions are NOT logged, only live user sessions are.


LOCKING happens at 'adminstart' level.

This means that only ONE administrator is permitted to modify the system at a time.

Only one script/playbook can be making changes to the system at a time.

In case adminmode is active for more then 6 hours, it will be exited forcefully and all unsaved changes will get reverted (when NOT running 'DRYRUN' mode).


To test your SCDRM installation and restore process you can use example test.yml:

        ansible-playbook playbooks/test.yml

5.0 CHANGE TRACKING
-----------------------------------------
With Ansible you could be deploying and managing clusters and fleets of servers sharing common configuration.

You could be managing hundreds of stand-alone, configuration specific nodes.

For SCDRM it does not matter as it does not need to have any idea about your configuration postures.

What it cares about is the state or version of monitored directories.


Standard change process will update the local Git commit IDs locally as host variables for later comparison.


To collect the host vars run:

	ansible-playbook playbooks/change_tracking.yml -e "update=yes"
			or
	scdrm-update-change-tracking

To check for configuration version discrepancies run:

	ansible-playbook playbooks/change_tracking.yml -e "check=yes"
			or
	scdrm-check-change-tracking

To revert to previously recorded version run:

	ansible-playbook playbooks/change_tracking.yml -e "enforce=yes"
			or
	ansible-playbook playbooks/consistency-enforce.yml
			or
	scdrm-enforce-consistency


Bottom line, SCDRM takes care of configuration consistency in two levels:

 - local git revert mechanism per each machine

 - central and global git commit ID store

Local mechanism 'prevents' unauthorized changes per machine.

Global tracking mechanism also 'prevents' change process bypass/abuse, while depending on the local mechanism.

By having global tracking in 'personal' Git, we have records and possibility to revert to virtualy any environment version (presuming no broken dependencies).


Optionally, you could use your Gitlab/Github instance to keep host_vars in. The installation wizard will ask for this.

6.0 UPDATING CONFIGURATION
-----------------------------------------

Making changes to any SCDRM configuration file, should be done locally in the project templates/files/playbooks.

If variables you are looking for are not in the scdrm.yml, look for templates and update.yml task.

Then run this to provision the updates to your fleet:


	ansible-playbook playbooks/scdrm.yml -e "update=yes"
			or
	scdrm-config-update

Configuration update can be done inline - without making changes to any template/playbook.

This is my perferred and suggested method for testing. 

For production you want the values hardcoded into scdrm.yml for persistence reason.

Simply run the scdrm.yml playbook with 'update=yes' and add any variable you need to update, i.e.

	ansible-playbook playbooks/scdrm.yml -e "update=yes dryrun=0"
				or
	scdrm-config-update--dryrun-off


Here you can update the check frequency from default 5 minutes to 30 seconds:

	ansible-playbook playbooks/scdrm.yml -e "update=yes dryrun=0 check_freq=30"


To remove SCDRM from your inventory run:

	ansible-playbook playbooks/scdrm.yml -e "remove=yes"

7.0 SESSION LOGGING
-----------------------------------------

Terminal session logging is enabled only for user logins, not for scripted runs.

The logging is done locally in ~/.log_local directory for accounting, back-tracking and debugging purpose.

For RHEL7 'script' is used and for newer versions 'tlog'.

This will allow you to back-track and investigate in greater detail than before.


8.0 DISASTER RECOVERY
-----------------------------------------

Here comes a collection of real-life disaster cases that are easy to check and fix.

All of these I have witnessed personally or heard from a colleague.

Most of these were and are caused by human error.

When one simple thing like stopping SSH service on Nk machines happens, it is no fun at all.


Here I will list all active DR sanity checks currently performed:
 - ownership and permissions of /, /usr, /etc, 
 - is sshd active and unmasked
 - is (network|NetworkManger).service active and unmasked
 - if /etc/nologin or /var/run/nologin exist
 - rpm verify sshd, coreutils, bash, glibc
 - if default route is missing, restore it
 - any other modification to given directories that can cause system unreachability or malfunction

SCDRM will try to prevent accidental deletion of its files and git by making files immutable.

It will download rpms (glibc, rpm, coreutils) into /scdrm for different rescue scenarios.


When the SCDRM has been set fully and the change process respected, you can use fast consistency enforcing.

If the last recorded commit IDs differs from live state, SCDRM will revert to last recorded commit.


	ansible-playbook playbooks/consistency-enforce.yml
			or
	scdrm-enforce-consistency


Project also provides a simple revert playbook for fast config revert. You would use this with CAUTION.

To BLINDLY revert to the last change ONLY in /etc (by default) just run:


	ansible-playbook playbooks/revert-last_change.yml

This differes from 'consistency-enforce.yml' as it will not check last recorded commit.

It will blindly revert the last change no matter what it is and what it does.

This is useful only in certain scenarios and I would NOT recommend using this in production unless absolutely sure what you are doing.

USE WITH CAUTION.


To revert the whole infrastructure to an older version, you would do git revert in /scdrm/host_vars and then execute consistency-enforce.yml.

If you are lucky and no dependencies are broken, this should work just fine.

9.0 KNOWN LIMITATIONS
-----------------------------------------

 - Re-configuring exclusions in aide.conf and .gitignore is usually the first step to make as probably expectations won't be met using defaults.
 - Exiting admin mode will ALWAYS run AIDE update, which on some systems can take longer time. It also updates local vars.
 - In case you don't use default route on your systems, you will have no use of route-guard as it restores previously recorded default route when one found missing from the system. If you have multiple default routes, you could have PROBLEMS with route-guard!
 - Maybe you don't want your networking services automatically restarted when not active, as part of DR.
 - Maybe default RPM verification won't suit your case, test it.
 - Removal of SCDRM will remove AIDE and tlog from the system! You have been notified.
 - If you don't want to use AIDE - it can be turned off easily.
 - Automated revert possible only for dirs managed, currently ONLY /etc, /usr/lib/systemd/, /usr/lib64/security.
 - Race condition/piggy back commits - during admin mode is activated and another user is making changes to the system, they will be committed by user using 'adminstart'
 - Intended to be ran from single central management jumphost as it keeps the commit vars locally
 - Ansible strategy 'free' seems to break my 'consistency-enforce.yml' so works fine with 'linear' - have to look into this


10.0 FUTURE UPDATES
-----------------------------------------

This is just a beginning.

I have many other ideas I would like to implement here for a complete and robust

Security Change Disaster-Recovery Manager for Red hat Enterprise Linux, and beyond.

All updates will be pushed and published on Github and Ansible Galaxy accordingly.


After properly testing SCDRM on a larger infrastructure, I intend to extend support for Debian based systems.

Several other 'modules' are planned and should be released in the 2023, like scdrm-reboot (pre-reboot sanity checker).

Let there be uptime!


11.0 Important change log
-----------------------------------------
| Version |	Date   |	Description 	|
-----------------------------------------
1.7.5	    2022-11-18   Improvements and updates to scdrm_rc functions

1.7.4	    2022-11-18   Improvements and updates to reporting and usage

1.6.0	    2022-11-17   Added fast and easy modularity - AIDE and route-guard ON/OFF switch

1.5.1	    2022-11-17   Revert config - service restart fix for: ssh,sssd,firewalld,cron,udev,kdump,NetworkManger,journal; Handle LVM restores - ignore /etc/lvm/[archive,backup,cache,profile]. Now does proper service restart after revert change

1.5.0	    2022-11-16   For usage convenience added 'scdrm-' bash functions to ~/.bashrc

1.4.9	    2022-11-16   README updates, set tag and mark for pre_release

1.4.3-8	    2022-11-16   Bug fixes to route-guard, remove.yml, additions to aide.conf, bash-guard added, minor bug fixes

1.4.2	    2022-11-11   Bug fix, rsyslog restart added when changed

1.4.1	    2022-11-10   Sanity checks added: systemd verify, fstab/device checker, IO error check; bug fixes

1.3.1	    2022-11-10   Systemd reload after removing units

1.3.0	    2022-11-07   Added log_local rotation

1.2.16	    2022-11-02   Added separate aide.conf for SELinux, minor fixes

1.2.13 	    2022-10-31   Fixes and updates for working collection

1.2.12 	    2022-10-30   Fixes and updates for working collection

1.1.2 	    2022-10-29   Global host_vars are under Git protection, multiuser environment set

1.0.15      2022-10-28   SCDRM goes public

1.0.10      2022-10-20   Initial Gitlab commit


12.0 Thanks to
-----------------------------------------
My wife for the support!

IBM for the opportunity!

Red hat for great innovation and education!

Steffen Froemer@Redhat for motivation!

12.0 Author
-----------------------------------------
Currently working as Linux architect @ IBM Croatia.

You can contact me at kresimir.lovric@ibm.com,
Linkedin or Github.

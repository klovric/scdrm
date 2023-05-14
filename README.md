# SCDRM - Security Change Disaster-Recovery Manager		
-----------------------

What is SCDRM?
-----------------------------------------
It is a wrapper that acts as Ansible local agent and disaster recovery daemon designed for enterprise environments.

Software stack:
 - AIDE: Intrusion detection environment
 - Ansible: change management
 - Git: change tracking and reverting; Ansible local agent
 - script/tlog: terminal session logging

It enforces secure change process, does security reporting, change tracking and management, automated disaster recovery. 

Can work with existing change management systems like Puppet, Chef, CFEngine, Ansible...

Tested and working on:

	Manged nodes OS:
	RHEL: 7 - 9
	Debian: 10 - 11
	Ubuntu: 20.04 - 23.04

	Management node ENV:
	ansible [core 2.13.3]
  	python version = 3.9.13
  	jinja version = 3.1.2


Why would someone use it?
-----------------------------------------
Because people make mistakes!

Because managing N(N)k machines in a heavily segmented infrastructure with Ansible is fun and easy!


Answer this question! 
-----------------------------------------
If a junior sysadmin makes a typo in a playbook which got executed on ENTIRE inventory, and the typo resulted in /etc/passwd file getting deleted/overwritten, how long would it take you to recover from this?


How bad would it hurt?


How to use it?
-----------------------------------------
Get the project from Github or Ansible Galaxy.

Install and adjust settings to your needs - change configuration variables, update .gitignore and aide.conf.

By default your systems are actively set for disaster recovery, few most common scenarios.

Spin it up from 'dryrun' mode to active system critical configuration protection.

Embrace new change process into your environment by starting using 'adminmode/adminstart(-auto)' for all and any system changes, both interactive and automated.

Get daily reports and make unauthorized changes impossible.


Targeted audience
-----------------------------------------
Anyone looking for a free Ansible local agent.

Anyone interested in running a free local disaster recovery agent for Linux and keep up the uptime.

Enterprise environments requiring hard and heavy security posture and compliance.

Security managers looking for a free change management/reporting/audit tool.


What does it do / how does it work
-----------------------------------------
By default it will check for few most common man made disaster scenarios, like stopped network/SSH, missing default route, bad permissions, etc. When found it will revert to defaults, i.e. restart network/SSH or restore previously known default route.

When set with 'dryrun=0' it will actively protect (/etc, /usr/lib/systemd/, /usr/lib64/security) by using local Git instance for every directory. Git will revert any unauthorized change.

It will record terminal user session for debugging/accounting purposes.

All changes are committed with local Git instances and AIDE update is ran after EVERY exit from 'adminmode'.

A global Git instance is taking care of commit metadata for every machine in your inventory, so the management node should always know about the current state in the infrastructure.

This makes possible to revert settings for entire infrastructure in automated manner or just hunt for any unauthorized changes.


Admin mode allows ONLY ONE user/script/playbook at a time to be doing changes to a machine. No more stepping over toes.


LICENCE
-----------------------------------------
GNU GPL v3 - see LICENSE for more details.


VIDEO DEMO - Coming up
-----------------------------------------

Coming soon...
											

CONTRIBUTE
-----------------------------------------
There are many ways you could contribute to this project:


First, I would like to hear your disaster stories, that happened to you or someone you know.

One of my favorites is dropping the database on the wrong server, standalone with no backups of course.

What I am looking for are easy wins, stuff that can be easily checked and fixed. 

You can hardly protect against bad rm -rf without a proper backup...


Secondly, any good idea is taken into consideration, regarding approach, extensions and features or methodology.


Most importantly PLEASE do report all and any bugs, errors or weird stuff SCDRM might spit out.

Only you and your feedback can bring up SCDRM to a truly stable thing.

This has been tested working but still needs a proper test in Nk environment.

-----------------------------------------------------------------------------------------------------------------------------------------------------------




P.S. For those willing to read more, here we go in more detail.




INTRODUCTION
-----------------------------------------

While working with Nk RHEL infrastructure managed by CFEngine, I've immediately started to write Ansible plays to deal with the infrastructure.

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



APPROACH
-----------------------------------------

The idea is to check for changed/deleted critical system configuration files and revert.

Check for common connection problems and known disaster scenarios and try to recover/prevent.

Use AIDE for checking the rest of the system and send summary reporting to Security Management.

Enforce a standard change process, that is safe and fast to revert, transparent, traceable and easy to manage.

Keep up the uptime by trying to keep server reachable.



SCDRM does not care about your configuration posture or how many tasks you need to execute to get the 'end results'.

It will record the current state and monitor for changes upon which it acts accordingly.

Simple change revert can bring you system back to operational state.



DESCRIPTION
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



MODULARITY
-----------------------------------------

SCDRM is written with modularity in mind. This means that you can turn off almost any part of the stack to comply with your needs.

The following 'modules' or modes can be easily turned off/on:

	1. AIDE and its checking frequency
	2. Route-guard
	3. Change management exclusion (Ansible,Chef,Puppet...)
	4. Upstream Git
	5. Automatic configuration reverting (dryrun)
	6. Paranoid mode
	7. MOTD setup

Every important variable can be easily changed to create even the most complex setups.

You can always add a new directory to be watched by Git to the 'dirlist' variables in case you are not satisfied with the defaults (etc, systemd, pam).


INSTALLATION
-----------------------------------------

You should use the 'installation_wizard' for new setup. 

It will setup some variable and invoke two plays:

	playbooks/scdrm.yml    (installs fleet of hosts)
	playbooks/adminssh.yml (installs only localhost)


To install SCDRM to some new hosts add a host or a inventory file, i.e.:

	scdrm-install <hostname or inventory file>


Default install will set SCDRM in DRYRUN mode, meaning it will do only DR checks but will NOT revert any changes/deletions in given directories.

This allows you to run if for some time and collect enough intelligence about changes in your environment.

Then you would most probably want to update your aide.conf and .gitignore to reflect your environment needs.

Once updated 'DRYRUN=0' in /etc/scdrm/scdrm.conf you have activated change/deletion recovery for given directories for that machine(s).

To do this for you inventory you can simply execute:


	scdrm-config-update-dryrun-off

 
DEINSTALLATION
-----------------------------------------

Removes SCDRM from all inventory specified machines and localhost.


	scdrm-uninstall-all


If you want to remove SCDRM just from the inventory or some specific machines, use:


	scdrm-remove


CHANGE PROCESS
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
      roles:
        - role: adminstart
        - role: install_httpd
        - role: adminstop
    ...

Or you could call the binaries directly, i.e. 

    ---
    - name: Adminstart
      become: yes
      command: /usr/local/sbin/adminstart-auto

    - name: Update system
      yum: 
        name: "*"
        update_only: yes
        state: latest

    - name: Adminstop
      become: yes
      command: /usr/local/sbin/adminstop-auto
...

The only difference between 'adminstop/adminstart' and 'adminstart-auto/adminstop-auto' is user session logging.

Scripted/automated sessions are NOT logged, only live user sessions are.


LOCKING happens at 'adminstart' level.

This means that only ONE administrator is permitted to modify the system at a time.

Only one script/playbook can be making changes to the system at a time.

In case adminmode is active for more then 6 hours, it will be exited forcefully and all unsaved changes will get reverted (when NOT running 'DRYRUN' mode).


When the play is interrupted before 'adminstop' removes the lock files, you are left with 'locked' systems.

To remove all stale locks in such cases, just run:

scdrm-cleanup-stale-admin-locks


To test your SCDRM installation and restore process you can use example test.yml:

        ansible-playbook playbooks/test.yml

CHANGE TRACKING
-----------------------------------------
With Ansible you could be deploying and managing clusters and fleets of servers sharing common configuration.

You could be managing hundreds of stand-alone, configuration specific nodes.

For SCDRM it does not matter as it does not need to have any idea about your configuration postures.

What it cares about is the state or version of monitored directories.


Standard change process will update the local Git commit IDs locally as host variables for later comparison.


To collect the host vars run:


	scdrm-update-change-tracking


To check for configuration version discrepancies run:


	scdrm-check-change-tracking


To revert to previously recorded version run:


	scdrm-enforce-consistency


Bottom line, SCDRM takes care of configuration consistency in two levels:

 - local git revert mechanism per each machine

 - central and global git commit ID store

Local mechanism 'prevents' unauthorized changes per machine.

Global tracking mechanism also 'prevents' change process bypass/abuse, while depending on the local mechanism.

By having global tracking in 'personal' Git, we have records and possibility to revert to virtualy any environment version (presuming no broken dependencies).


Optionally, you could use your Gitlab/Github instance to keep host_vars in. The installation wizard will ask for this.


UPDATING CONFIGURATION
-----------------------------------------

Making changes to any SCDRM configuration file, should be done locally in the project templates/files/playbooks.

If variables you are looking for are not in the scdrm.yml, look for templates and update.yml task.

Then run this to provision the updates to your fleet:


	scdrm-config-update


For production you want the values hardcoded into scdrm.yml for persistence reason.

Simply run the scdrm.yml playbook with 'update=yes' and add any variable you need to update, i.e.

Some configuration changes do not require re-syncing the stack, thus they have been picked out

for convenience and fast usage, like:


	scdrm-config-update-dryrun-off - exit dryrun mode for active protection

	scdrm-config-update-paranoid-on - enter paranoid mode to presume every change is to be reverted

	scdrm-paranoid-acknowledge-change - stop change from being reverted / acknowlede as valid


MODULES 
----------------------------------------

Following options are made modular for ease of use. They are usually set in playbooks/vars/main.yml and playbooks/scdrm.yml.

  - AIDE: configure and use on demand; more frequent checks as secondary option
  - Route-guard: in case you are not using default gateways (or have multiple), this should be turned off
  - Active configuration protection by Git: when turned off is called 'DRYRUN' mode
  - Change management exclusion: exclude files containing specific header (like 'Managed by Ansible')
  - Upstream Git: sync host_vars to upstream Git instance
  - Paranoid mode: revert change if not acknowledged by human, presume the change is bad - good for testing SSH/PAM changes


SESSION LOGGING
----------------------------------------

Terminal session logging is enabled only for user logins, not for scripted runs.

The logging is done locally in ~/.log_local directory for accounting, back-tracking and debugging purpose.

For RHEL7 'script' is used and for newer versions 'tlog'.

This will allow you to back-track and investigate in greater detail than before.


DISASTER RECOVERY
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
 - verify sshd, coreutils, bash, glibc, rpm/dpkg
 - make sure default route is up and reachable
 - verify systemd units
 - filesystem sanity check

SCDRM will try to prevent accidental deletion of its files and git by making files immutable.

It will download rpms (glibc, rpm, coreutils) into /scdrm for different rescue scenarios.


When the SCDRM has been set fully and the change process respected, you can use fast consistency enforcing.

If the last recorded commit IDs differs from live state, SCDRM will revert to last recorded commit.


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

KNOWN LIMITATIONS
-----------------------------------------

 - Re-configuring exclusions in aide.conf and .gitignore is usually the first step to make as probably expectations won't be met using defaults.
 - Exiting admin mode run AIDE update (when set so), which on some systems can take longer time. It also updates local vars.
 - In case you don't use default route on your systems, you will have no use of route-guard as it restores previously recorded default route when one found missing from the system. If you have multiple default routes, you could have PROBLEMS with route-guard! You can easily disable route-guard.
 - Maybe you don't want your networking services automatically restarted when not active, as part of DR.
 - Maybe default package verification won't suit your case, test it.
 - Removal of SCDRM will remove AIDE and tlog from the system! You have been notified.
 - Automated revert possible only for dirs managed, currently ONLY /etc, /usr/lib/systemd/, /usr/lib64/security.
 - Race condition/piggy back commits - during admin mode is activated and another user is making changes to the system, they will be committed by user using 'admin mode'
 - Intended to be ran from single central management jumphost as it keeps the commit vars locally
 - Ansible strategy 'free' seems to break my 'consistency-enforce.yml' so works fine with 'linear' - have to look into this


FUTURE UPDATES
-----------------------------------------

This is just a beginning.

I have many other ideas I would like to implement here for a complete and robust

Security Change Disaster-Recovery Manager for Red hat Enterprise Linux, and beyond.

All updates will be pushed and published on Github and Ansible Galaxy accordingly.


After properly testing SCDRM on a larger infrastructure, I intend to extend support for Debian based systems.

Several other 'modules' are planned and should be released in the 2023, like scdrm-reboot (pre-reboot sanity checker).

Let there be uptime!


Thanks to
-----------------------------------------
My wife for the support!

IBM for the opportunity!

Red hat for great products and education!

Steffen Froemer@Redhat for motivation!

Author
-----------------------------------------
Kresimir Lovric

You can contact me at klovric@kripteia.eu
Linkedin or Github.


Important change log
-----------------------------------------
| Version |	Date   |	Description 	|
-----------------------------------------
1.9.4	    2023-05-14   Introducing auto 'scdrm.conf' DR backup and restore, some minor fixes and improvements

1.9.3	    2023-03-10   Included unpingable gateway setups in route-guard:v2 ; PARANOID mode included ; MOTD included ; several minor fixes

1.8.12	    2023-03-06   Proper handling of ${dirlist} fixed ; Community RFC marked with v1.8.10

1.8.10	    2023-03-05   Bug fix to installation_wizard and config update task

1.8.9	    2023-03-01   Route-guard V2 implemented

1.8.6	    2023-01-23   Multiple fixes for Debian support, minor improvements.

1.8.3	    2023-01-21   Extended support for Debian/Ubuntu; modular AIDE auto-check feature added; minor fixes

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

# RadiUID ![RadiUID][logo]

An application to extract User-to-IP mappings from RADIUS accounting data and send them to Palo Alto firewalls for use by the User-ID function.



####   VERSION   ####
-----------------------------------------
The version of RadiUID documented here is: **v2.3.0**


####   TABLE OF CONTENTS   ####
-----------------------------------------
1. [What is RadiUID?](#what-is-radiuid)
2. [How it Works](#how-it-works)
3. [Screenshots](#screenshots)
4. [Requirements](#requirements)
5. [Tested Environments](#tested-environments)
6. [Docker Install Instructions](#docker-install-instructions)
7. [OS Install Instructions](#os-install-instructions)
8. [Command Interface](#command-interface)
9. [Timeout Tuning](#timeout-tuning)
10. [The Munge Engine](#the-munge-engine)
11. [1.1.0 TO 2.0.0 Updates](#updates-in-v110----v200)
12. [2.0.0 TO 2.0.1 Updates](#updates-in-v200----v201)
13. [2.0.1 TO 2.1.0 Updates](#updates-in-v201----v210)
14. [2.1.0 TO 2.2.0 Updates](#updates-in-v210----v220)
15. [2.2.0 TO 2.2.1 Updates](#updates-in-v220----v221)
16. [2.2.1 TO 2.3.0 Updates](#updates-in-v221----v230)
17. [Upgrade Processes](#upgrade-processes)
18. [Docker Files](#dockerfiles)
19. [Contributing](#contributing)


####   WHAT IS RADIUID   ####
-----------------------------------------

User-based firewall filtering is a novel and attractive concept which can often be difficult to implement due to the requirement by firewalls to map IP addresses to users. One common method of getting user-to-IP mapping information for your firewall is to install a log-reading agent onto an Active Directory domain controller which can look over transaction logs and send the proper information to the firewall, but this assumes user endpoints interact and authenticate directly with the domain controllers, or that you have Active Directory at all!

RadiUID is a Linux-based application built to take everyday RADIUS accounting information generated by RADIUS authenticators like wireless systems, firewalls, etc (which contains username and IP info) and send that ephemeral IP and username mapping info to a Palo Alto firewall to be used by the User-ID system for user or group-based access-list filtering, or intelligent reporting. 



####   HOW IT WORKS   ####
--------------------------------------

RadiUID uses FreeRADIUS as a backend service to listen on RADIUS accounting ports (typically TCP\UDP 1813) and write recieved accounting information to accounting logs.

RadiUID then parses these logs, pulls down the User and IP mapping information and pushes those mappings to the Palo Alto firewall using the published RESTful XML API.

RadiUID runs as a system service on Linux and is very easy to configure and use. All configuration and interaction with RadiUID is via command line on the Linux BASH shell. Once the installer completes, RadiUID can be invoked from the command shell by typing `radiuid` followed by the desired command. Hit the [TAB] key for command options or hit [ENTER] for the list of options!



####   SCREENSHOTS   ####
--------------------------------------

**The main list of CLI command options**
![RadiUID][all-args]

**Output from the `show log` command**
![RadiUID][show-log]

**Output from the `show config` command**
![RadiUID][show-config]

**Output from the `show config set` command**
![RadiUID][show-config-set]

**Pushing a mapping using the `push` command and checking the current mappings using the `show mappings` command**
![RadiUID][push-and-show]

**Test your munge rule-set using the `request munge-test` command**
![RadiUID][radiuid-munge-test]



####   REQUIREMENTS   ####
--------------------------------------

OS:			**Any modern Debian, RHEL distro (CentOS 6 or 7, Ubuntu 14 or 16...), or Docker container host**

Interpreter:		**Python 2.7.X** *(Also works on Python 2.6.6 and up)*

PAN-OS Version:		**6.X and 7.X**



####   TESTED ENVIRONMENTS   ####
--------------------------------------

RadiUID has been written and tested in a few environments to date as it was purpose-built for a specific environment, but it should be very adaptable as it uses standardized RADIUS accounting to source user information and the published API to push that info to Palo Alto firewalls.

RadiUID has currently been tested with the following Operating Systems, RADIUS servers, and authenticators

Operating Systems: **CentOS 7, CentOS 6, Ubuntu 16 Server, Ubuntu 14 Server, Docker 1.10.3**

Identity Systems: **JumpCloud RADIUS service, Windows 2012 NPS Server (with Active Directory)**

Authenticators: **Meraki Wireless Access Points, Cisco Wireless (Controller-based), Ruckus Zonedirector**



####   DOCKER INSTALL INSTRUCTIONS   ####
----------------------------------------------

Downloading and running RadiUID on a Docker host is the fastest and easiest way to get it up and running. There are two versions of the RadiUID image maintained on Docker Hub: an image **with SSH**, and an image **without SSH**. The image **with SSH** has the SSH server installed and pre-configured with a login username and password. All you have to do is change the password. The Dockerfile build scripts which were used to build the images are available in the [Docker Files](#dockerfiles) section in case you want to perform the build yourself. 

1. From the Docker host, download and run the image in interactive mode
	1. To run the image **with SSH**: `docker run -it -p 1813:1813/udp -p 1813:1813/tcp -p 222:22/tcp --name radiuid -t packetsar/radiuid-ssh:latest`
	2. To run the image **without SSH**: `docker run -it -p 1813:1813/udp -p 1813:1813/tcp --name RADIUID -t packetsar/radiuid:latest`
2. *If you ran the image with SSH:* The default SSH username and password is root\radiuid. Run the command `passwd root` to change the SSH password.
3. Run the command `radiuid show config set` to see the default configuration.
4. Run the `radiuid clear target all` command to delete the firewall target configurations, then use the `radiuid set target [parameters]` command to configure the application with your Palo Alto target firewall paramaters.
5. Run the `radiuid set client [parameters]` command to configure FreeRADIUS to accept RADIUS accounting data from your RADIUS authenticators.
6. Once configuration is complete, run the `radiuid service all restart` to restart the services so the new configuration takes effect.
7. Take a look at your logs using the `radiuid show log` command to see what the application is doing.




####   OS INSTALL INSTRUCTIONS   ####
----------------------------------------------

The install of RadiUID is very quick and straightforward using the built-in installer.
NOTE: You need to be logged in as root or have sudo privileges on the system to install RadiUID


1. Install OS with appropriate IP and OS settings and update to latest patches (recommended)
	- Check out the [CentOS Minimal Server - Post-Install Setup][centos-post-install] and the [Ubuntu Server - Post Install Setup][ubuntu-post-install] for help with some of the post-OS-install configuration steps.
2. Install the Git client (unless you already have the RadiUID files): `sudo yum install git -y` or `sudo apt install git -y`
3. Clone the RadiUID repo to any location on the box: `git clone https://github.com/PackeTsar/radiuid.git`
4. Change to the directory where the RadiUID main code file (radiuid.py) and config file (radiuid.conf) are stored: `cd radiuid`
	- (OPTIONAL) Change to the development branch (perform this step only if you are prepared for a version which is under active development and may have broken features): `git checkout devX.X.X`
5. Run the RadiUID program in install mode to perform the installation: `sudo python radiuid.py install`
	- NOTE: Make sure that you have the .conf file in the same directory as the .py directory for the initial install
6. Follow the on-screen prompts to install FreeRADIUS and the RadiUID application
	- The installer should let you know if everything installed correctly and services are running, but in the next section are the CLI commands you can run to check up on it.



####   COMMAND INTERFACE   ####
----------------------------------------------

The RadiUID system is meant to run in the background as a system service: constantly checking for new RADIUS accounting data and pushing User-ID mapping information to the firewall, but it also has an easy to use command interface. This command interface is meant to be used for regular maintenance, troubleshooting, and operation of the system.

Below is the CLI guide for the RadiUID service.

*You can see this guide by typing 'python radiuid.py' (before installation) or 'radiuid' (after installation) and hitting ENTER.*

     -------------------------------------------------------------------------------------------------------------------------------
                          ARGUMENTS                    |                                  DESCRIPTIONS
     -------------------------------------------------------------------------------------------------------------------------------
     
      - run                                            |  Run the RadiUID main program in shell mode begin pushing User-ID information
     -------------------------------------------------------------------------------------------------------------------------------
     
      - install                                        |  Run RadiUID Install/Maintenance Utility
     -------------------------------------------------------------------------------------------------------------------------------
     
      - show log                                       |  Show the RadiUID log file
      - show acct-logs                                 |  Show the log files currently in the FreeRADIUS accounting directory
      - show run (xml | set)                           |  Show the RadiUID configuration in XML format (default) or as set commands
      - show config (xml | set)                        |  Show the RadiUID configuration in XML format (default) or as set commands
      - show clients (file | table)                    |  Show the FreeRADIUS clients and config file
      - show status                                    |  Show the RadiUID and FreeRADIUS service statuses
      - show mappings (<target> | all | consistency)   |  Show the current IP-to-User mappings of one or all targets or check consistency
     -------------------------------------------------------------------------------------------------------------------------------
     
      - set logfile                                    |  Set the RadiUID logfile path
      - set radiuslogpath                              |  Set the path used to find FreeRADIUS accounting log files
      - set maxloglines <number-of-lines>              |  Set the max number of lines allowed in the log ('0' turns circular logging off)
      - set userdomain (none | <domain name>)          |  Set the domain name prepended to User-ID mappings
      - set timeout                                    |  Set the timeout (in minutes) for User-ID mappings sent to the firewall targets
      - set client <ip-block> <shared-secret>          |  Set configuration elements for RADIUS clients to send accounting data FreeRADIUS
      - set munge <rule>.<step> [parameters]           |  Set munge (string processing rules) for User-IDs
      - set target <hostname>:<vsys-id> [parameters]   |  Set configuration elements for existing or new firewall targets
     -------------------------------------------------------------------------------------------------------------------------------
     
      - push (<hostname>:<vsys-id> | all) [parameters] |  Manually push a User-ID mapping to one or all firewall targets
     -------------------------------------------------------------------------------------------------------------------------------
     
      - tail log (<# of lines>)                        |  Watch the RadiUID log file in real time
     -------------------------------------------------------------------------------------------------------------------------------
     
      - clear log                                      |  Delete the content in the log file
      - clear acct-logs                                |  Delete the log files currently in the FreeRADIUS accounting directory
      - clear client (<ip-block> | all)                |  Delete one or all RADIUS client IP blocks in FreeRADIUS config file
      - clear munge (<rule> | all) (<step> | all)      |  Delete one or all munge rules in the config file
      - clear target (<hostname>:<vsys-id> | all)      |  Delete one or all firewall targets in the config file
      - clear mappings [parameters]                    |  Remove one or all IP-to-User mappings from one or all firewalls
     -------------------------------------------------------------------------------------------------------------------------------
     
      - edit config                                    |  Edit the RadiUID config file
      - edit clients                                   |  Edit RADIUS client config file for FreeRADIUS
     -------------------------------------------------------------------------------------------------------------------------------
     
      - service [parameters]                           |  Control the RadiUID and FreeRADIUS system services
     -------------------------------------------------------------------------------------------------------------------------------
     
      - request [parameters]                           |  Make system-level changes for RadiUID service
     -------------------------------------------------------------------------------------------------------------------------------
     
      - version                                        |  Show the current version of RadiUID and FreeRADIUS
     -------------------------------------------------------------------------------------------------------------------------------



####   TIMEOUT TUNING   ####
----------------------------------------------

RadiUID pushes ephemeral User-ID information to the firewall whenever new RADIUS accounting information is recieved and by default sets a timeout of 60 minutes. If this accounting information comes from a wireless system (where most devices re-authenticate regularly) then you may be able to tune down that timeout to make the mapping information expire more quickly. If the RADIUS authenticator is something like a VPN concentrator (where re-authentication doesn't typically happen), then you may want to turn up the timeout period. Either way, you should expect to have to play with the timeout settings to make sure your firewalls are not prematurely expiring User-ID data from their mapping tables.



####   THE MUNGE ENGINE   ####
----------------------------------------------
The Munge Engine is a rule-based string processor which is used in RadiUID to filter and process User-IDs based on rules you configure. The munge feature was introduced in version 2.2.0.

- A sample munge configuration can be seen below. This configuration will instruct the Munge Engine to find any User-ID which contains a double-backslash and reconstruct it with only one backslash. Then it will find any User-ID which contains the name 'vendor' and discard it (prevent it from being pushed to the Palo Alto). This example uses both of the Munge Engine complex actions (`set-variable`, and `assemble`), but only uses one of the simple actions (`discard`), it does not use the `accept` simple action.

    *NOTE: The double-backslash in the `101.0 match` statement is represented by a quad-backslash because BASH recognizes the backslash character as an escape. You will always need to use a double-backslash to represent a single-backslash. You also should always wrap your regular expressions in quotes when entering them.*
	- `radiuid set munge 101.0 match "\\\\" partial`
	- `radiuid set munge 101.10 set-variable domain from-match "^[a-zA-Z0-9]+"`
	- `radiuid set munge 101.20 set-variable user from-match "[a-zA-Z0-9]+$"`
	- `radiuid set munge 101.30 set-variable slash from-string "\\"`
	- `radiuid set munge 101.40 assemble domain slash user`
	- `radiuid set munge 102.0 match "vendor" partial`
    - `radiuid set munge 102.10 discard`

- Munge rules are broken down into rules and steps and are configured/ordered in a dot-notation as `<rule number>.<step number>`. The rules and steps can be numbered as desired with one exception (`X.0`) which is described below. The rules and steps are processed in order by their numbers, so when configuring them, you may want to leave gaps in the assigned numbers for insertion of other rules or steps later between the existing ones.
- The only requirement for rule numbering is that step '0' in each rule (X.0) must be a match statement, as it is used to determine whether or not to process the rule on the User-ID. All rules must begin with a `X.0 match` statement followed by either an `any` keyword (which will match all inputs) or by a regular expression.
	- If the `X.0 match` statement uses a regular expression, it will require a `complete` or `partial` keyword at the end which is used as the return criterion. The `complete` keyword requires that the regular expression match and return the entire input User-ID. The `partial` keyword activates the rule upon a partial return of the input User-ID from the regular expression match operation.
- Other than the required `X.0 match` action, Munge has four actions which are broken down into two groups:
	- Simple Actions: `accept`, `discard`
	- Complex Actions: `set-variable`, `assemble`
- The various actions are explained below
	- The `accept` action halts all rule and step processing and passes the input (User-ID) back out of the engine without any further filtering or changes.
	- The `discard` action halts all rule and step processing and discards the current input; not allowing it to pass out of the engine at all.
	- The `set-variable` action instructs the engine to save a string (either part of the input/User-ID, or a manually configured static string) in memory for use later by the assemble action. The variable's name is configured right after the `set-variable` term and it can be any alphanumerical word. The string's source-type can be either a regular expression match (using the `from-match` term) or it can be a statically configured string (using the `from-string` term).
		- *NOTE: Set variables are usable across rules. They do not have to be used by the assemble action within the same rule. If you reuse the same variable name in different rule, note that the variable value set in an earlier rule will be overwritten by the later rule.*
	- The `assemble` action is used to assemble previously set variables into one string. A list of strings should be provided in order after the `assemble` verb seperated by spaces.
- The `request munge-test` command can be used to test a Munge rule-set on a provided input. You can also provide the `debug` term at the end of the command to see a walk-through of the steps taken by the Munge Engine and how it processed the configured rules to modify/filter the input provided in the command.
- The `radiuid push` command has the new keyword `bypass-munge` available at the end to either let the Munge Engine process the input User-ID (by default) or bypass the Munge Engine and push only the input User-ID.



####   UPDATES IN V1.1.0 --> V2.0.0   ####
--------------------------------------

**ADDED FEATURES:**

- The RadiUID config file has been changed to a simpler XML format. Config file management no longer depends on the ConfigParser module.

- All configuration settings (including the RADIUS client configuration for FreeRADIUS) are configurable using `set` commands. Just type `radiuid set` and hit [ENTER] to see the options or type `show config set` and hit [ENTER] to see the current configuration as a series of `set` commands.

- Multiple target firewalls are now supported; mappings can be pushed to multiple firewalls using different credentials.

- Multi-vsys functionality has been added so a configured firewall target includes parameters for the target vsys. If you want to control multiple vsys on the same firewall, you will need to add multiple targets.

- Improved HTTP error handling to keep application from crashing.

- Added CLI auto-complete functionality to allow you to use the [TAB] key to automatically complete commands or see the available options.

- Circular logging was added to maintain the size of the log file. The number of lines allowed in the log file is controlled by the *maxloglines* parameter which is configurable using the `set maxloglines` command.

- The `show mappings` was command added to pull and view mappings directly from one or all firewalls. The `consistency` parameter can also be used to check the consistency of mappings across all configured firewalls.

- The `push` command was added to allow you to manually push a User-to-IP mapping to one or all the firewalls. *NOTE: The user and IP address can be anything you want, they do not have to be legitimate users or working IP addresses*

- The `show config set` command was added to display the current configuration as a series of `set` commands which can be copied and pasted to configure the application.

- The `show clients`, `set client`, and `clear client` commands were added to allow you to more easily control the RADIUS clients configured in the FreeRADIUS clients.conf file. Now it can all be administered using RadiUID commands. The `show config set` output even includes the current RADIUS clients as `set client` commands.



####   UPDATES IN V2.0.0 --> V2.0.1   ####
--------------------------------------

**BUG FIXES:**

- *ISSUE #13*: The RadiUID 'merge_dicts' method was throwing a `KeyError` exception and quitting the loop (service) when a FreeRADIUS log was scraped which didn't contain the three required fields (`usernameterm`, `ipaddressterm`, and the `delineatorterm`). An error handler has been added to detect the `KeyError`, dump the dictionary data to the log, and continue in the loop.
    - This issue was reproduced on v2.0.0 code by removing the line in the FreeRADIUS log containing the `usernameterm` text and running the loop (`radiuid run`).
    - It is highly recommended to update to v2.0.1 or later to fix this bug as it affects the stability of the RadiUID RADIUS log capturing functionality.

- *ISSUE #14*: The default configured `delineatorterm` was documented as non-functional for RADIUS accounting messages from a Ruckus wireless system by Dan Hume on his blog at http://www.dhume.co.uk. He had to change the default `delineatorterm` to "Accounting-Session-ID" to make the log parsing work properly.
    - This bug has been fixed by adding functionality for RadiUID to recognize the paragraph separations between FreeRADIUS log entries within the same file and use those paragraph separations as the delineator. To enable this functionality, you must upgrade to v2.0.1 and use the [PARAGRAPH] keyword as the `delineatorterm` value (which is now the default value in the config file).



####   UPDATES IN V2.0.1 --> V2.1.0   ####
--------------------------------------

**ADDED FEATURES:**

- Multi-OS support: Previously only CentOS 7 was supported for installation due to dependencies on specific file paths and OS commands for interaction with the OS. Now RadiUID should fully work on any modern Debian or RHEL distro. It has been QA tested on CentOS 7.2, CentOS 6.8, Ubuntu 16.04, and Ubuntu 14.04.

- `radiuid show acct-logs` and `radiuid clear acct-logs` commands now added to help with controlling the FreeRADIUS logs in the accounting directory.

- The new `request` top-level command now gives access to some of the more advanced system-level functions in RadiUID. These commands can now be used to fully install RadiUID as an alternative to the Install/Maintenance Utility
    - `request auto-complete` runs the script to install and activate the BASH Auto-Completion feature
    - `request freeradius-install` installs the FreeRADIUS app using the proper package manager
    - `request xml-update` downloads and installs an update to the xml.etree.ElementTree Python module. This upgrades ElementTree to 1.3.0 which is the minimum version required for RadiUID to run properly. This is only required when running Python 2.6.X.
    - `request reinstall keep-config` performs a reinstall of the RadiUID binary, BASH Auto-Completion feature, RadiUID service, and restarts the service, but leaves the current config file alone to preserve the config during an upgrade. This command is now the preferred way to perform an update of an existing installation of RadiUID.
    - `request reinstall replace-config`performs all of the tasks listed above with the exception that it also replaces the existing configuration (if one exists) with the default config. This command can be used to do a quick net-new install of RadiUID without using the classic installer. 

**BUG FIXES:**

- *ISSUE #16*: Newer builds of urllib2 in Python 2.7 started generating an error when the SSL certificate was invalid. Some logic has been added to detect when to use the proper SSL handling

- *ISSUE #17*: The RadiUID XML assembler was hard set to allow up to 100 UIDs in a single API call before splitting into multiple calls which was overwhelming to the Palo Alto. That setting has been moved down to 50 UIDs per call and the setting `maxuidspercall` has been moved to the internal settings area near the top of the radiuid.py file.
    - This fix was reproduced and verified fixed by having RadiUID eat a FreeRADIUS Accounting log file with 2000 UIDs and push them into a test firewall. 50 UIDs per API call seems to work even with long usernames (50 characters).



####   UPDATES IN V2.1.0 --> V2.2.0   ####
--------------------------------------

**ADDED FEATURES:**

- The Munge Engine: RadiUID now includes a built-in rule-based string processor. The Munge Engine allows users to create a list of rules which will be used by RadiUID to filter, dissect, and reassemble User-IDs as they pass through the RadiUID service. More details on this feature can be found in the [Munge Engine](#the-munge-engine) section.

**BUG FIXES:**

- *ISSUE #19*: The `userdomain` configuration element now allows the use of the value `none` to specify that no domain should be prepended to User-IDs.



####   UPDATES IN V2.2.0 --> V2.2.1   ####
--------------------------------------

**BUG FIXES:**

- *ISSUE #20*: Added the `no-confirm` switch to the end of the commands `request reinstall keep-config`, `request reinstall replace-config`, `request freeradius-install` commands.



####   UPDATES IN V2.2.1 --> V2.3.0   ####
--------------------------------------

**ADDED FEATURES:**

- RadiUID is now available as a Docker image on [Docker Hub][docker-hub]. Small code changes were made to allow RadiUID to recognize when it is being run in a container and to be able to stop, start, and restart services while the container is running. 




####   UPGRADE PROCESSES   ####
--------------------------------------

**Upgrading from v2.X to v2.3.0:**

1. Perform a `radiuid show config set` command and save the `set` commands displayed in a safe place (just in case)
2. Download the v2.2.0 code from the GitHub repo by using `git clone https://github.com/PackeTsar/radiuid.git`
    - If the "radiuid" folder already exists, you may want to use git to update the clone `cd radiuid/; git pull`
3. Move to the radiuid folder created by git using the `cd radiuid/` command
4. Perform a quick reinstall/update of RadiUID using the command `python radiuid.py request reinstall keep-config`
5. Type in CONFIRM and hit ENTER to confirm you want to perform the reinstall
6. Once the installer exits, you should run `radiuid show config set` and see your configuration from before.
7. Perform a `radiuid service all restart` command to restart RadiUID to use the new app version
	- *NOTE: The RadiUID service will continue running in the background throughout the install process. It is not until you restart/stop the service that the new version and configuration will take effect.* 
8. You may also want to log out of the shell and back in to activate any new auto-complete functions.


**Upgrading from v1.X to v2.3.0:**

1. Change the name of your config file (/etc/radiuid/radiuid.conf) by issuing the command `mv /etc/radiuid/radiuid.conf /etc/radiuid/radiuid.conf.backup`
2. Grab the contents to have them handy during the install of the new version `more /etc/radiuid/radiuid.conf.backup`
3. Download the v2.X.X code from the GitHub repo by using `git clone https://github.com/PackeTsar/radiuid.git`
    - If the "radiuid" folder already exists, you may want to use git to update the clone `cd radiuid/; git pull`
4. Move to the radiuid folder created by git using the `cd radiuid/` command
5. Perform a full install of RadiUID using the command `python radiuid.py install`
6. Follow the prompts and fill out the appropriate information using the information from the old configuration file
7. Once the installer exits, you should run `radiuid show config set` and see your configuration.
8. Perform a `radiuid service all restart` command to restart RadiUID to use the new app version



####   DOCKERFILES   ####
--------------------------------------

These are the dockerfile script files used to build the SSH and non-SSH Docker images hosted on [Docker Hub][docker-hub]. You can use these on a Docker host to build your own RadiUID image instead of downloading the pre-made one from Docker Hub.

**With SSH**

    FROM centos:latest
    MAINTAINER John W Kerns "jkerns@packetsar.com"
    
    ### Install and configure SSH Server for SSH access to container ###
    RUN yum install -y openssh openssh-server openssh-clients sudo passwd
    RUN sshd-keygen
    RUN sed -i "s/UsePAM.*/UsePAM no/g" /etc/ssh/sshd_config
    RUN sed -i "s/#UsePrivilegeSeparation.*/UsePrivilegeSeparation no/g" /etc/ssh/sshd_config
    RUN useradd admin -G wheel -s /bin/bash -m
    RUN echo 'root:radiuid' | chpasswd
    RUN echo '%wheel ALL=(ALL) ALL' >> /etc/sudoers
    
    ### Download and install RadiUID from latest release ###
    RUN curl -sL https://codeload.github.com/PackeTsar/radiuid/tar.gz/2.2.1 | tar xz
    RUN cd radiuid-2.2.1;python radiuid.py request reinstall replace-config no-confirm
    RUN cd radiuid-2.2.1;python radiuid.py request freeradius-install no-confirm
    
    ### Expose ports and provide run commands ###
    EXPOSE 1813/udp
    EXPOSE 1813/tcp
    EXPOSE 22/tcp
    CMD radiusd & radiuid run >> /etc/radiuid/STDOUT & /usr/sbin/sshd >> /etc/radiuid/SSH-STDOUT & /bin/bash

**Without SSH**

    FROM centos:latest
    MAINTAINER John W Kerns "jkerns@packetsar.com"
    
    ### Download and install RadiUID from latest release ###
    RUN curl -sL https://codeload.github.com/PackeTsar/radiuid/tar.gz/2.2.1 | tar xz
    RUN cd radiuid-2.2.1;python radiuid.py request reinstall replace-config no-confirm
    RUN cd radiuid-2.2.1;python radiuid.py request freeradius-install no-confirm
    
    ### Expose ports and provide run commands ###
    EXPOSE 1813/udp
    EXPOSE 1813/tcp
    CMD radiusd & radiuid run >> /etc/radiuid/STDOUT & /bin/bash




####   CONTRIBUTING   ####
--------------------------------------

If you would like to help out by contributing code or reporting issues, please do!

Visit the GitHub page (https://github.com/PackeTsar/radiuid) and either report an issue or fork the project, commit some changes, and submit a pull request.

[logo]: http://www.packetsar.com/wp-content/uploads/radiuid-logo-tiny-100.png
[all-args]: http://www.packetsar.com/wp-content/uploads/radiuid-all-args.png
[show-log]: http://www.packetsar.com/wp-content/uploads/radiuid-show-log.png
[docker-hub]: https://hub.docker.com/r/packetsar/
[show-config]: http://www.packetsar.com/wp-content/uploads/radiuid-show-config.png
[show-config-set]: http://www.packetsar.com/wp-content/uploads/radiuid-show-config-set.png
[push-and-show]: http://www.packetsar.com/wp-content/uploads/radiuid-push-and-show.png
[radiuid-munge-test]: http://www.packetsar.com/wp-content/uploads/radiuid-munge-test.png
[centos-post-install]: https://github.com/PackeTsar/scriptfury/blob/master/CentOS_Post_Install.md
[ubuntu-post-install]: https://github.com/PackeTsar/scriptfury/blob/master/Ubuntu_Post_Install.md
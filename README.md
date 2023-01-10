# 42_born2beroot

My *Born2beRoot* project for **42 Nice**, updated on November 28th 2022, at 10:20.

## CentOS VS Debian

To display info on the OS, use ```cat /etc/issue``` or ```cat /etc/os-release```.  
This project is made with **Debian 11** (*bullseye*).  
*CentOS* and *Debian* are two different Linux distributions.

### Debian
*Debian* is smaller and more stable. It takes less resources to run properly and is easy to update. It is non-commercial and only offers free software.

### CentOS
*CentOS* completes the Red Hat Enterprise Linux versions while staying available for non-professionals. It has stable versions with long-term support (each version is supported for 10 years).

## Partition

To display the partition, use ```lsblk```.  
Partition is done with *LVM* (Logical Volume Management).

*LVM* is a file system manager. It uses 3 concepts:
- Volume groups, a named collection of physical and logical volumes.
- Physical volumes, physical storage disks with the space to store logical volumes.
- Logical volumes, contain file systems and can span multiples disks (they are not physically continuous).

*LVM* can execute operations on the fly, while the partition is in use. This isn't the case with other partitioners, like *gparted*.  
The basic filesystem for GNU/Linux is currently the *ext4* standard, which officially replaced *ext3* in 2008.

## Hostname

To display the hostname, use ```hostnamectl```.  
To modify it, use ```hostnamectl set-hostname <new_name>``` and edit the file located in ```/etc/hosts```.  
To set a pretty hostname, use ```hostnamectl set-hostname <pretty_name> --pretty```.

## Groups and users

For a list of all logged users, use ```who``` or ```w```.  
For a list of all the groups a user is part of, use ```id -Gn <username>```.  
For a list of all groups and their users, use ```cat /etc/group```.  
To create a user, use ```useradd <username>```.  
To delete a user, use ```userdel <username>```.  
To add a user to a group, use ```usermod --append -G <groupname> <username>``` (this only adds a group to a user without touching the previous groups they were part of).  
To create a group, use ```groupadd <username>```.

## Passwords

The password policy is enforced through *LibPam-CrackLib*.  
To see when a password will expire, use ```chage -l <username>```.  
Its settings are found in ```/etc/pam.d/common_password```.

At around line 25 is the password policy definition. Here's what I added:
- ```retry=3``` -> only 3 retries.
- ```minlen=10``` -> at least 10 characters.
- ```difok=7``` -> at least 7 different characters form the previous password. (this is never enforced for root)
- ```ucredit=-1``` -> see further down.
- ```dcredit=-1``` -> see further down.
- ```lcredit=0``` -> see further down.
- ```ocredit=0``` -> see further down.
- ```reject_username``` -> rejects the password if it contains the username.
- ```enforce_for_root``` -> enforces these rules for the root user.

The **credit** part is tricky. Minimum length is 10 but, by default (default is ```*credit=1```), each character type, when used at least once, gives you 1 credit, which counts as length for ```minlen```. So a 9 characters long password of only lowercases will be counted as having a length of 10. As such, length here is misnamed, as it evaluates strength. As such, I set the credits given for all character types to 0 and enforce the need to have at least 1 digit and 1 uppercase (with the negative values).

## Services and daemons

A *daemon* is a non-interactive program that runs in the background. (the word *daemon* is specific to UNIX and is not universal)  
A *service* is an interactive program that can be accessed by users and other programs. A service may also be a daemon or use daemons.  

To get a list of all services, use ```sudo service --status-all```.  
- ```[+]``` means the service is active.
- ```[-]``` means the service is inactive.
- ```[?]``` means the service does not have a ```status``` command.

To get a list of all daemons, use ```ps -eo 'tty,pid,comm' | grep ^?```.  
- ```ps``` lists all ongoing processes. ```-e``` gives the environment.
- ```-o``` specifies an order, here it is ```tty,pid,comm```, so first the terminal, then the process id, and finally a specified command (here ```grep ^?```).
- ```grep ^?``` only takes lines that start with a question mark in the *tty* column. So it only displays processes that don't have a terminal, so *daemons*.

## AppArmor VS SELinux

Both securise programs by limiting their access to only the resources they need. They both are open source.

*SELinux* was originally created by the NSA. It's much more extensive but also more difficult to use.

## Apt VS Aptitude

These 2 commands manage packages, cleanly install and uninstall software and let users search for them more easily.

### Apt

*Apt* (Advanced Packaging Tool) is an open source piece of software.  
It first appeared on Debian but was later adapted to work with RPM (Red Hat Package Manager, see *CentOS VS Debian*).  
It doesn't have a GUI but is highly configurable.  
The sources and dependencies of a package must be specified in ```/etc/apt/sources.list```.
The ```apt``` command uses functionalities from *apt-cache* and *apt-get*, which are low-level complex commands.

### Aptitude

Aptitude is also open source.  
It has a GUI and a multitude of functionalities (display, colors, package marking, etc.).  
It also uses functionalities from *apt-cache* and *apt-get*, but also *apt-cache*, which makes this software high-level and more extensive than *apt*.

## Ports

Ports *0* through *1023* are usually reserved for common services.  
For example, port *22* is for **SSH**, *80* for **HTTP**, *443* for **HTTPS**, *68* for **DHCP** or *3306* for **MySQL**.  
Ports *1024* through *49151* are user ports.  
Ports *49152* through *65535* are private or dynamic ports.

To get a list of all opened ports, use ```ss -lntu```.

## Firewall and UFW

A firewall's goal is to protect your machine's ports.

UFW (Uncomplicated FireWall) allows complete novices to very simply set up a basic firewall.
- ```ufw status``` tells you if the service is active or not.
  - ```ufw status numbered``` gives you a numbered list of all opened ports.
- ```ufw enable``` activates the service.
- ```ufw disable``` desactivates the service.
- ```ufw allow``` opens a port or aset of ports.
  - ```ufw allow 42``` opens port *42*.
  - ```ufw allow 69:420/udp``` opens ports *69* through *420* as **UDP** ports.
- ```ufw delete n``` deletes the *nth* line in the rules list (use ```ufw status numbered``` to get the line numbers).
- ```ufw reset``` resets everything.

UFW sets up a whitelist firewall, meaning that all ports are closed except the ones you open deliberately.  
A blacklist firewall would be the opposite, it would open all ports except the ones you close deliberately.

## Sudo

*Sudo* (Super User Do) is a service that allows a user to get super user rights for one command only, through a password system.  
To use it, place ```sudo``` before any command.  
To directly become a super user (root), use the command ```su```.

In simpler terms, *sudo* is just like when your computer asks you to execute an action with administrator rights. Even though you own the computer, you're usually logged in as a user, not as root!

To safely configure *sudo*, use ```sudo visudo```. Here's what I edited:
- ```passwd_tries=3```-> only 3 tries to get the password right.
- ```badpass_message="hihi"``` -> the message "*hihi*" is displayed in case of a wrong password.
- ```log_input, log_output``` -> log all inputs and outputs.
- ```iolog_dir="/var/log/sudo"``` -> logs location.
- ```logfile="/var/log/sudo/<logfile_name>"``` -> a human readable version of the logs is recorded in the specified file.
- ```requiretty``` -> requires a terminal to be used, so stops daemons from sudoing.
- ```secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"``` -> restricts the paths *sudo* can use to the specified ones.

### TTY

A *TTY* (TeleTypeWriter) is basically a terminal.  
It's a sub-system of *Linux* and *Unix* permitting process and session management, as well as line editing at the kernel level.

### Kernel

The *kernel* is the core of an operating system.  
It manages ressources (processor, memory, scheduling) and allows the hardware and software to communicate (peripherals, files).

___Note:___ Scheduling refers to the choice of the execution order of processes, which is primordial for performance!  
It is done by the scheduler.

## SSH

*SSH* (Super Shell) is a service that manages port connections. It usually uses port *22*.  
It further securises *TCP* connections and is thus often used for remote terminals.

To check on it, use ```service sshd status```.  
Its configuration file is in ```/etc/ssh/sshd_config```. What I modified is:
- line 15: ```Port 4242```.
- line 34: ```PermitRootLogin no```.

### Connections

*TCP* (Transfer Control Protocol) is a secure protocol allowing communication between 2 machines or more.  
The data transfer is accompanied by confirmations for sending and receiving.

*UDP* (User Datagram Protocol) is another communication protocol which is much faster but has no verifications.  
It is thus much more prone to data loss or corruption.

Both of these protocols are part of the *IP* (Internet Protocol) family, along with *IPv4*, *IPv6*, *Ethernet*, etc.

## Cron and monitoring

The monitoring script is in ```/usr/bin/monitoring.sh```.

*Cron* is a time-based scheduling service.  
To open *cron*, use ```crontab -e```.  
To make *cron* wait, use ```sleep 30s``` (This will make *cron* wait for 30 seconds).

## WordPress and MariaDB

### MariaDB

*MariaDB* is a database manager. Its GUI is in **SQL**.  
I installed it, deleted the demo database, created mine and gave myself all rights to manage it.  
To check, use ```mariadb -u <username> -p```, log in, then use ```SHOW DATABASES;```.  
To exit, use ```EXIT;```.

### WordPress

*WordPress* is a low entry level, extensive tool for easily creating webpages.  
Its configuration file is in ```/var/www/html/wp-config.php```.  
For *WordPress* to work, I opened port *80* for **HTTP**.

To see your *WordPress* page, enter your server's IP as an adress in your internet browser.  
For me, it was ```127.0.0.1```.

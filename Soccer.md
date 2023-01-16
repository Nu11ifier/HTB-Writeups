# Reconnaissance

## Port Scan (nmap)

**Command:**\
*sudo nmap -sC -sV -O -v -T4 -p- {IP address}*

**Command Description:**\
* -sC : Use default scripts
* -sV Service version
* -O : OS detection
* -v : Verbose
* -T4 : Scan intensity (aggressive), 
* -p- : Scan all ports
* {IP address} - Target host

**Results:**\
TCP Ports Open:
* 22 - SSH
* 80 - HTTP
* 9091 - mltec-xmlmail

**Actions:**\
Edit */etc/hosts* and add the follwing line to display the website correctly on port 80 (HTTP): *{IP address} 	soccer.htb*

## Directory Scan (gobuster)

**Command:**\
*gobuster dir --url http://soccer.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt*

**Command Description:**
* dir : Directory scan
* --url : Target url
* -w : Directory wordlist

**Results:**\
Directory found: */tiny*

**Actions:**\
Navigate to: *http://soccer.htb/tiny*

## OSINT

**Actions:**\
Search on google for: *tinyfilemanager default credentials*

**Results**\
URL: https://github.com/prasathmani/tinyfilemanager

Default Credentials:\
Username/Password:
* *admin/admin@123*
* *user/12345*

Note: TinyFileManager uses PHP.

## PHP Reverse Shell

**Actions:**\
Create a new file with this php reverse shell script: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php (or any other PHP reverse shell script).\
Edit the file and change the following variables *$ip* (the machine a victim should connect to) and *$port* (the port a victim should connect to).

## Reverse Shell Listener (netcat)

**Command:** *sudo nc -lvnp 9001*

**Command Description:**\
* -l : Listener (listen for incoming connections)
* -v : Verbose
* -n : Numeric only ip addresses (no DNS)
* -p : Port

**Results:**\
Listening for incoming connections on port 9001.


## Account Access

**Actions:**\
Use admin credentials acquired from the *OSINT* phase (*admin/admin@123*) to log into the tinyfilemanger portal *soccer.htb/tiny*

## File Upload

**Actions:**\
Navigate to *tiny* directory and then *uploads* directory and then press upload on the top right corner and choose the previously created modified php reverse shell from the *PHP Reverse Shell* phase.

## Triggering The PHP Reverse Shell

**Actions:**\
Navigate to: http://soccer.htb/tiny/uploads/php-reverse-shell.php

**Results:**\
The PHP reverse shell will trigger and open a shell from the netcat listener that was setup in the *Reverse Shell Listener (netcat)* phase.

TO BE CONTINUE:

# Goals

* USER Flag:
* ROOT Flag:












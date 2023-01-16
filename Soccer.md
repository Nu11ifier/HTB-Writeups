# Reconnaissance

## Port Scan (nmap)

**Command:**\
*sudo nmap -sC -sV -O -v -T4 -p- {IP address}*

**Command Description:**\
Use default scripts, service version, OS detection, verbose, scan intensity (aggressive), all ports, target

**Results:**\
TCP Ports Open:\
22 - SSH\
80 - HTTP\
9091 - mltec-xmlmail

**Actions:**\
Edit */etc/hosts* and add the follwing line to display the website correctly on port 80 (HTTP): *{IP address} 	soccer.htb*

## Directory Scan (gobuster)

**Command:**\
*gobuster dir --url http://soccer.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt*

**Command Description:**\
directory scan, target url, directory wordlist

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
Default username/password:\
*admin/admin@123*\
*user/12345*

TinyFileManager uses PHP.

# Weaponization

## PHP Reverse Shell

**Actions:**\
Create a new file with this php reverse shell script: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php (or any other PHP reverse shell script).\
Edit the file and change the following variables *$ip* (the machine a victim should connect to) and *$port* (the port a victim should connect to).

## Reverse Shell Listener (netcat)

**Command:** *sudo nc -lvnp 9001*

**Command Description:**\
listener (listen for incoming connections), verbose, numeric only ip addresses (no DNS), port

**Results:**\
Listening for incoming connections on port 9001.

# Delivery

## Account Access

**Actions:**\
Use admin credentials acquired from the *Reconnaissance* phase (*admin/admin@123*) to log into the tinyfilemanger portal *soccer.htb/tiny*

## File Upload

**Actions:**\
Navigate to *tiny* directory and then *uploads* directory and then press upload on the top right corner and choose the previously created modified php reverse shell from the *Weaponization* phase.

# Exploitation

## Triggering The PHP Reverse Shell

**Actions:**\
Navigate to: http://soccer.htb/tiny/uploads/php-reverse-shell.php

**Results:**\
The PHP reverse shell will trigger and open a shell from the netcat listener that was setup in the *Weaponization* phase.

# Attack On Objectives

USER FLAG:
ROOT FLAG:












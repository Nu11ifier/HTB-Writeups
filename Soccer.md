# Reconnaissance

## Port Scan (nmap)

**Command:** *sudo nmap -sC -sV -O -v -T4 -p- {IP address}*

**Description:**  Use default scripts, service version, OS detection, verbose, scan intensity (aggressive), all ports, target

**Results:** TCP Ports Open:
22 - SSH
80 - HHTTP
9091 - mltec-xmlmail

**Actions:** edit */etc/hosts* and add the follwing line to display the website on port 80 (HTTP) correctly: *{IP address} 	soccer.htb*

## Directory Scan (gobuster)

**Command:** *gobuster dir --url http://soccer.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt*

**Description:** directory fuzz, target url, directory wordlist

**Results:** */tiny* directory found

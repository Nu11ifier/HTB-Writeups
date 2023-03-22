# Reconnaissance

## Port Scan (nmap)

**Command:**\
*sudo nmap -sC -sV -p- {IP address}*

**Command Description:**\
* -sC : Use default scripts
* -sV Service version
* -p- : Scan all ports
* {IP address} - Target host

**Results:**\
TCP Ports Open:
* 21 - FTP
* 22 - SSH
* 80 - HTTP

**Actions:**\
Edit */etc/hosts* and add the follwing line to display the website correctly on port 80 (HTTP): *{IP address} 	metapress.htb*

Looking at the page we can see that it is most likely WordPress. We can check the source code of http://metapress.htb/events/ or also use *Wappalyzer* extension to see that it is using WordPress with version 5.6.2.

![image](https://user-images.githubusercontent.com/85443537/226896247-4f303963-a538-4210-a13e-2759fa3cf1ed.png)

## Directory Scan (gobuster)

**Command:**\
*gobuster dir -u http://metapress.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt*

**Command Description:**
* dir : Directory scan
* -u : Target url
* -w : Directory wordlist

**Results:**\
The interesting directory that was found: */wp-admin*

**Actions:**\
Navigate to: *http://metapress.htb/wp-admin*

After navigating to wp-admin we find that it requires a login. We try default passwords but with no luck.

##  Finding possible WordPress vulnerabilities using WPScan (wpscan)

We type the following command to run the wordpress security scan.

**Command:**\
*wpscan --url metapress.htb --api-token {Your API-token}
*

**Results:**\
We find a possible vulnerability "Authenticated XXE Within the Media Library Affecting PHP 8" for WordPress versions 5.6-5.7. The WordPress version of the webserver found was 5.6.2.

![image](https://user-images.githubusercontent.com/85443537/226898865-a6e1404a-e6a6-469f-bf6c-b7ad5882afea.png)

We navigate to the first link of "References" (https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5
) to learn more about how to exploit this identified vulnerability.





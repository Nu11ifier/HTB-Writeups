# Reconnaissance

## Port Scan (nmap)
We first perform a nmap portscan.

Command: *sudo nmap -sC -sV -p- 10.10.11.204 -oA nmap/inject*

we can see port 8080 is open we add inject to */etc/hosts* file.

We then navigate to inject:8080 and we are presented with a page:

![image](https://user-images.githubusercontent.com/85443537/229883216-56fb3183-da96-415a-aabb-ac28b6fc471a.png)

Crawling the page we find nothing interesting apart from a upload site.

## File Path Traversal

After trying to upload different shells/commands with various extensions/magic bytes, we get no progress.

After uploading and viewing the file we always get a url with parameter that points to the uploaded file.

![image](https://user-images.githubusercontent.com/85443537/229884181-3424e4aa-0de2-48ad-93b9-4040dec3dd1e.png)

Trying to include a file after the "img" parameter does not work, so we try file path traversal with the help of burp suite.

After capturing the request we can send it to the *Repeater* tab in burpsuite:

![image](https://user-images.githubusercontent.com/85443537/229884667-90271e0d-5562-42b5-aae4-4816dab10fa7.png)

We now have the abillity to request local system files (like the image above).

Looking at the */etc/passwd* file we find that the users that exists are root, frank and phil.

Traversing the target system using burpsuite we find a hidden directory ".m2" in frank's home directory using the command "ls -la". We then change to the ".m2" directory and find a *settings.xml* file.

The image below displays username and password used for maven.
Username: phil
Password: DocPhillovestoInject123

![image](https://user-images.githubusercontent.com/85443537/229886184-085a8440-6bce-4546-b9e8-95fd5dc67147.png)

Trying the aquired credentials on SSH does not work, so we continue looking...

## Finding Dependency Vulnerabilities

We know it is using maven, so we look for a *pom.xml* file that contains the dependencies used for the project.

We find the pom.xml file and look for any dependencies that have vulnerable versions.

![image](https://user-images.githubusercontent.com/85443537/230088343-f53a47da-8314-4feb-b7bf-3467d9c7ea70.png)

We find a vulnerable dependecy affecting *org.springframework.cloud* with the CVE *CVE-2022-22963*.

![image](https://user-images.githubusercontent.com/85443537/230090290-8a242226-526e-4707-8bcc-20116ca29a91.png)

Going to *spring.io* we can read more about the vulnerability https://spring.io/security/cve-2022-22963

## Exploiting Spring Vulnerability in MSFConsole

We open up msfconsole and find the exploit:

![image](https://user-images.githubusercontent.com/85443537/230091596-f654c65f-353f-4f58-a25c-eedffd7710c8.png)

Description of the vulnerability:
*Spring Cloud Function versions prior to 3.1.7 and 3.2.3 are vulnerable to remote code execution due to using an unsafe evaluation context with user-provided queries. By crafting a request to the application and setting the spring.cloud.function.routing-expression header, an unauthenticated attacker can gain remote code execution. Both patched and unpatched servers will respond with a 500 server error and a JSON encoded*

We fill in the necessary settings into msfconsole (LHOST and RHOSTS) and then type exploit.

![image](https://user-images.githubusercontent.com/85443537/230092294-4b1f9133-8dd4-4c9e-b58f-4250f28a0d80.png)

We get a reverse meterpreter shell.
We then type shell in the meterpreter shell to get a normal shell.
We go to the home directory of phil */home/phil* and find the user flag, but we have no permission to read it, so we try to change user locally to phil with the command *su phil*. It prompt us to input a password. We try to use the previously aquired password for phil:

Username: phil
Password: DocPhillovestoInject123

And we succefully log into phil. We are now able to read the user.txt flag.

![image](https://user-images.githubusercontent.com/85443537/230094502-9b5d5d37-aaa0-4e7f-9ab2-51c452128a2c.png)

## Getting Root Through Ansible Automation

Looking for files we found a directory in franks home directory called ansible. It looks like the system is using ansible for automation. We continue looking for files related to ansible and we find a *.yml* file that performs a automated task.

![image](https://user-images.githubusercontent.com/85443537/230095529-1cee9c12-9e55-4616-ac2e-d46f3b8d01ac.png)

Doing some OSINT we find that ansible can execute commands through the playbook *.yml* file (Reference: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html#playbook-syntax).

We create a new file and call it *playbook.yml*. 

We change it so it looks similar to the image below. We set the command to *chmod u+s /bin/bash* to set the suid of bash, so when a user runs it, it will be running with higher privileges.

![image](https://user-images.githubusercontent.com/85443537/230097463-5b00c7b9-5a70-4d4f-ae48-5eee328b76cf.png)

We then upload the playbook.yml file we created to the same directory (*/opt/automation/tasks/*) using either python webserver (*python3 -m http.server*) or using the meterpreter shell.

Once we have it in the *tasks* directory...

![image](https://user-images.githubusercontent.com/85443537/230099539-7de58215-a704-462a-afb0-9a7e315ea6d6.png)

 We can then type the command *bash -p* and we get root.

![image](https://user-images.githubusercontent.com/85443537/230099838-2f17d47c-bdd3-4591-8b6b-c8906da45e7d.png)

We can now aquire the root.txt flag.

![image](https://user-images.githubusercontent.com/85443537/230100192-b0f0f821-dea3-40f6-ae4c-cb38993d6860.png)




# Goals

* USER FLAG: 0e4c38a01edd6187c787231ccc6715f8
* ROOT FLAG: 0c7973cd1eb545f21857997139761660








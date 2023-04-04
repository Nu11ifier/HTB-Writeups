# Reconnaissance

## Port Scan (nmap)
We first perform a nmap portscan.

Command: *sudo nmap -sC -sV -p- 10.10.11.204 -oA nmap/inject*

we can see port 8080 is open we add inject to */etc/hosts* file.

We then navigate to inject:8080 and we are presented with a page:

![image](https://user-images.githubusercontent.com/85443537/229883216-56fb3183-da96-415a-aabb-ac28b6fc471a.png)

Crawling the page we find nothing interesting apart from a upload site.

After trying to upload different shells/commands with various extensions/magic bytes, we get no progress.

After uploading and viewing the file we always get a url with parameter that points to the uploaded file.

![image](https://user-images.githubusercontent.com/85443537/229884181-3424e4aa-0de2-48ad-93b9-4040dec3dd1e.png)

Trying to include a file after the "img" parameter does not work, so we try file path traversal with the help of burp suite.

After capturing the request we can send it to the *Repeater* tab in burpsuite:

![image](https://user-images.githubusercontent.com/85443537/229884667-90271e0d-5562-42b5-aae4-4816dab10fa7.png)

We now have the abillity to request local system files (like the image above).

Looking at the */etc/passwd* file we find that the users that exists are root, frank and phil.

Traversing the target system using burpsuite we find a hidden directory ".m2" in frank's home directory using the command "ls -la". We then change to the ".m2" directory and find a *settingss.xml* file.

The image below displays username and password used for maven.
Username: phil
Password: DocPhillovestoInject123

![image](https://user-images.githubusercontent.com/85443537/229886184-085a8440-6bce-4546-b9e8-95fd5dc67147.png)

Trying the aquired credentials on SSH does not work, so we continue looking...



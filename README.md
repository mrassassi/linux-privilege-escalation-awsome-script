# LinPEAS - Linux Privilege Escalation Awsome Script (with colors!!)

Also valid for **Unix systems**


**LinPEAS performs the linux privilege escalation checks explained in [book.hacktricks.xyz](https://book.hacktricks.xyz/linux-unix/privilege-escalation)**

[![asciicast](https://asciinema.org/a/250532.png)](https://asciinema.org/a/250532)


The goal of this script is to search for possible **Privilege Escalation vectors**.

This script doesn't have any dependency.

The script can be run in everything that have **/bin/sh** (even OpenBSD, FreeBSD and other OS with /bin/sh).

It could take from **2 to 3 minutes** to execute the whole script (less than 1 min to make almost all the checks, almost 1 min to search for possible passwords inside all the accesible files of the system and 1 min to monitor the processes in order to find very frequent cron jobs). 

You can **decrease this** time use the parameters: 
- **-f** (fast) - This will bypass checking processes during 1 min
- **-v** (veryfast) - This will bypass the previous check and other time consuming checks.

This script has **several lists** included inside of it to be able to **color the results** in order to highlight PE vector.

LinPEAS also **exports a new PATH** variable if common folders aren't present in the original PATH variable. It also **exports** `export HISTSIZE=0` so no command executed during the session will be saved in the history file.

## Colors

LinPEAS uses colors to indicate where does each section begin. But **it also uses them the identify potencial misconfigurations**.

The ![](https://placehold.it/15/b32400/000000?text=+) **Red/Yellow** ![](https://placehold.it/15/fff500/000000?text=+) color is used for identifing configurations that lead to PE (99% sure).

The ![](https://placehold.it/15/b32400/000000?text=+) **Red** color is used for identifing suspicious configurations that could lead to PE:
- Possible exploitable kernel versions
- Vulnerable sudo versions
- Identify processes running as root
- Not mounted devices
- Dangerous fstab permissions
- Writable files in interesting directories
- SUID/SGID binaries that have some vulnerable version (it also specifies the vulnerable version)
- SUDO binaries that can be used to escalate privileges in sudo -l (without passwd) (https://gtfobins.github.io/)
- Check /etc/doas.conf
- 127.0.0.1 in netstat
- Known files that could contain passwords
- Capabilities in interesting binaries
- Interesting capabilities of a binary
- Writable folders and wilcards inside info about cron jobs
- Writables folders in PATH
- Groups that could lead to root
- Files that could contains passwords

The ![](https://placehold.it/15/66ff33/000000?text=+) **Green** color is used for:
- Common processes run by root
- Common not interesting devices to mount
- Not dangerous fstab permissions
- SUID/SGID common binaries (the bin was already found in other machines and searchsploit doesn't identify any vulnerable version)
- Common .sh files in path
- Common names of users executing processes

The ![](https://placehold.it/15/0066ff/000000?text=+) **Blue** color is used for:
- Users without shell
- Mounted devices

The ![](https://placehold.it/15/33ccff/000000?text=+) **Light Cyan** color is used for:
- Users with shell

The ![](https://placehold.it/15/bf80ff/000000?text=+) **Light Magenta** color is used for:
- Current username


**The color filtering is not available in the one-liner** (the lists are too big)


## One liner

Here you have an old linpe version script in one line, **just copy and paste it**;)

This one-liner is deprecated (I am not going to update it more), but it could be useful in some cases so it will remain here:

The default file where all the data is recorded is: */tmp/linPE* (you can change it at the beginning of the script)


```sh
file="/tmp/linPE";RED='\033[0;31m';Y='\033[0;33m';B='\033[0;34m';NC='\033[0m';rm -rf $file;echo "File: $file";echo "[+]Gathering system information...";printf $B"[*] "$RED"BASIC SYSTEM INFO\n"$NC >> $file ;echo "" >> $file;printf $Y"[+] "$RED"Operative system\n"$NC >> $file;(cat /proc/version || uname -a ) 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"PATH\n"$NC >> $file;echo $PATH 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Date\n"$NC >> $file;date 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Sudo version\n"$NC >> $file;sudo -V 2>/dev/null| grep "Sudo ver" >> $file;echo "" >> $file;printf $Y"[+] "$RED"selinux enabled?\n"$NC >> $file;sestatus 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Useful software?\n"$NC >> $file;which nc ncat netcat wget curl ping gcc make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Capabilities\n"$NC >> $file;getcap -r / 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Environment\n"$NC >> $file;(set || env) 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Top and cleaned proccesses\n"$NC >> $file;ps aux 2>/dev/null | grep -v "\[" >> $file;echo "" >> $file;printf $Y"[+] "$RED"Binary processes permissions\n"$NC >> $file;ps aux 2>/dev/null | awk '{print $11}'|xargs -r ls -la 2>/dev/null |awk '!x[$0]++' 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Services\n"$NC >> $file;(/usr/sbin/service --status-all || /sbin/chkconfig --list || /bin/rc-status) 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Different processes executed during 1 min (HTB)\n"$NC >> $file;if [ "`ps -e --format cmd`" ]; then for i in {1..121}; do ps -e --format cmd >> $file.tmp1; sleep 0.5; done; sort $file.tmp1 | uniq | grep -v "\[" | sed '/^.\{500\}./d' >> $file; rm $file.tmp1; fi;echo "" >> $file;printf $Y"[+] "$RED"Proccesses binary permissions\n"$NC >> $file;ps aux 2>/dev/null | awk '{print $11}'|xargs -r ls -la 2>/dev/null |awk '!x[$0]++' 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Scheduled tasks\n"$NC >> $file;crontab -l 2>/dev/null >> $file;ls -al /etc/cron* 2>/dev/null >> $file;cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root /var/spool/anacron 2>/dev/null | grep -v "^#" >> $file;echo "" >> $file;printf $Y"[+] "$RED"Any sd* disk in /dev?\n"$NC >> $file;ls /dev 2>/dev/null | grep -i "sd" >> $file;echo "" >> $file;printf $Y"[+] "$RED"Storage information\n"$NC >> $file;df -h 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Unmounted file-system?\n"$NC >> $file;cat /etc/fstab 2>/dev/null | grep -v "^#" >> $file;echo "" >> $file;printf $Y"[+] "$RED"Printer?\n"$NC >> $file;lpstat -a 2>/dev/null >> $file;echo "" >> $file;echo "" >> $file;echo "[+]Gathering network information...";printf $B"[*] "$RED"NETWORK INFO\n"$NC >> $file ;echo "" >> $file;printf $Y"[+] "$RED"Hostname, hosts and DNS\n"$NC >> $file;cat /etc/hostname /etc/hosts /etc/resolv.conf 2>/dev/null | grep -v "^#" >> $file;dnsdomainname 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Networks and neightbours\n"$NC >> $file;cat /etc/networks 2>/dev/null >> $file;(ifconfig || ip a) 2>/dev/null >> $file;iptables -L 2>/dev/null >> $file;ip n 2>/dev/null >> $file;route -n 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Ports\n"$NC >> $file;(netstat -punta || ss -t; ss -u) 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Can I sniff with tcpdump?\n"$NC >> $file;timeout 1 tcpdump >> $file 2>&1;echo "" >> $file;echo "" >> $file;echo "[+]Gathering users information...";printf $B"[*] "$RED"USERS INFO\n"$NC >> $file ;echo "" >> $file;printf $Y"[+] "$RED"Me\n"$NC >> $file;(id || (whoami && groups)) 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Sudo -l without password\n"$NC >> $file;echo '' | sudo -S -l -k 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Do I have PGP keys?\n"$NC >> $file;gpg --list-keys 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Superusers\n"$NC >> $file;awk -F: '($3 == "0") {print}' /etc/passwd 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Login\n"$NC >> $file;w 2>/dev/null >> $file;last 2>/dev/null | tail >> $file;echo "" >> $file;printf $Y"[+] "$RED"Users with console\n"$NC >> $file;cat /etc/passwd 2>/dev/null | grep "sh$" >> $file;echo "" >> $file;printf $Y"[+] "$RED"All users\n"$NC >> $file;cat /etc/passwd 2>/dev/null | cut -d: -f1 >> $file;echo "" >> $file;echo "" >> $file;echo "[+]Gathering files information...";printf $B"[*] "$RED"INTERESTING FILES\n"$NC >> $file ;echo "" >> $file;printf $Y"[+] "$RED"SUID\n"$NC >> $file;find / -perm -4000 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"SGID\n"$NC >> $file;find / -perm -g=s -type f 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Files inside \$HOME (limit 20)\n"$NC >> $file;ls -la $HOME 2>/dev/null | head -n 20 >> $file;echo "" >> $file;printf $Y"[+] "$RED"20 First files of /home\n"$NC >> $file;find /home -type f 2>/dev/null | column -t | grep -v -i "/"$USER | head -n 20 >> $file;echo "" >> $file;printf $Y"[+] "$RED"Files inside .ssh directory?\n"$NC >> $file;find  /home /root -name .ssh 2>/dev/null -exec ls -laR {} \; >> $file;echo "" >> $file;printf $Y"[+] "$RED"*sa_key* files\n"$NC >> $file;find / -type f -name "*sa_key*" -ls 2>/dev/null -exec ls -l {} \; >> $file;echo "" >> $file;printf $Y"[+] "$RED"Mails?\n"$NC >> $file;ls -alh /var/mail/ /var/spool/mail/ 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"NFS exports?\n"$NC >> $file;cat /etc/exports 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Hashes inside /etc/passwd? Readable /etc/shadow or /etc/master.passwd?\n"$NC >> $file;grep -v '^[^:]*:[x]' /etc/passwd 2>/dev/null >> $file;cat /etc/shadow /etc/master.passwd 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Readable /root?\n"$NC >> $file;ls -ahl /root/ 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Inside docker or lxc?\n"$NC >> $file;dockercontainer=`grep -i docker /proc/self/cgroup  2>/dev/null; find / -name "*dockerenv*" -exec ls -la {} \; 2>/dev/null`;lxccontainer=`grep -qa container=lxc /proc/1/environ 2>/dev/null`;if [ "$dockercontainer" ]; then echo "Looks like we're in a Docker container" >> $file; fi;if [ "$lxccontainer" ]; then echo "Looks like we're in a LXC container" >> $file; fi;echo "" >> $file;printf $Y"[+] "$RED"*_history, profile, bashrc, httpd.conf\n"$NC >> $file;find / -type f \( -name "*_history" -o -name "profile" -o -name "*bashrc" -o -name "httpd.conf" \) -exec ls -l {} \; 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"All hidden files (not in /sys/) (limit 100)\n"$NC >> $file;find / -type f -iname ".*" -ls 2>/dev/null | grep -v "/sys/" | head -n 100 >> $file;echo "" >> $file;printf $Y"[+] "$RED"What inside /tmp, /var/tmp, /var/backups\n"$NC >> $file;ls -a /tmp /var/tmp /var/backups 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Interesting writable Files\n"$NC >> $file;USER=`whoami`;HOME=/home/$USER;find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | grep -v '/sys/fs'| sort | uniq >> $file;for g in `groups`; do find / \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME | grep -v '/sys/fs'; done >> $file;echo "" >> $file;printf $Y"[+] "$RED"Web files?(output limited)\n"$NC >> $file;ls -alhR /var/www/ 2>/dev/null | head >> $file;ls -alhR /srv/www/htdocs/ 2>/dev/null | head >> $file;ls -alhR /usr/local/www/apache22/data/ 2>/dev/null | head >> $file;ls -alhR /opt/lampp/htdocs/ 2>/dev/null | head >> $file;echo "" >> $file;printf $Y"[+] "$RED"Backup files?\n"$NC >> $file;find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*back*" -o -name "*bck*" \) 2>/dev/null >> $file;echo "" >> $file;printf $Y"[+] "$RED"Find IPs inside logs\n"$NC >> $file;grep -a -R -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' /var/log/ 2>/dev/null | sort | uniq >> $file;echo "" >> $file;printf $Y"[+] "$RED"Find 'password' or 'passw' string inside /home, /var/www, /var/log, /etc\n"$NC >> $file;grep -lRi "password\|passw" /home /var/www /var/log 2>/dev/null | sort | uniq >> $file;echo "" >> $file;printf $Y"[+] "$RED"Sudo -l (you need to puts the password and the result appear in console)\n"$NC >> $file;sudo -l;
```


## What does linpe look for
- **System Information**
  - [x] SO & kernel version 
  - [x] Sudo version
  - [x] PATH
  - [x] Date
  - [x] System stats
  - [x] Environment vars
  - [x] SElinux
  - [x] Printers
  - [x] Dmesg (signature verifications)
  - [x] Container?

- **Devices**
  - [x] sd* in /dev
  - [x] Unmounted filesystems

- **Available Software**
  - [x] Useful software
  - [x] Installed compilers

- **Processes & Cron & Services**
  - [x] Cleaned processes
  - [x] Binary processes permissions
  - [x] Different processes executed during 1 min
  - [x] Cron jobs
  - [x] Services

- **Network Information**
  - [x] Hostname, hosts & dns
  - [x] Content of /etc/inetd.conf
  - [x] Networks and neighbours
  - [x] Active ports
  - [x] Sniff permissions (tcpdump)

- **Users Information**
  - [x] Info about current user
  - [x] PGP keys
  - [x] `sudo -l` without password
  - [x] doas config file
  - [x] Pkexec policy
  - [x] Try to login using `su` as other users (using null pass and the username)
  - [x] List of superusers
  - [x] List of users with console
  - [x] Login info
  - [x] List of all users

- **Software Information**
  - [x] MySQl (Version, user being configured, loging as "root:root","root:toor","root:", user hashes extraction via DB and file, possible backup user configured)
  - [x] PostgreSQL (Version, try login in "template0" and "template1" as: "postgres:", "psql:")
  - [x] Apache (Version)
  - [x] PHP cookies
  - [x] Wordpress (Database credentials)
  - [x] Tomcat (Credentials)
  - [x] Mongo (Version)
  - [x] Supervisor (Credentials)
  - [x] Cesi (Credentials)
  - [x] Rsyncd (Credentials)
  - [x] Hostapd (Credentials)
  - [x] Wifi (Credentials)
  - [x] Anaconda-ks (Credentials)
  - [x] VNC (Credentials)
  - [x] LDAP database (Credentials)
  - [x] Open VPN files (Credentials)
  - [x] SSH (private keys, known_hosts, authorized_hosts, authorized_keys, main config parameters in sshd_config, certificates)
  - [X] PAM-SSH (Unexpected "auth" values)
  - [x] AWS (Files with AWS keys)
  - [x] NFS (privilege escalation misconfiguration)
  - [x] Kerberos (configuration & tickets in /tmp)
  - [x] Kibana (credentials)
  - [x] Logstash (Username and possible code execution)
  - [x] Elasticseach (Config info and Version via port 9200)
  - [x] Vault-ssh (Config values, secrets list and .vault-token files)


- **Generic Interesting Files**
  - [x] SUID & SGID files
  - [x] Capabilities
  - [x] .sh scripts in PATH
  - [x] Hashes (passwd, shadow & master.passwd)
  - [x] Try to read root dir
  - [x] Files owned by root inside /home
  - [x] Reduced list of files inside my home and /home
  - [x] Mails
  - [x] Backup files
  - [x] DB files
  - [x] Web files
  - [x] Files that can contain passwords (and search for passwords inside *_history files)
  - [x] List of all hidden files
  - [x] List ALL writable files for current user (global, user and groups)
  - [x] Inside /tmp, /var/tmp and /var/backups
  - [x] Password ins config PHP files
  - [x] Get IPs, passwords and emails from logs
  - [x] "pwd" and "passw" inside files (and get most probable lines)


By Polop<sup>(TM)</sup>

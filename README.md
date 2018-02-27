<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [OHammami Sys Notes](#ohammami-sys-notes)
  - [MySQL Notes](#mysql-notes)
    - [How do I make sure that BOTH my server and MYSQL have the same timezone?](#how-do-i-make-sure-that-both-my-server-and-mysql-have-the-same-timezone)
    - [Log all queries in mysql for a session](#log-all-queries-in-mysql-for-a-session)
    - [Replace MariaDB with MySQL](#replace-mariadb-with-mysql)
    - [Centos 7 Reset mysql root password](#centos-7-reset-mysql-root-password)
    - [Create mysql user](#create-mysql-user)
  - [Hostname Centos 7](#hostname-centos-7)
  - [Amazon Restoring a volume from a snapshot](#amazon-restoring-a-volume-from-a-snapshot)
  - [iTerm](#iterm)
  - [Create new swap file on CentOS 7 (google cloud)](#create-new-swap-file-on-centos-7-google-cloud)
  - [Asterisk Chan_dongle](#asterisk-chan_dongle)
  - [How to find processes using serial port](#how-to-find-processes-using-serial-port)
  - [Smart bash commands](#smart-bash-commands)
  - [Let's Encrypt](#lets-encrypt)
  - [NTPd Centos 7](#ntpd-centos-7)
    - [Enable NTP](#enable-ntp)
    - [Disable Chrony](#disable-chrony)
  - [How can I instruct yum to install a specific version of package X?](#how-can-i-instruct-yum-to-install-a-specific-version-of-package-x)
  - [OpenSSL](#openssl)
    - [Verifying that a Private Key Matches a Certificate](#verifying-that-a-private-key-matches-a-certificate)
  - [Yum transaction database](#yum-transaction-database)
  - [How to bind to port number less than 1024 with non root access?](#how-to-bind-to-port-number-less-than-1024-with-non-root-access)
  - [Package manager and a Ruby Environment Manager On Mac](#package-manager-and-a-ruby-environment-manager-on-mac)
  - [Fix locale problems in CentOS 6 Linux](#fix-locale-problems-in-centos-6-linux)
  - [Edit a remote file over SSH](#edit-a-remote-file-over-ssh)
  - [SSH Mac OS X config](#ssh-mac-os-x-config)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

OHammami Sys Notes
=================

This repo contains my tips, tricks, and notes that I have found useful and tricky to remember.

Because I enjoy copy & past !

## MySQL Notes

### How do I make sure that BOTH my server and MYSQL have the same timezone?

You don't mention an OS but for RedHat derived it should be system-config-time to setup your timezone. For MySQL read this URL:

http://dev.mysql.com/doc/refman/5.1/en/mysql-tzinfo-to-sql.html

To summarize, I had to load the timezones from the OS: 

```
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql
```

Add the following line to `/etc/my.cnf` in the mysqld section and restart:

```
default-time-zone='Pacific/Honolulu'
```

And Bob as your uncle hhhhhhh :) @ouss

```
mysql> SELECT @@global.time_zone, @@session.time_zone;
+--------------------+---------------------+
| @@global.time_zone | @@session.time_zone |
+--------------------+---------------------+
| Pacific/Honolulu   | Pacific/Honolulu    | 
+--------------------+---------------------+
1 row in set (0.00 sec)
```

Test with


```
# systemctl restart mysqld
# mysql -uroot -pY2M4N2EwMWJmMzZ mysql -e "select NOW();SELECT @@global.time_zone, @@session.time_zone;select count(*) from time_zone;"

+---------------------+
| NOW()               |
+---------------------+
| 2017-07-12 11:05:31 |
+---------------------+
+--------------------+---------------------+
| @@global.time_zone | @@session.time_zone |
+--------------------+---------------------+
| Europe/Stockholm   | Europe/Stockholm    |
+--------------------+---------------------+
+----------+
| count(*) |
+----------+
|     1780 |
+----------+
```

### Log all queries in mysql for a session 

```
# mysql -uroot -pmypassword -e "SELECT @@global.general_log;"
# mysql -uroot -pmypassword

mysql> SET global log_output = 'FILE';
Query OK, 0 rows affected (0.00 sec)

mysql> SET global general_log_file='/var/log/mysqld.log';
Query OK, 0 rows affected (0.00 sec)

mysql> SET global general_log = 1;
Query OK, 0 rows affected (0.00 sec)

```

### Replace MariaDB with MySQL

```
# yum install http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# yum update
```

### Centos 7 Reset mysql root password

To reset the root password, you still start mySQL with --skip-grant-tables options and update the user table, but how you do it has changed.

```
1. Stop mysql:
systemctl stop mysqld

2. Set the mySQL environment option 
systemctl set-environment MYSQLD_OPTS="--skip-grant-tables"

3. Start mysql usig the options you just set
systemctl start mysqld

4. Login as root
mysql -u root

5. Update the root user password with these mysql commands
mysql> UPDATE mysql.user SET authentication_string = PASSWORD('MyNewPassword')
    -> WHERE User = 'root' AND Host = 'localhost';
mysql> FLUSH PRIVILEGES;
mysql> quit

6. Stop mysql
systemctl stop mysqld

7. Unset the mySQL envitroment option so it starts normally next time
systemctl unset-environment MYSQLD_OPTS

8. Start mysql normally:
systemctl start mysqld

Try to login using your new password:
7. mysql -u root -p
```

### Create mysql user

```
mysql> GRANT ALL ON kamailio.* TO kamailio@[IP ADRESS] IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> GRANT ALL ON asterisk.* TO asterisk@[IP ADRESS] IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> GRANT SELECT ON kamailio.* TO kamailioro@[IP ADRESS] IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.02 sec)
```

## Hostname Centos 7

I found that my CentOS 7 Instance uses Cloud-Init every reboot and it sets to originally given hostname every time I reboot the instance.

I found a solution here:

https://www.ovh.pt/g1928.hostname

Which tells that and to get around must have to deactivate an cloud-init module with: manage_etc_hosts: false in /etc/cloud/cloud.cfg file, and then hostname to whatever you want.

But since my /etc/cloud/cloud.cfg file was different I just deleted:

- set_hostname
- update_hostname

under cloud_init_modules and it worked for me


## Amazon Restoring a volume from a snapshot

To replace a volume attached to an instance with a new volume created from a snapshot:

- Create a volume from the snapshot in the same availability zone the instance is in (right click on snapshot and click "create volume from snapshot")
- Best to stop the instance to avoid any application from crashing. Wait until instance is stopped.
- Write down the exact device name of the original volume (it is written in the AWS console under instances view or volumes view)
- Detach the old volume, delete it afterwards if you don't need it.
- Attach the newly created volume (from the snapshot) to the instance with the same device name.
- Start the instance again


## iTerm

**Paste History**

> To access everything you’ve pasted into iTerm, use `⌘``⇧``h`

**Input to all panes**

> input to all panes in current tab `⌘``⇧``i`

**Autocomplete**

> To use autocomplete, type the beginning of a word and then press `⌘``⇧``;`

## Create new swap file on CentOS 7 (google cloud)

**Prerequisites**

Must have free space on mounted disk. You can check by using `df -Th` command.

**Steps to create / add new swap file on Linux**

1. Create swapfile-additional file with dd command in / (root). You can select any other partition but it should be mounted (For eg. /opt, /usr ,/NewMountedPartition). 

   ```
   dd if=/dev/zero of=/swapfile-additional bs=1M count=2048
   ```
   
  * **dd =** It is a unix command used for convert and copy a file
  * **if =** read from FILE instead of stdin
  * **/dev/zero =** /dev/zero is a special file in Unix-like operating systems that provides as many null characters (ASCII NUL, 0x00) as are read from it
  * **of =** write to FILE instead of stdout. 
  * **/swapfile-additional =** file named swapfile-additional will be created in /
  * **bs** = Read and write bytes at a time but if you do not mention MB or GB like only number it will read as bytes. for eg. bs=1024 means 1024 bytes
  * **count** = Copy input blocks in our case it is 1024 (1M * 4048 = 4GB)

2. Run mkswap command to make swap area

   ```
   mkswap /swapfile-additional
   ```

3. Change the permission of file swapfile-additional

   ```
   chmod 600 /swapfile-additional
   ```

4. Permanent mounting the swap space by editing the /etc/fstab file .
Use your file editor, I generally use vi editor.

   ```
   vi /etc/fstab
   ```
   
   Paste below given content in `/etc/fstab` file

   ```
   /swapfile-additional                                              swap    swap            0 0
   ```

5. Now mount the swap area, run below given command.

   ```
   mount -a
   ```

6. Enable the swap area

   ```
   swapon -a
   ```
**NB** if you get this error 

   ```
   # swapon -a
   swapon: /swapfile-additional: read swap header failed: Invalid argument

   # mkswap /swapfile-additional
   Setting up swapspace version 1, size = 2097148 KiB
   no label, UUID=1d6bd462-0c92-42d2-8b34-80be0fc5e406
   
   # mount -a
   # swapon -a
   ```
   
7. Check the number swap space mounted on your system

   ```
   swapon -s
   ```

8. To check how much is swap space available on system.Run below given command

   ```
   free -m
   ```

## Asterisk Chan_dongle

can only switch to 2001:7d0e , 2001:7d02 is for the B1 version so edit your config files TargetProduct to reflect that.
The only thing you miss now is the option serial driver which you'll have to bind by manual cmds:

```
modprobe option
echo "2001 7d0e" > /sys/bus/usb-serial/drivers/option1/new_id

You should then see 4 ttyUSB devices created in /dev , ttyUSB0 is the ppp dialup modem. 
```

Ref:
http://www.draisberghof.de/usb_modeswitch/bb/viewtopic.php?f=3&t=2496

https://www.serveradminblog.com/2015/01/huawei-e1552e1800e173-on-centos-6/

```
[root@rebteloffice ~]# ll /dev/ttyUSB*
crw-rw----. 1 root dialout 188, 0 Sep 11 18:51 /dev/ttyUSB0
crw-rw----. 1 root dialout 188, 1 Sep 11 18:51 /dev/ttyUSB1
crw-rw----. 1 root dialout 188, 2 Sep 11 18:51 /dev/ttyUSB2
crw-rw----. 1 root dialout 188, 3 Sep 11 18:51 /dev/ttyUSB3

# dmesg
[ 3295.851782] usbcore: registered new interface driver option
[ 3295.851812] usbserial: USB Serial support registered for GSM modem (1-port)
[ 3357.125194] option 2-1.3:1.2: GSM modem (1-port) converter detected
[ 3357.125748] usb 2-1.3: GSM modem (1-port) converter now attached to ttyUSB0
[ 3357.125857] option 2-1.3:1.3: GSM modem (1-port) converter detected
[ 3357.126007] usb 2-1.3: GSM modem (1-port) converter now attached to ttyUSB1
[ 3357.126111] option 2-1.3:1.4: GSM modem (1-port) converter detected
[ 3357.126325] usb 2-1.3: GSM modem (1-port) converter now attached to ttyUSB2
[ 3357.126470] option 2-1.3:1.5: GSM modem (1-port) converter detected
[ 3357.126943] usb 2-1.3: GSM modem (1-port) converter now attached to ttyUSB3
```


http://www.raspberry-asterisk.org/documentation/gsm-voip-gateway-with-chan_dongle/
https://boffblog.wordpress.com/2013/09/09/how-to-autostart-a-usb-3g-key/


## How to find processes using serial port

```
# ls -l /proc/[0-9]*/fd/* |grep /dev/ttyUSB0
lrwx------ 1 root     root     64 sep 12 16:24 /proc/1747/fd/8 -> /dev/ttyUSB0
# ps aux|grep 1747
root      1747  0.0  0.0  34900  1304 ?        S    15:50   0:00 pppd call 3g
```

## Smart bash commands

  1. List all files recursively in the directory /opt/ejabberd/ and replace in the filename string `ejabberd` by `ejabberd-17.09`.

  ```
  # for file in `find  /opt/ejabberd/ -type f`; do echo file found in /opt/ejabberd : ${file/ejabberd/ejabberd-17.09}; done
  ```

  2. Rename files rpmsave for example

  ```
  # ls *.conf.rpmsave | xargs basename -s .conf.rpmsave | xargs -I {} mv {}.conf.rpmsave {}.conf
  ```

## Let's Encrypt


**Centos 7**

```
# yum install certbot
# certbot certonly --standalone -d web-proxy-001.hammami.ch
# openssl x509 -in /etc/letsencrypt/live/web-proxy-001.hammami.ch/fullchain.pem -text -noout
# openssl rsa -in /etc/letsencrypt/live/web-proxy-001.hammami.ch/privkey.pem -noout -tex
# a link is better # cp /etc/letsencrypt/live/web-proxy-001.hammami.ch/privkey.pem /etc/ssl/certs/web-proxy-001.hammami.ch.privkey.pem
# a link is better # cp /etc/letsencrypt/live/web-proxy-001.hammami.ch/fullchain.pem /etc/ssl/certs/web-proxy-001.hammami.ch.fullchain.pem
# cd /etc/ssl/certs
# ln -s /etc/letsencrypt/live/web-proxy-001.hammami.ch/fullchain.pem web-proxy-001.hammami.ch.fullchain.pem
# ln -s /etc/letsencrypt/live/web-proxy-001.hammami.ch/privkey.pem web-proxy-001.hammami.ch.privkey.pem
# cat /etc/cron.d/certbot
30 2 * * * /usr/bin/certbot renew >> /var/log/le-renew.log

# fix let's encrypt certificates Permission
# chmod 755 /etc/letsencrypt/live/
# chmod 755 /etc/letsencrypt/archive/
```

**Centos 6**

```
# yum install centos-release-SCL
# yum install python27 python27-python-devel python27-python-setuptools python27-python-tools python27-python-virtualenv
# ln -s /opt/rh/python27/root/usr/lib64/libpython2.7.so.1.0 /usr/lib64/libpython2.7.so.1.0
# ln -s /opt/rh/python27/root/usr/lib64/libpython2.7.so.1.0 /usr/lib64/libpython2.7.so
# ll /usr/lib64/libpyt*
lrwxrwxrwx. 1 root root      19 Aug 22  2016 /usr/lib64/libpython2.6.so -> libpython2.6.so.1.0
-r-xr-xr-x. 1 root root 1669840 Aug 18  2016 /usr/lib64/libpython2.6.so.1.0
lrwxrwxrwx  1 root root      51 Mar  1 17:59 /usr/lib64/libpython2.7.so -> /opt/rh/python27/root/usr/lib64/libpython2.7.so.1.0
lrwxrwxrwx  1 root root      51 Mar  1 17:59 /usr/lib64/libpython2.7.so.1.0 -> /opt/rh/python27/root/usr/lib64/libpython2.7.so.1.0
# /opt/rh/python27/root/usr/bin/python2.7 -V
Python 2.7.8
# vim ~/.bash_profile
PATH=/opt/rh/python27/root/usr/bin/:$PATH:$HOME/bin
export PATH
# python -V
# source  ~/.bash_profile
# python -V
# git clone https://github.com/letsencrypt/letsencrypt
# cd /opt/letsencrypt/
# service nginx stop
# ./letsencrypt-auto certonly --standalone -d gate38-adqa.adeya.ch
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel):sysop@adeya.ch

-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf. You must agree
in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A

-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: N
Obtaining a new certificate
Performing the following challenges:
tls-sni-01 challenge for demo.adeya.ch
Waiting for verification...
Cleaning up challenges
Generating key (2048 bits): /etc/letsencrypt/keys/0000_key-certbot.pem
Creating CSR: /etc/letsencrypt/csr/0000_csr-certbot.pem

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/demo.adeya.ch/fullchain.pem. Your cert will
   expire on 2017-05-30. To obtain a new or tweaked version of this
   certificate in the future, simply run letsencrypt-auto again. To
   non-interactively renew *all* of your certificates, run
   "letsencrypt-auto renew"
 - If you lose your account credentials, you can recover through
   e-mails sent to sysop@adeya.ch.
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

# ls /etc/letsencrypt/live/
demo.adeya.ch
# ls -al /etc/letsencrypt/live/demo.adeya.ch/
total 12
drwxr-xr-x. 2 root root 4096 Mar  1 18:11 .
drwx------. 3 root root 4096 Mar  1 18:11 ..
lrwxrwxrwx. 1 root root   37 Mar  1 18:11 cert.pem -> ../../archive/demo.adeya.ch/cert1.pem
lrwxrwxrwx. 1 root root   38 Mar  1 18:11 chain.pem -> ../../archive/demo.adeya.ch/chain1.pem
lrwxrwxrwx. 1 root root   42 Mar  1 18:11 fullchain.pem -> ../../archive/demo.adeya.ch/fullchain1.pem
lrwxrwxrwx. 1 root root   40 Mar  1 18:11 privkey.pem -> ../../archive/demo.adeya.ch/privkey1.pem
-rw-r--r--. 1 root root  543 Mar  1 18:11 README
# vim /etc/nginx/nginx.conf
        ssl_certificate /etc/letsencrypt/live/demo.adeya.ch/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/demo.adeya.ch/privkey.pem;
# vim 
SLCertificateFile /etc/letsencrypt/live/packages.adeya.ch/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/packages.adeya.ch/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
ServerAlias packages.adeya.ch
SSLCertificateChainFile /etc/letsencrypt/live/packages.adeya.ch/chain.pem

# cat /etc/cron.d/certbot
30 2 * * * /root/src/letsencrypt/letsencrypt-auto renew >> /var/log/le-renew.log
```

## NTPd Centos 7 

### Enable NTP

```
# systemctl enable ntpd
# timedatectl status
# timedatectl set-ntp on
# timedatectl status
```

Two main packages are used in RHEL 7 to set up the client side:

- **ntp** this is the classic package, already existing in RHEL 6, RHEL 5, etc. It can be used both as a NTP client or server.
- **chrony** this is a new solution better suited for portable PC or machines with network connection problems (time synchronization is quicker). It can only be used as a NTP client. chrony is the default package in RHEL 7.

**Caution:** ntpd and chronyd shouldn’t run at the same time. Choose one and only one of them! There are reports from RHCE candidates noting that one of them is purposely already running at the beginning of the exam.

### Disable Chrony

Chrony is introduced as new NTP client to replace the ntp as the default time syncing package since RHEL7, so if you configure NTP during the installation process, it just enables the chronyd service, not ntpd service.


```
# systemclt status ntpd.service
ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled)
   Active: inactive (dead)
```

Even when you have enabled NTP to start on boot, it will not start when chrony is enabled. So to enable NTP to start on boot, we have to disable the chrony service

In case you want to use NTP only, then below is the procedure to do so :

Please follow steps below to enable NTP service on RHEL 7:

1. Disable chronyd service.

  To stop chronyd, issue the following command as root:

  ```
  # systemctl stop chronyd
  ```

  To prevent chronyd from starting automatically at system start, issue the following command as root:

  ```
  # systemctl disable chronyd
  ```

2. Install ntp using yum:

  ```
  # yum install ntp
  ```

3. Then enable and start ntpd service:

  ```
  # systemctl enable ntpd.service
  # systemctl start ntpd.service
  ```

4. Reboot and verify.

  ```
  # systemctl status ntpd.service
  ntpd.service - Network Time Service
     Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled)
     Active: active (running) since Fri 2015-01-09 16:14:00 EST; 53s ago
    Process: 664 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
    Main PID: 700 (ntpd)
     CGroup: /system.slice/ntpd.service
             └─700 /usr/sbin/ntpd -u ntp:ntp -g
  ```



## How can I instruct yum to install a specific version of package X?

https://unix.stackexchange.com/questions/151689/how-can-i-instruct-yum-to-install-a-specific-version-of-package-x

## OpenSSL

### Verifying that a Private Key Matches a Certificate

```
# openssl pkey -in /etc/letsencrypt/live/ejabberd-server-001.hammami.ch/privkey.pem -pubout -outform pem | sha256sum
0fb3f333fa4c9899ed0f2c43682812e65918f2f3427a0af2523e4cf19e64256f  -

# openssl x509 -in /etc/letsencrypt/live/ejabberd-server-001.hammami.ch/fullchain.pem -pubkey -noout -outform pem | sha256sum
0fb3f333fa4c9899ed0f2c43682812e65918f2f3427a0af2523e4cf19e64256f  -
```

## Yum transaction database 

* yum stores a sqlite database of information about each transaction. The history is organized terms of transaction ids and is updated every time a yum transaction affects the package configuration of the system. Mostly this database can be found in the `/var/lib/yum/history/` directory.

* The `yum history` command allows the user to view the history of transactions.

* The following command lists the history of all transactions :-

```
# yum history list all
```

* This will list the transaction ID along with the date and time, the actions performed and the number of packages altered :-

* For more information on a particular transaction, note the transaction ID for that transaction and use it in the below command :-

```
# yum history info <transaction_ID>
```

## How to bind to port number less than 1024 with non root access?

Using `CAP_NET_BIND_SERVICE` to grant low-numbered port access to a process :
For this, we just need to run following command in terminal :

```
# getcap /usr/sbin/kamailio
# setcap CAP_NET_BIND_SERVICE=+eip /usr/sbin/kamailio
# getcap /usr/sbin/kamailio
/usr/sbin/kamailio = cap_net_bind_service+eip
```
**NB** you have to execute this command again after updating kamailio because the kamailio have new binary file.

**WHY** The setcap on file store the capacities in an extended attribute with a call to setxattr, this extended attribute is stored like other attributes (ownership, rights...) in the filesyste

[getcap, setcap and file capabilities](https://www.insecure.ws/linux/getcap_setcap.html)


## Package manager and a Ruby Environment Manager On Mac

```
brew update
brew install ruby
# If you use bash
echo 'export PATH=/usr/local/Cellar/ruby/2.4.1_1/bin:$PATH' >> ~/.bash_profile 

# If you use ZSH:
echo 'export PATH=/usr/local/Cellar/ruby/2.4.1_1/bin:$PATH' >> ~/.zprofile
```

You can do that but I suggest using an Environment Manager for Ruby. You have rbenv and RVM.
IMO go for rbenv:

```
brew install rbenv ruby-build
# bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile  

# zsh
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zprofile
echo 'eval "$(rbenv init -)"' >> ~/.zprofile  

# list all available versions:
rbenv install -l

# install a Ruby version:
rbenv install 2.4.1

# set ruby version for a specific dir
rbenv local 2.4.1

# set ruby version globally
rbenv global 2.4.1

rbenv rehash
gem update --system
```

## Fix locale problems in CentOS 6 Linux

If you get this warning when running perl scripts:

```
# perl -v
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = "(unset)",
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
 
This is perl, v5.10.1 (*) built for x86_64-linux-thread-multi
```

then to solve the problem add the following line to ~/.bashrc or to /etc/profile:

```
export LC_ALL=C
```

## Edit a remote file over SSH

First, we have to mount the remote folder on OS X over SSH by installing: 

* OSXFUSE 2.7.3
* SSHFS 2.5.0

The easy way to install `SSHFS` is navigate to [http://osxfuse.github.io](http://osxfuse.github.io).

```
$ sshfs admin@media-server-xxx:/home/admin/rebtel-installer /Users/ohammami/rebtel-installer/
```

if you get this error:  `mount_osxfuse: the file system is not available (255)`

```
$ sudo kextunload -b com.apple.filesystems.smbfs
```
To check if folder is a mounted remote filesystem

```
$ df -P -t /Users/ohammami/rebtel-installer/
Filesystem                                          512-blocks    Used Available Capacity  Mounted on
admin@media-server-xxx:/home/admin/rebtel-installer   41907120 9646928  32260192    24%    /Users/ohammami/rebtel-installer
```
To unmount the folder:

```
$ diskutil unmount /Users/ohammami/rebtel-installer
```

## SSH Mac OS X config

To keep SSH connections alive even when they are idle: use the `ServerAliveInterval` to send data along every few seconds. 

To fix the setlocale warning: configure OpenSSH Client to not send the LC_* variables.

Edit `/etc/ssh/ssh_config` or `/etc/ssh_config` file, enter:

```
Host *
   #SendEnv LANG LC_*
   ServerAliveInterval 10
```
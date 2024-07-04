# ToFtp
local transfer script

**[Install FTP: proftpd server](https://github.com/hatimyra/ToFtp/edit/main/README.md#install-proftpd-on-server)**  
**[StartUP FileTransfer](https://github.com/hatimyra/ToFtp/edit/main/README.md#settings-filetransfer)**


# **Settings FileTransfer**

**Get FileTransfer pack:**
```
sudo su
mkdir /opt/service
cd /opt/service
git clone ***
```
**Config service**  
change file service 
```
nano filetransfer/transfer.service
cp /etc/systemd/system/
```

change rules folder 
```
chmod +xr -R filetransfer
```

change fields FileTransfer in config.json
```
nano filetransfer/config/config.json
```

start service 
```
systemctl enable transfer.service
systemctl daemon-reload
systemctl start transfer.service
```


# **Install @proftpd on server**
***Folder for sources***
```
sudo su
mkdir /opt/SOURCES
```

***Download and unzip***
```
cd /opt/SOURCES
wget ftp://ftp.proftpd.org/distrib/source/proftpd-1.3.8.tar.gz
tar -xvf proftpd-1.3.8.tar.gz
cd proftpd-1.3.8
```

***Next we will install GCC for compilation***
```
yum -y install gcc
```

***Compile sources into binary***

prepare output directory:
```
mkdir /opt/proftpd-1.3.8
```

configure: 
```
sudo ./configure --enable-openssl --with-modules=mod_digest
```

compiling:
```
make
make install
```

```
cd /opt/proftpd-1.3.8
```

_to check compiled-in modules:_
```
sbin/proftpd -l
and we will get as output:
```

**Config Proftpd:**

Create user and group and affect user to the group:

useradd proftpu -d / -s /bin/false
groupadd proftpg
usermod proftpu -g proftpg


Update PROFTPD configuration
Update proftpd.conf lines via nano etc/proftpd.conf:

```
nano etc/proftpd.conf
```

User                            proftpu
Group                           proftpg
If you don't want anonymous FTP logins comment out all <Anonymous ~ftp> block.

Force users default root to theirs home directory by adding this to global configuration:

> DefaultRoot ~  
> Enable mod_facts by adding this to global configuration:  
> \<IfModule mod_facts.c>  
>     FactsAdvertise on  
> \</IfModule>

_Create new user for your ftp server:_
```
useradd vuta
passwd vuta
```
_Create ftpusers file with  nano etc/ftpusers and put new user inside_
vuta

_Create startup service script_
```
nano /etc/init.d/proftpd
```
Add inside the content of below file (proftpd init.d service) Add permissions to run and reload systemctl:

```
chmod +x /etc/init.d/proftpd
systemctl daemon-reload
```
Now you can start your ftp server:
```
service proftpd start
```  

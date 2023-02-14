# Ubuntu VM installation guide

## install required software


execute the following command to updating your systems repositories
```apt-get update```

### Installing ftp-server
```apt-get install vsftpd```

### Installing web-server
```apt-get install apache2```

### Installing database-server
```apt-get install mariadb-server```

### Installing name-server
```apt-get install pdns-server```

```apt-get install pdns-backend-mysql```

### Installing netplan

```apt-get install```


## Configure Static Network Configuration
## Using netplan

GATEWAY_IP_ADDRESS=the ip address of your gateway<br>
GATEWAY_IP_ADDRESS="192.168.1.254"

HOST_IP_ADDRESS=the desired ip address for your host on your local network<br>
HOST_IP_ADDRESS="192.168.1.58/24"

ADAPTER_NAME=the name of your network adapter<br>
ADAPTER_NAME=enp0s3

To find your Network Device name write the following command
```ip a```

We add the following text using the following command
nano /etc/netplan/00-installer-config.yaml

```network:
  ethernets:
    ADAPTER_NAME:
      dhcp4: no
      addresses: [HOST_IP_ADDRESS]
      routes:
      - to: default
        via: GATEWAY_IP_ADDRESS
  version: 2
```
<br>

We test the configuration before apply our configuration with command
```netplan try```

<br>

to enable your configuration execute the following command
```netplan apply```


## Configure Firewall

```
ufw allow apache
ufw allow samba
ufw allow ssh
ufw allow bind9
ufw allow ftp
```

### Checking If Rules are applied

ufw status

netstat -pantul



## Configure powerdns




### Setting up Powerdns using mysql

To enable mysql backend we use the following command
```nano /etc/powerdns/pdns.d/pdns.local.gmysql.conf```


```
# MySQL Configuration
#
# Launch gmysql backend
launch+=gmysql

# gmysql parameters
gmysql-host=127.0.0.1
gmysql-port=3306
gmysql-dbname=pdns
gmysql-user=pdnsadmin
gmysql-password=g4398jf_fFAefw
gmysql-dnssec=yes

# gmysql-socket=
```



### Creating zones for each domain

```
pdnsutil create-zone domain

pdnsutil create-zone first.domain

pdnsutil create-zone second.domain

pdnsutil create-zone thirddomain
```

## Configure Web Server apache2


Add each user to group www-data
```
modprobe first www-data
modprobe second www-data
modprobe third www-data
```

Make each user's group www-data home directory 
chown first:www-data /home/first
chown second:www-data /home/second
chown third:www-data /home/third


we add the following text inside in order to have access
to each users home directory 

```
##Giving access to document root on /home dir of each user
<Directory /home/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```
We use the following to represent values in your domain

Represents your servers alias
YOUR_ALIAS=www.first.domain

Your domain 
YOUR_SRVNAME=first.domain

The domain's administrator email
YOUR_SRVADMIN=

The location where your site's data resides
YOUR_DOCROOT=/home/first/public_html



Insert the following for each domain using the following command 
```nano /etc/apache2/sites-available/first.domain.conf ```



```
<VirtualHost *:80>

        ServerAdmin YOUR_SRVADMIN
        ServerName YOUR_SRVNAME
        ServerAlias YOUR_ALIAS
        DocumentRoot /home/first/public_html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```


and then enable it using the following command

```
a2ensite  first.domain
a2ensite  second.domain
a2ensite  third.domain
```

## Setting up Samba





Create sambashare directory in each user's home directory
```mkdir /home/<username>/sambashare/```


Execute the following command to edit the following file
```nano /etc/samba/smb.conf```

Add this to the end of the file exit and save
```
[sambashare]
    comment = Samba on Ubuntu
    path = /home/username/sambashare
    read only = no
    browsable = yes
```

Restart samba to apply changes
```service smbd restart```


Assign a samba password for the user
```smbpasswd -a username```

To check that your samba host is working type the following command
```\\ip-address\sambashare```


###



Provide file-based storage
NFS exports and SMB files shares.

# NFS Server

Install the NFS package

```
yum install nfs-utils
```

Enable and start the nfs service.
```
systemctl enable nfs.service
systemctl start nfs.service
systemctl status nfs.service
```

Allow the NFS Service.

```
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload  
firewall-cmd --list-all  
```

Edit /etc/exports

```
/software 163.1.164.0/23(rw,sync)
```

/software		 
163.1.164.0/23
rw			 allow clients read write access to the share.
sync

Export the filesystem

```
exportfs -av
```

# NFS Client

# AUTOFS

```
yum install autofs
```


# SAMBA

Install the samba package

```
yum install samba
```

Samba Daemons and Services

smbd  
nmb  
winbind  

```
systemctl enable smb nmb
systemctl start smb nmb
```

Allow samba service.

```
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload  
firewall-cmd --list-all  
```

Configuring a Samba Server
The default configuration file /etc/samba/smb.conf

Samba uses /etc/samba/smb.conf as its configuration file.
If you change this configuration file, the changes do not take effect until you restart the Samba daemon
with the following command, as root:

```
systemctl restart smb.service
```

Prepare shared directories

/srv/ansys_docs		a read-only-share  
/srv/scratch		a read-write-share all-users  
/srv/devops_group 	a group share  

```
mkdir /srv/{ansys_docs,scratch,devops_group}
```

Make backup of smb.conf

``` 
cp /etc/samba/smb.conf /etc/samba/smb.conf.`date -I`
```

Configure the samba server

```
vi /etc/samba/smb.conf
```

```
[global]
	workgroup = RHCE
	security = user
	guest account = nobody
	map to guest = Bad user
	server string = RHCE Samba server sharing ansys_docs, scratch and samba_group
 	netbios name = RHCE

	passdb backend = tdbsam

	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw

```
read-only share 

```
[ANSYS_DOCS]
	comment = ANSYS_DOCS Read-Only-Samba-Share
	path = /srv/ansys_docs
	readonly = yes
	guest ok = yes
```


read-write share all users

```
mkdir /srv/scratch  
chmod -R 0755 /srv/scratch/  
chown -R nobody: /srv/scratch/  
```

```
[SCRATCH]
        comment = SCRATCH_SPACE all-users-read-write
        path = /srv/scratch
        browsable = yes
        guest ok = yes
        read only = no
```

group share read-write-for-devops-group   
We want to give read-only privileges for all users who are not members of the devops group    

```
groupadd -r devops
chgrp devops /srv/devops_group
chmod 2775 /srv/devops_group
```

```
useradd -s /sbin/nologin -G devops dev1
useradd -s /sbin/nologin dev2
````

```
smbpasswd -a dev1
New SMB passwd:
Retype new SMB password:
Added user dev1.

smbpasswd -a dev2
New SMB passwd:
Retype new SMB password:
Added user dev2.
```

```
[DEVOPS]
	comment = devops group share
	path = /srv/devops_group
	write list = @devops
``` 

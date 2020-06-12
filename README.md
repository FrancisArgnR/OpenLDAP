
![OpenLDAPlogo](https://manuelfrancoblog.files.wordpress.com/2017/10/openldap.png)

### Table of Contents
====================

* [Introduction](#introduction)
* [LDAP structure](#ldap-structure)
* [Basic components of LDAP](#basic-components-of-ldap)
* [OpenLDAP installation](#openldap-installation)
  * [Installation of the OpenLDAP server (Fedora)](#installation-of-the-openldap-server-fedora)
  * [Installation of the OpenLDAP server (Ubuntu)](#installation-of-the-openldap-server-ubuntu)
  * [Installation of the OpenLDAP client (Ubuntu)](#installation-of-the-openldap-client-ubuntu)
* [OpenLDAP management functionalities](#openldap-management-functionalities)
  * [Basic commands](#basic-commands)
  * [Web interface](#web-interface)
* [References](#references)

====================

## Introduction

LDAP (Lightweight Directory Access Protocol) is a collection of open source protocols used to access centralized remote hierarchical directory services on a network. A directory service is a shared information infrastructure that allows you to access, manage, or modify resources on a network such as users, groups, devices, and other objects. The most common use of LDAP is to use it as a centralized user account management system to authenticate users on the network and to store all information concerning these accounts. LDAP is based on a client/server protocol where directory data is stored on the LDAP server and is typically used to contain the stored information of users. This centralized storage of user accounts on a single server greatly simplifies system administration. More specifically, OpenLDAP is an open source implementation of LDAP that runs on Linux/UNIX systems.

## LDAP structure

An LDAP directory has a tree structure (Data Information Trees, DIT). This structure is composed by a group of entries (or objects) that are the LDAP information model and have a defined position within this hierarchy. The entries are uniquely identified by the full path in the tree to them, which is called the Distinguished Name (DN). Entries can contain other entries or be in a leaf node so they have no subordinate entries. Entries are composed of one or more object classes that contain attributes. 

The top of the DIT tree hierarchy consists of a root element. This element usually contains entries of type c (country), dc (domain component), or o (organization) as subordinate elements, and these in turn contain organizational entries, ou (organizational unit). The type of an entry is determined by the object class that determines which attributes the entry in question must or can be assigned. 

![LDAPStructure](https://docs.oracle.com/cd/E19182-01/820-6573/images/LDAP_Directory_Strucuture.gif) <br>
*Image obtained from: https://docs.oracle.com/cd/E19182-01/820-6573/6nht2e5a4/index.html*

## Basic components of LDAP

### Data Information Trees (DIT)
A directory information tree (DIT) is a hierarchical, tree-like structure that contains all the entries in the LDAP system stored as branches.

### Entries
An entry is a collection of information. Entries are made up of one or more class objects (or, more simply, a set of attributes).

### Attributes
The attributes are the lowest level elements of an LDAP system and are the ones that effectively contain the data (the other elements within LDAP are used for structure, organization, etc.). These attributes are made up of key-value pairs, where the keys have predefined names dictated by the objectClass selected for the entry.

### ObjectClass
Object classes are containers of attributes. They specify attribute groupings that describe particular entities (for example, 'person' is an Object Class). Each entry has a structural object class, which indicates which object type an entry represents, and you can also have further auxiliary object classes if additional attributes or characteristics are required for that entry.

Then to create an entry that describes a person, you have to include the objectClass person: <br> 
  _dn: . . ._ <br> 
  _objectClass: person_ <br> <br> 

This allows you to set all the attributes concerning the person within the entry: <br> 
  _cn: common name_ <br> 
  _sn: last name_ <br> 
  _userPassword: password for the user_ <br> 
  _…_ <br> 


## OpenLDAP installation

### Previous recommendations

Before deploying OpenLDAP it would be convenient to update the system packages (e.g. Fedora):

_$ dnf update_ <br>
_$ dnf upgrade_

It is recommended that the node name is in the domain where the server will be created. The name must follow an FDN (fully distinguished name) format.

### Installation of the OpenLDAP server (Fedora)

#### Installation

To __install__ OpenLDAP, it is necessary to install the openldap packages along with the server and client package:

_$ dnf -y install openldap openldap-servers openldap-clients_

To __enable__ (start automatically at boot time) and __start__ the OpenLDAP server service you have to run:

_$ systemctl enable slapd_ <br>
_$ systemctl start slapd_

After installation, the LDAP administrator password must be established. First you have to generate a hash for the password that will be used later. This is done with the following command:

_$ slappasswd_

Although you can also establish the password on the same line:

_$ slappasswd -h {SHA} -s password_

The OpenLDAP configuration files can be found in the _/etc/openldap/slapd.d_ directory, and although it is possible to modify them directly it is not recommended.

#### Configuration

To __configure__ the OpenLDAP database, first copy the database configuration by renaming it as follows:

_$ cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG_

And set the property of the LDAP database configuration directory to the ldap user.

_$ chown -R ldap:ldap /var/lib/ldap_

Next, the basic OpenLDAP schemas must be imported:

_$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif_ <br>
_$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif_ <br>
_$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif_ <br>

At this point we have to generate the configuration of our basic scheme. To do this we have to modify the configuration file of the OpenLDAP database to replace the default domain/suffix of the Directory Information Tree (DIT) with the one we want to have. To do this it is necessary to modify the values of the attributes: _olcSuffix_ (base domain), _olcRootDN_ (administrator user name) and _olcAccess_ (password access permission). The way to make these modifications is by creating an _.ldif_ file as shown below:

```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=admin,dc=example,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}PASSWORD

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=admin,dc=example,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=admin,dc=example,dc=com" write by * read
```

Then add the above configuration to the LDAP database with the command:

_$ ldapmodify -Y EXTERNAL -H ldapi:/// -f mod_domain.ldif_

The following command can be used to verify that the changes have been made correctly:

_$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={2}mdb -LLL_ <br>
ó <br>
_$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\* <br>
$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}monitor -LLL_ <br>

You can also check the configuration:

_$ slaptest -u_

_** Explanation of the previous file ** <br>
To configure the database, you must modify the backend of the primary database (/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif) and also the access control list for the LDAP monitor backend (olcDatabase\={1}monitor.ldif).
These modifications are not made directly on these files, instead they are made by generating an .ldif file containing the modifications. 
In the .ldif file, to identify the element on which you want to act, its DN is used. Then the first line in the file must be the DN: dn: olcDatabase={2}hdb,cn=config. In the next line you have to specify if you want to add or modify, this is done through: changeType: modify. Then you must specify the element to be replaced or if you want to delete it: replace: olcSuffix. And finally write the new value of the changed attribute: olcSuffix: dc=domain,dc=local._

#### LDAP structure generation

The next action to be taken is to generate the structure of our LDAP directory. To do this, first create the directory base (_dn: dc=example,dc=com_) and the administrator user (_dn: cn=admin,dc=example,dc=com_), and then add the organizational structures for the users (_dn: ou=users_) and for the user groups (_dn: ou=groups_). For each of these elements you have to specify a series of attributes, which depending on the type of object you create, you will have to specify one attribute or another. This, again, is done through an _.ldif_ file:

```
dn: dc=example,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: Example Com
dc: Example

dn: cn=admin,dc=example,dc=com
objectClass: organizationalRole
cn: admin
description: LDAP Directory Manager

dn: ou=People,dc=example,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=example,dc=com
objectClass: organizationalUnit
ou: Group
```

And to add the changes to the database:

_$ ldapadd -x -D cn=admin,dc=example,dc=com -W -f basedn.ldif_

Finally, the process of adding users and groups is again done from an .ldif file (this can be done in two different files):

```
dn: uid=usuario,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: First Name
sn: Last Name
uid: usuario
userPassword: {SSHA}MI/malE7t763EWw7YiRzXsojGETmqMJq
loginShell: /bin/bash
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/usuario

dn: cn=departamento,ou=Group,dc=example,dc=com
objectClass: posixGroup
cn: Departamento
gidNumber: 10000
member: cn=usuario,ou=users,dc=example,dc=com
```

And to be effective:

_$ ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_user.ldif_

As it is the case of the previous example, if you want to add the password to the user, previously you must create and encrypt the user's password and the SSHA obtained is passed to the previous process (although in some OS you can write it directly). 

_$ slappasswd_ <br> 
_$ New password: password_ <br> 
_$ Re-enter new password: password_ <br> 
_$ {SSHA}MI/malE7t763EWw7YiRzXsojGETmqMJq_ <br> 

Again to verify that the user has been created successfully:

_$ ldapsearch -x uid=usuario -b dc=example,dc=com -LLL_

To delete an entry, use the following command with the name of the entry to be deleted (where the final password is the admin password):

_$ ldapdelete "uid=usuario,ou=People,dc=example,dc=com" -D "cn=admin,dc=example,dc=com" -w password_ <br>
_$ ldapdelete -W "uid=usuario,ou=People,dc=example,dc=com" -D "cn=admin,dc=example,dc=com"_ <br>

Finally, the LDAP service must be allowed in the __firewall__:

_$ firewall-cmd --add-service={ldap,ldaps} --permanent <br>
$ firewall-cmd --reload_ <br>

ó <br>

_$ firewall-cmd --permanent --add-service=ldapfirewall-cmd -reload_ <br>


### Installation of the OpenLDAP server (Ubuntu)

#### Installation

If you are using Ubuntu as a system, the installation process is very similar. First of all you have to __install__ the slapd package and it is also convenient to install the ldap-utils package:

_$ sudo apt-get install slapd_ <br>
_$ sudo apt-get install ldap-utils_

During the installation of the package, the administration password will be requested in the LDAP directory.

The following commands are used to __start__ the OpenLDAP daemon, to __enable__ autostarting at server startup and to check the status:

_$ sudo systemctl start slapd_ <br>
_$ sudo systemctl enable slapd_ <br>
_$ sudo systemctl status slapd_ <br>

In addition, the __firewall must be opened__ to allow requests to the LDAP server daemon (previous _sudo ufw enable_):

_$ sudo ufw allow ldap_

#### Configuration

To __configure__ the LDAP database, copy the configuration file of the model database for slapd from the /var/lib/ldap directory:

_$ sudo cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG_

And establish the correct permissions for the file (and then restart the LDAP service):

_$ sudo chown -R ldap:ldap /var/lib/ldap/DB_CONFIG_ <br>
_$ sudo systemctl restart slapd_ <br>

Some basic LDAP schemes are then imported from the _/etc/openldap/schema directory_ as follows:

_$ sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif_ <br>
_$ sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif_ <br>
_$ sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif_ <br>

The next step is to generate the __configuration of our basic scheme__. To do this you need to add your domain/suffix to the LDAP database. This is done by generating an .ldif file (see the full explanation of the file in the previous Fedora installation process) where we modify the values of the attributes: _olcSuffix, olcRootDN_ and _olcAccess_. To add the configuration to the database, you have to run the command:

_$ sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ldapdomain.ldif_

** Alternative process **
Ubuntu offers an alternative configuration process to the previous one that can be performed automatically. To do so, you must execute the following command and enter a series of parameters:

_$ sudo dpkg-reconfigure slapd_

The automatic configuration already establishes the basis of the directory and the admin. Then it is not necessary to reload them when generating the structure with the basedn.ldif file. Otherwise, an error will be generated.

#### LDAP structure generation

The next action to be taken is to generate our LDAP directory structure (for details see again what was done in the Fedora installation above). And add the modification made to LDAP:

_$ sudo ldapadd -Y EXTERNAL -x -D cn=admin,dc=example,dc=com -W -f basedn.ldif_

Once the organizational structures are added, the first groups and users are then added using an _.ldif_ file (see these files in more detail in the previous Fedora installation):

_$ ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_user.ldif_

To add the password to the user, it is necessary to have previously generated the encrypted password to pass it in the _.ldif_ file.

_$ sudo useradd tecmint_ <br>
_$ sudo passwd tecmint_ <br>

#### Uninstallation

To uninstall the server is used:

_sudo apt-get remove slapd_ <br>
_sudo apt-get remove --purge slapd_ <br>
_sudo apt-get remove ldap-utils_ <br>
_sudo apt-get remove --purge ldap-utils_ <br>


### Installation of the OpenLDAP client (Ubuntu)

Once an OpenLDAP server is configured, the client needs to be installed and configured in order to connect. First, several packages need to be installed:

_$ sudo apt-get –y install libnss-ldap libpam-ldap ldap-utils nscd_ <br>
_$ sudo apt install libnss-ldap libpam-ldap ldap-utils nscd_ <br>

During the installation, the package installer will require a few parameters from the server for configuration (the ldap-auth-config package that is installed automatically does most of the configuration). You will be asked to enter the OpenLDAP server address, the domain name (dc: example, dc: com), the LDAP version, the administration account, etc. If during the installation process we detect any error or we have introduced some parameter incorrectly, we can always do it again by executing the command:

_$ sudo dpkg-reconfigure ldap-auth-config_

The installer does much of the configuration, but there are still some details to make LDAP authentication work, so we will need to adjust the behavior of the NSS and PAM services. First you have to modify the NSSwitch configuration (in the _/etc/nsswitch.conf file_) to use LDAP as a data source by adding the lines:

```
passwd:         compat ldap
group:          compat ldap
shadow:         compat ldap
```

This can also be done automatically as follows:

_$ sudo auth-client-config -t nss -p lac_ldap_

Next, the use of LDAP for authentication is configured by updating the PAM settings:

_$ sudo pam-auth-update_

Finally, if you want to establish a home directory to be created automatically with the first login of the user, you have to add the following lines to the _/etc/pam.d/common-session_ file:

```
session required pam_mkhomedir.so skel=/etc/skel umask=077
```

In case the previous option does not work, add in _/usr/share/pam-configs/mkhomedir_:

```
Name: activate mkhomedir
Default: yes
Priority: 900
Session-Type: Additional
Session:
required pam_mkhomedir.so umask=0022 skel=/etc/skel
```

The NCSD service is then restarted:

_$ sudo systemctl restart nscd_ <br>
_$ sudo systemctl enable nscd_ <br>

To verify that the client configuration has been done correctly, check that a user can be obtained from the server:

_$ getent passwd user_


## OpenLDAP MANAGEMENT FUNCTIONALITIES

### Basic commands

The most commonly used OpenLDAP commands include the following:
   - ldapbind: used to authenticate to a directory server.
   - ldapsearch: used to search for specific entries in a directory.
   - ldapadd: used to add entries to a directory.
   - ldapdelete: used to delete entries in a directory.
   - ldapmodify: used to modify existing entries in a directory.
   - ldapmoddn: used to change the name of an entry or to move a subtree to another location in the directory.
   - ldappasswd: used to modify passwords.

The above commands contain the following common parameters, among others:
   - -D: The DN.
   - -W: Prompt for the password of the DN.
   - -x: Use of simple authentication.
   
Some specific parameters of each command are as follows:
   - ldappasswd:
       - -S: Prompt for the new password.

Useful examples:
   - ldapsearch:
       - Search for the entire structure of the DIT: _$ ldapsearch -x -b "dc=example,dc=com"_
       - Search for a concrete ou (organizational unit): _$ ldapsearch -x -b "ou=People,dc=example,dc=com"_
       - Search for a concrete user: _$ ldapsearch -x -b "uid=usuario,ou=People,dc=example,dc=com"_
   - ldappasswd: 
       - Set a new password for a user: _$ ldappasswd -x -D "cn=admin,dc=example,dc=com" -W -S "uid=usuario,ou=People,dc=example,dc=com"_

_ ** If the search command is not executed on the server, i.e. it is executed on the client, then the server host must be specified with the_ -H ldap://dirIP

### Web interface

As seen during the installation processes, LDAP allows you to work with commands and _.ldif_ files to create and modify elements of the LDAP directory, but this can sometimes be a little difficult. LDAP offers LDAP directory browsers that make this task easier. Here we can highlight __phpldapadmin__, which is a graphical administration tool for managing LDAP servers. It can be installed as follows, which will enable the necessary Apache settings:

_(Fedora) $ yum -y install phpldapadmin_ <br>
_(Ubuntu) $ sudo apt-get -y install phpldapadmin_

Once installed, it is necessary to make some small configuration changes to use our domain. To do this you must edit the file _/etc/phpldapadmin/config.php_ and change the following lines:

```
$servers->setValue('server','name','OpenLDAP Server Name or IP');
$servers->setValue('server','base', array('dc=example,dc=com'));
#$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
$config->custom->appearance['hide_template_warning'] = true;
```

The first line refers to the name of your server, the second to which is the root of the directory, the third line should be commented as it auto-fills the admin login in the web interface, while the last one controls the visibility of some warning messages.

Now that the web interface is configured, the way to access it from the web browser is as follows:

_$ firefox http://dir_ip/phpldapadmin &_

Where a home page appears for logging in. The login DN is the user name that is used, containing the account name (cn), and the server domain name (dc). 

![web](https://tr1.cbsistatic.com/hub/i/r/2016/12/01/0a8ad7b4-1684-4295-84c8-64bfe33ee6b9/resize/1200x/a773679c11ab780c1513b5a5d3d342d1/ldapb.jpg)
*Image obtained from: https://www.techrepublic.com/article/how-to-install-and-configure-ldap-and-phpldapadmin/*

Once connected you have the possibility to add users, organizational units, groups and relationships.

_** Possible appearance of errors on the web ** <br>
Errors (deprecated autoload() & deprecated create_function()) may be displayed when running the website. These errors will impede the correct use of the management website. The solution to these errors is detailed in:_ https://stackoverflow.com/questions/50698477/cant-create-new-entry-phpldapadmin 

## References

LDAP definition <br>
https://www.zytrax.com/books/ldap/ch2/index.html <br>
https://www.pks.mpg.de/~mueller/docs/suse10.1/suselinux-manual_en/manual/sec.ldap.tree.html <br>
https://www.digitalocean.com/community/tutorials/understanding-the-ldap-protocol-data-hierarchy-and-entry-components <br>
https://github.com/manuparra/docker_ldap <br>
https://www.digitalocean.com/community/tutorials/how-to-configure-openldap-and-perform-administrative-ldap-tasks <br>

Server installation <br>
https://likegeeks.com/es/servidor-ldap-de-linux/ <br>
https://kifarunix.com/install-and-configure-openldap-server-on-fedora-29/ <br>
https://www.server-world.info/en/note?os=Fedora_28&p=openldap&f=1 <br>
https://www.tecmint.com/install-openldap-server-for-centralized-authentication/ <br>
https://instructorbenyblanco.wordpress.com/configuracion-basica-de-servidor-ldap/ <br>
https://www.techrepublic.com/article/how-to-install-openldap-on-ubuntu-18-04/ <br>

Client installation <br>
https://www.tecmint.com/configure-ldap-client-to-connect-external-authentication/ <br>
https://computingforgeeks.com/how-to-configure-ubuntu-18-04-ubuntu-16-04-lts-as-ldap-client/ <br>
https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/configure-ldap-client-on-ubuntu-16-04-debian-8.html <br>
https://www.howtoforge.com/set-up-openldap-client-on-debian-10/ <br>
http://somebooks.es/12-9-configurar-un-equipo-cliente-con-ubuntu-para-autenticarse-en-el-servidor-openldap/ <br>

Web interface <br>
https://www.alibabacloud.com/blog/how-to-install-openldap-and-phpldapadmin-on-ubuntu-16-04_594318 <br>
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-openldap-and-phpldapadmin-on-ubuntu-16-04 <br>

OpenLDAP commands <br>
https://docs.oracle.com/cd/B10501_01/network.920/a96579/comtools.htm <br>
https://manpages.debian.org/jessie/ldap-utils/index.html <br>

Others <br>
https://www.thegeekstuff.com/2015/02/openldap-add-users-groups/ <br>
https://tylersguides.com/guides/openldap-how-to-add-a-user/ <br>
https://tylersguides.com/guides/how-to-change-an-openldap-password/ <br>
https://simp.readthedocs.io/en/master/user_guide/User_Management/LDAP.html <br>
https://www.techrepublic.com/article/how-to-populate-an-ldap-server-with-users-and-groups-via-phpldapadmin/ <br>
https://devconnected.com/how-to-search-ldap-using-ldapsearch-examples/ <br>
https://www.youtube.com/watch?v=fPcIfftZTps <br>
https://www.youtube.com/watch?v=Zmj6A5ggcgg&t <br>
https://www.youtube.com/watch?v=l0e8rG0mku8 <br>

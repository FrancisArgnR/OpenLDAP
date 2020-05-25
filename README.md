
UNDER CONSTRUCTION

![OpenLDAPlogo](https://manuelfrancoblog.files.wordpress.com/2017/10/openldap.png)

### Table of Contents
====================

* [Introduction](#introduction)
* [LDAP structure](#ldap-structure)
* [Basic components of LDAP](#basic-components-of-ldap)
* [OpenLDAP installation](#openldap-installation)
  * [Installation of the OpenLDAP server (Fedora)](installation-of-the-openldap-server-fedora)
  * [Installation of the OpenLDAP server (Ubuntu)](installation-of-the-openldap-server-ubuntu)

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

_$ dnf update_
_$ dnf upgrade_

### Installation of the OpenLDAP server (Fedora)

To __install__ OpenLDAP, it is necessary to install the openldap packages along with the server and client package:

_$ dnf -y install openldap openldap-servers openldap-clients_

To __enable__ (start automatically at boot time) and __start__ the OpenLDAP server service you have to run:

_$ systemctl enable slapd_
_$ systemctl start slapd_

After installation, the LDAP administrator password must be established. This is done with the following command:

_$ slappasswd_

Although you can also establish the password on the same line:

_$ slappasswd -h {SHA} -s password_

The OpenLDAP configuration files can be found in the _/etc/openldap/slapd.d_ directory, and although it is possible to modify them directly it is not recommended.

To __configure__ the OpenLDAP database, first copy the database configuration by renaming it as follows:

_$ cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG_

And set the property of the LDAP database configuration directory to the ldap user.

_$ chown -R ldap:ldap /var/lib/ldap_

Next, the basic OpenLDAP schemes must be imported:

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

_$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={2}mdb -LLL_
ó
_$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase=\*
$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config olcDatabase={1}monitor -LLL_

You can also check the configuration:

_$ slaptest -u_

_** Explanation of the previous file **
To configure the database, you must modify the backend of the primary database (/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif) and also the access control list for the LDAP monitor backend (olcDatabase\={1}monitor.ldif).
These modifications are not made directly on these files, instead they are made by generating an .ldif file containing the modifications. 
In the .ldif file, to identify the element on which you want to act, its DN is used. Then the first line in the file must be the DN: dn: olcDatabase={2}hdb,cn=config. In the next line you have to specify if you want to add or modify, this is done through: changeType: modify. Then you must specify the element to be replaced or if you want to delete it: replace: olcSuffix. And finally write the new value of the changed attribute: olcSuffix: dc=domain,dc=local._

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
cn: Nombre
sn: Usuario
userPassword: {SSHA}QLXFlVsiNY7bLgcwx8yurJqMZVaErD9b
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

_$ ldapadd -x -D cn=adminr,dc=example,dc=com -W -f add_user.ldif_

As it is the case of the previous example, if you want to add the password to the user, previously you must create and encrypt the user's password and the SSHA obtained is passed to the previous process (although in some OS you can write it directly). 

Again to verify that the user has been created successfully:

_$ ldapsearch -x uid=amosm -b dc=example,dc=com -LLL_

To delete an entry, use the following command with the name of the entry to be deleted:

_$ ldapdelete "cn=user,ou=users,dc=example,dc=com" -D cn=admin,dc=example,dc=com -w password_

Finally, the LDAP service must be allowed in the __firewall__:

_$ firewall-cmd --add-service={ldap,ldaps} --permanent
$ firewall-cmd --reload_

ó

_$ firewall-cmd --permanent --add-service=ldapfirewall-cmd -reload_


### Installation of the OpenLDAP server (Ubuntu)

### Installation of the OpenLDAP client (Ubuntu)

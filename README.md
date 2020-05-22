
UNDER CONSTRUCTION

![OpenLDAPlogo](https://manuelfrancoblog.files.wordpress.com/2017/10/openldap.png)

### Table of Contents
====================

* [Introduction](#introduction)
* [LDAP structure](#ldap-structure)
* [Basic components of LDAP](#basic-components-of-ldap)

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

Then to create an entry that describes a person, you have to include the objectClass person:  
  dn: . . .
  objectClass: person

This allows you to set all the attributes concerning the person within the entry:
  cn: common name
  sn: last name
  userPassword: password for the user
  …

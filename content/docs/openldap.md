---
title: OpenLDAP 
prev: /docs/selfhosting/nextcloud
---

### Directory Service

{{< callout type="info" >}}
[Difference between a folder and a directory](https://stackoverflow.com/questions/5078676/what-is-the-difference-between-a-directory-and-a-folder)
Folder is for grouping items.
Directory has index. It is for finding specific item,Directory is a filesystem concept.
In simple terms think ```directory``` like a telephone directory which is in a hierarchial structure.
{{< /callout >}}

The term directory service refers to the collection of software, hardware, and processes that store information about an enterprise, subscribers, or both, and make that information available to users. A directory service consists of at least one instance of Directory Server and at least one directory client program. Client programs can access names, phone numbers, addresses, and other data stored in the directory service.

A directory is similar to database,which is attribute-based data;where data is read more often than write.

Directory Server provides Global Directory Services which means it provides information to wide variety of applications,rather than using databases with different applications,which is very hard to administrate.Directory server is a single solution to manage the same information
{{<callout type="info">}}
For example, an organization has three different applications running like nextcloud,email and matrix server and all the applications are accessed by same credentials,if separate database schema's are used for each application it would be hard to manage,if user requesting a password change in one application maybe not be replicated into another application;this problem is solved single,centralized repository of directory information.
{{</callout>}}

LDAP provides a common language that client applications and servers use to communicate with one another. LDAP is a "lightweight" version of the Directory Access Protocol (DAP)

### RDBMS vs Directory Service

{{% details title="1. How often does your data change?"%}}
Directory servers are used for ```reads```,if your data changes often and have many write operations directory service is not a ideal choice,RDBMS would be the ideal choice.
{{% /details %}}


{{% details title="2. Type of Data? "%}}
If data is defined in ```Key:Value``` pair or ```Attribute:Value``` pair, Directory service would be the best choice,like user profile.
{{% /details %}}

{{% details title="3. Data in Hierarchial tree like structure" %}}
If data can be modeled into a tree like structure,accessing the parent and child node in the tree,directory service 
{{% /details %}}

### OpenLDAP
LDAP stands for Lightweight Directory Access Protocol, for accessing directory services.OpenLDAP is the implementation of the LDAP protocol,is a communications protocol that defines the methods in which a directory service can be accessed. 
The LDAP information model is based on entries, which is a collection of attributes that has a globally unique Distinguished Name```(DN)```
OpenLDAP is the implementation of the LDAP protocol which belong to User Management and Authentication in tech.

The LDAP protocol both authenticates and authorize's users to their resources.The protocol authenticates users with a bind operation that allows users to communicate with LDAP directory
then authorizes the authenticated user to resources they need if they have access that are defined in rules.Once a user is successfully authenticated, they need to be authorized to access the resource(s) requested. 
With OpenLDAP, for example, users belong to groups that can be assigned different permissions. If the authenticating user is assigned the correct permissions to access a certain resource, the LDAP protocol will authorize them to it; if not, the protocol will deny access.
#### LDAP Data components

1. Directory: an LDAP server

2. DIT: the tree of entries stored within a directory server

3. Attributes 

Data in LDAP system is stored in elements called attributes,like Key Value pair.Data in the attribute must match to the type defined in the attribute's initial declaration.

```bash
mail: user@example.com
dc:example,dc:com
```

4. Entries 

Attributes by themselves are not useful, a group or collection of ```attributes``` under a name represents an entry.
```bash
dn: ou=people,dc=example,dc=com
objectClass: person
sn: Ramesh
cn: Varma
```
An example entry displayed in LDIF ( LDAP Data Interchange Format).
```bash
$ cat ldif/user.ldif 
dn: uid=vinay.m,ou=People,dc=vinay,dc=im
objectClass: top
objectClass: inetOrgPerson
uid: vinay.m
cn: vinay
sn: m
userPassword: test
ou: People

dn: uid=akshay,ou=People,dc=vinay,dc=im
objectClass: top
objectClass: inetOrgPerson
uid: akshay
cn: akshay 
sn: p
userPassword: test
ou: People
```

5. ObjectClass

Object class: a collection of required (MUST) and optional (MAY) attributes. Object classes are further subdivided into STRUCTURAL and AUXILIARY classes, and they may inherit from each other.Every entry has a structural Object class which indicates what type of object an entry is and also can have more auxiliary object that have additional characteristics for that entry.

 The ObjectClass definitions are stored in the schema files.Object class must have an object identifier (OID)  Object classes may also list a set of required attribute types (so that any entry with that object class must also include those attributes) and/or a set of optional attribute types (so that any entry with that object class may optionally include those attributes).OID's are sequence of numbers separated by periods(.), “1.2.840.113556.1.4.473” 

6. Schema 

  Schema's define the directory, specifying the configuration of the directories including syntax,object classes,attribute types and matching rules.

#### slapd - Standalone LDAP Daemon

```slapd``` is a LDAP directory server,which stands for Standalone LDAP daemon.Providing simple auth and security layer.

```bash
$ sudo apt install slapd ldapvi ldap-utils
```

{{< callout type="warning" >}}
 when asked for administration password prompt during installation just press ```Enter```,we reconfigure slapd using dpkg-reconfigure after the installation.
{{< /callout >}}

```bash
$sudo dpkg-reconfigure slapd
```
{{< callout type="info" >}}
Reconfiguration: 
1. Omit initial LDAP server config : ```No``` we obviously want to create intial configuration.  
2. DNS Domain Name : domain name to build the base DN of LDAP directory in this case we are choosing ```vinay.im```.  
3. Organization Name: Type down the organization name( here XYZ Pvt Ltd)
5. Choose an Admin Password of your choice( for tutorial purpose i've choosed test) and choose MDB as backend database
6. If asked to purge database when slapd is removed we choose ```No```,will be helpful when we want to switch to a different LDAP server. 
7. Choose Yes if you want to backup the current existing database to ```/var/backups```.

{{< /callout >}}

To have a look at the LDAP database , simple execute ```slapcat``` with sudo privileges.
```bash{filename="$ sudo slapcat"}
$ sudo slapcat
dn: dc=vinay,dc=im
objectClass: top
objectClass: dcObject
objectClass: organization
o: XYZ Pvt Ltd
dc: vinay
structuralObjectClass: organization
entryUUID: 8057316c-ed6e-103d-8b93-b9da23579469
creatorsName: cn=admin,dc=vinay,dc=im
createTimestamp: 20230922083350Z
modifiersName: cn=admin,dc=vinay,dc=im
modifyTimestamp: 20230922083350Z
```
{{< callout type="info" >}}
Config files are present in ```/etc/ldap``` directory.   
 Schemas can be added within the ```slap.d``` directory for server customization.  
 Database is stored in ```/var/lib/ldap``` having two files ```data.mdb``` and ```lock.mdb```.
{{< /callout >}}

```bash
$ sudo cp /usr/share/doc/slapd/examples/slapd.conf /etc/ldap/
```
Copy the example config file ```slapd.conf``` to ```/etc/ldap```, and replace DNS domain components ```dc=example``` to ```dc=vinay``` and ```dc=com``` to ```dc=im``` everywhere in the config, also
update ```/etc/default/slapd``` from  ```SLAPD_CONF``` to ```SLAPD_CONF=/etc/ldap/slapd.conf``` and update slapd service by ```sudo systemctl restart slapd``` 

In ```/etc/ldap/slapd.conf``` under ```suffix       "dc=vinay,dc=com"``` add the following lines 
```bash
rootdn          "cn=admin,dc=vinay,dc=com"
rootpw          "test"
```
Restart the slapd service again.
```bash
$ sudo systemctl restart slapd
```
#### ldapsearch

```bash{filename="ldapsearch anonymous query"}
$ldapsearch -x -b "dc=vinay,dc=im"
# vinay.im
dn: dc=vinay,dc=im
objectClass: top
objectClass: dcObject
objectClass: organization
o: XYZ Pvt Ltd
dc: vinay

# search result
search: 2
result: 0 Success

# numResponses: 2
```
```bash{filename="ldapsearch authenticating with admin user"}
$ ldapsearch -D cn=admin,dc=vinay,dc=im -w test -b dc=vinay,dc=im
# extended LDIF
#
# LDAPv3
# base <dc=vinay,dc=im> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# vinay.im
dn: dc=vinay,dc=im
objectClass: top
objectClass: dcObject
objectClass: organization
o: XYZ Pvt Ltd
dc: vinay

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```
1.  ```-D``` {dn} / ```--bindDN``` {dn} — The DN to use to bind to the directory server when performing simple authentication,to use the distinguished binddn name to bind the LDAP directory.
2. ```-w``` - this option is used to provide the password on the command line for auth, ```-W``` option is used to ask for prompt for typing invisible password without actualling having to type the pass on cli.
3. ```-b``` - search base as the starting point for the search instead of default.
4. ```-x``` option in ldapsearch is used for simple authentication instead of SASL.

The above command search's through the ldap directory server with ```admin``` distinguished name providing password with the ```-w``` option and setting the searchbase to start from the rootdn.

-- To list all users on ldap
```bash
$ ldapsearch -D "cn=admin,dc=vinay,dc=com" -W -b "dc=vinay,dc=com"
$ slapcat
```
lists all users from the base dn

#### Adding OU (Organization Unit)

Organizational units (OUs) are used to organize entries within the directory tree and can be used to delegate administrative responsibilities within your organization. It’s important to keep your directory organized and well-structured from the beginning; otherwise it will quickly become unwieldy and difficult to manage.

Create a directory called ldif(LDAP Interchange Format) in ```/etc/ldap``` and create a file called people.ldif and paste the following contents.
```bash
$ cat /etc/ldap/ldif/people.ldif
dn: ou=People,dc=vinay,dc=com
ou: People
cn: people
sn: people
objectClass: top
objectClass: inetOrgPerson

$ ldapadd  -D cn=admin,dc=vinay,dc=im -w test -f /etc/ldap/ldif/people.ldif 
adding new entry "ou=People,dc=vinay,dc=im"
```
now ```slapcat``` command shows the OU added within the command output. 

#### Add new User

Adding new user within the newly created OU(Organizational Unit)
 
 ```bash{filename="/etc/ldap/john.ldif"}
 # cat john.ldif 
dn: uid=john,ou=People,dc=vinay,dc=com
objectClass: top
objectClass: inetOrgPerson
uid: john
cn: john
ou: People
sn: abraham
mail: john@vinay.com
userPassword: john
```
Adding the .ldif file using ldapadd command
```
$ sudo ldapadd -D "cn=admin,dc=vinay,dc=com" -W -f john.ldif 
Enter LDAP Password: 
adding new entry "uid=john,ou=People,dc=vinay,dc=com"

```

#### Read entries within OU as admin

Now we have added an ```OU``` and a user ```john``` to ```People``` OU,lets try to ```ldapsearch``` the users within the OU as admin
```bash{filename="display users of an OU as admin"}
$ ldapsearch -D "cn=admin,dc=vinay,dc=com" -w vinay.com -b "ou=People,dc=vinay,dc=com"

# extended LDIF
#
# LDAPv3
# base <ou=People,dc=vinay,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# People, vinay.com
dn: ou=People,dc=vinay,dc=com
ou: People
cn: people
sn: people
objectClass: top
objectClass: inetOrgPerson

# john, People, vinay.com
dn: uid=john,ou=People,dc=vinay,dc=com
objectClass: top
objectClass: inetOrgPerson
uid: john
cn: john
ou: People
ou: Support
sn: abraham
mail: john@vinay.com
userPassword:: am9obg==

```
#### Read entries within OU as normal user.

```bash
$ ldapsearch -D "uid=john,ou=People,dc=vinay,dc=com" -w john -b "ou=People,dc=vinay,dc=com"
# extended LDIF
#
# LDAPv3
# base <ou=People,dc=vinay,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# People, vinay.com
dn: ou=People,dc=vinay,dc=com
ou: People
cn: people
sn: people
objectClass: top
objectClass: inetOrgPerson

# john, People, vinay.com
dn: uid=john,ou=People,dc=vinay,dc=com
objectClass: top
objectClass: inetOrgPerson
uid: john
cn: john
ou: People
ou: Support
sn: abraham
mail: john@vinay.com
userPassword:: am9obg==

```

#### Modifying existing entries

Now to modify an already added record we use ldapmodify and the attributes that are to be modified are put into a separate file,here ```john-modify.ldif``` and to demonstrate here an OU ```Support```
is added to the existing entry,along with ```People``` OU.

```bash{filename="john-modify.ldif"}
$  cat /etc/ldap/ldif/john-modify.ldif 
dn: uid=john,ou=People,dc=vinay,dc=com
changetype: modify
add: ou
ou: Support
```

```bash{filename="ldapmodify command for john-modify.ldif"}\
$ ldapmodify -D "cn=admin,dc=vinay,dc=com" -W -f john-modify.ldif
Enter LDAP Password: 
modifying entry "uid=john,ou=People,dc=vinay,dc=com"
```

Now running a slapcat command shows the updated OU ```Support```

```bash{linenos=table}
dn: uid=john,ou=People,dc=vinay,dc=com
objectClass: top
objectClass: inetOrgPerson
uid: john
cn: john
ou: People
ou: Support
sn: abraham
mail: john@vinay.com
userPassword:: am9obg==
structuralObjectClass: inetOrgPerson
entryUUID: 50ea0ea8-f23d-103d-816b-4d9c39504958
creatorsName: cn=admin,dc=vinay,dc=com
createTimestamp: 20230928112421Z
entryCSN: 20230928120656.291224Z#000000#000#000000
modifiersName: cn=admin,dc=vinay,dc=com
modifyTimestamp: 20230928120656Z

```
[Reference serverfault] https://serverfault.com/questions/290296/ldapadd-ldapmodify-clarifications-needed-about-these-commands
#### Verifying the ```slapd.conf``` Configuration file

```bash
$sudo  slaptest -v -f /etc/ldap/slapd.conf 
config file testing succeeded
```

```-f``` : Specifying an alternative configuration file.

```-v``` : enable verbose mode.

#### Conventions in OpenLDAP

dn - Distinguished Name  
RDN - Relative Distinguished Name  
cn - Common Name  
dc - Domain Component  
mail - Email Address  
ou - Organization Unit  
ldif - LDAP Data Interchange Format  
ldap - Lightweight Directory Access Protocol  

### References

1. https://access.redhat.com/documentation/en-us/red_hat_directory_server/11/html/deployment_guide/introduction_to_directory_services

2. https://www.zytrax.com/books/ldap

3. https://tylersguides.com/guides/openldap-how-to-add-a-user/

4. https://www.zytrax.com/books/ldap/

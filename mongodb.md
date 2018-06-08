# MongoDB Enterprise
## Configuration
MongoDB Enterprise supports authentication and authorization of users against LDAP, including support to Active Directory.

### Virtual Machine
The first requirement is to have MongoDB Enterprise running in a VM in Azure. To achieve this, you need to create a Linux VM in Azure and then install MongoDB Enterprise in it.

It's recommended to review the Linux distributions MongoDB supports on their official documentation, but you could start creating a CentOS 7.4 from the [Azure Gallery](https://portal.azure.com/?feature.customportal=false#create/RogueWave.CentOSbased74-ARM). The VM must be created on the same VNet Azure Active Directory Domain Services was enabled.

Once the VM is running, the next step is to install MongoDB Enterprise in it. MongoDB has guidance for several [Linux distributions](https://docs.mongodb.com/manual/administration/install-enterprise-linux/), but the CentOS guidance is available [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-enterprise-on-red-hat/).

### MongoDB Configuration
Once MongoDB Enterprise is running, you need to configure it to authenticate and authorize users against LDAP.
Connect to the MongoDB server using mongo shell:

```
mongo --host <hostname> --port <port>
```

If a SSH connection was stablished with the VM on Azure, you can simple connect to the localhost and default port:

```
mongo
```

In order to manage MongoDB users using AD, you need to create at least one role on the admin database that can create and manager roles.
```
use admin
db.createRole(
    {
        role: "<Directory Distinguished Name>",
        privileges: [],
        roles: [ "userAdminAnyDatabase" ]
   }
)
```

A good approach is to create a group on AAD and give permission to this group on MongoDB. You can then add users to this group to grant them admin access to MongoDB. Example:
```
use admin
db.createRole(
    {
        role: "CN=MongoDBUsers,OU=AADDC Users,DC=rgodinhoad,DC=onmicrosoft,DC=com",
        privileges: [],
        roles: [ "userAdminAnyDatabase" ]
   }
)
```

The next step is to modify MongoDB's configuration file, by default located on /etc/mongod.conf.

```
#security:
security:
  authorization: "enabled"
  ldap:
    authz:
      queryTemplate: "<Query Template>"
    transportSecurity: "none"
    servers: "<AAD Domain Name>"
    bind:
      queryUser: "<Query User>"
      queryPassword: "<Password>"
setParameter:
  authenticationMechanisms: 'PLAIN'
```

Example:
```
#security:
security:
  authorization: "enabled"
  ldap:
    authz:
      queryTemplate: "DC=rgodinhoad,DC=onmicrosoft,DC=com??sub?(member:1.2.840.113556.1.4.1941:={USER})"
    transportSecurity: "none"
    servers: "rgodinhoad.onmicrosoft.com"
    bind:
      queryUser: "mongodbuser@rgodinhoad.onmicrosoft.com"
      queryPassword: "<password>"
setParameter:
  authenticationMechanisms: 'PLAIN'
```

The final step is to start MongoDB. This is done by executing:

```
sudo mongod -f /etc/mongodb.conf
```

An alternative approach would be to run MongoDB as a service, but a communication issue was detected between MongoDB and the LDAP endpoint from AAD. An investigation needs to be done to identify the root cause of the issue.

## Testing
To test the configuration, just connect to the MongoDB using Mongo Shell with an AAD credential:

```
mongo --username "<Directory Distinguished Name>" --authenticationMechanism 'PLAIN' --authenticationDatabase '$external' --password <password>
```

Example:
```
mongo --username "CN=mongodbuser,OU=AADDC Users,DC=rgodinhoad,DC=onmicrosoft,DC=com" --authenticationMechanism 'PLAIN' --authenticationDatabase '$external' --password <password>
```

Everything should be properly configured if it is possible to list the databases:

```
MongoDB Enterprise > show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

## References

[Authenticate and Authorize Users Using Active Directory via Native LDAP](https://docs.mongodb.com/manual/tutorial/authenticate-nativeldap-activedirectory/) - MongoDB documentation about Active Directory
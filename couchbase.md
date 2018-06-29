# Couchbase Enterprise
## Configuration
Couchbase Enterprise supports authentication and authorization of users against LDAP, including support to Active Directory.

### Virtual Machine
The easiest way to use Couchbase Enterprise in Azure is to deploy an enviroment using one of their offers on Azure Marketplace:

- [BYOL](https://portal.azure.com/?feature.customportal=false#create/couchbase.couchbase-enterprisecouchbase-enterprise-byol)
- [Hourly Pricing](https://portal.azure.com/?feature.customportal=false#create/couchbase.couchbase-enterprisecouchbase-enterprise-hourly-pricing).

It's a requirement the VMs must communicate with the LDAP endpoints from Active Directory Domain Services, but since Couchbase offer on Azure Marketplace does not support deploying the cluster to an existing VNet, you have some options regarding this configuration:

- First deploy the Couchbase Cluster using an Azure Marketplace offer, and then configure Domain Services on the VNet that was created by the Couchbase deployment. You may need to restart the Couchbase VMs after updating VNet DNS configuration;
- Create a VNet peering between the VNet Domain Services was enabled and the Couchbase VNet;
- Manually deploy Couchbase Enterprise on VMs;

### Couchbase Configuration
The steps below must be executed for each Couchbase VM in the cluster.
The first step is to install SASL on the VMs
```
sudo apt-get install -y sasl2-bin
```

Then, update the following entries from the saslauthd file, located on /etc/default/saslauthd.
```
START=yes
MECHANISMS=”ldap”
```

Next, update or create the file /etc/saslauthd.conf, you may need to sudo, using the following template:
```
ldap_servers: ldap://<LDAP server>
ldap_search_base: <Search base distinguished name>
ldap_filter: (sAMAccountName=%u)
ldap_bind_dn: <User bind distinguished name>
ldap_password: <Password>
ldap_auth_method: bind
ldap_version: 3
ldap_use_sasl: no
ldap_restart: yes
ldap_deref: no
ldap_start_tls: no
```

Example:
```
ldap_servers: ldap://rgodinhoad.onmicrosoft.com:389
ldap_search_base: ou=AADDC Users,DC=rgodinhoad,DC=onmicrosoft,DC=com
ldap_filter: (sAMAccountName=%u)
ldap_bind_dn: cn=cbuser,ou=AADDC Users,DC=rgodinhoad,DC=onmicrosoft,DC=com
ldap_password: <Hidden password>
ldap_auth_method: bind
ldap_version: 3
ldap_use_sasl: no
ldap_restart: yes
ldap_deref: no
ldap_start_tls: no
```

Add Couchbase user to the sasl group
```
sudo adduser couchbase sasl
```

Restart saslauthd service
```
sudo service saslauthd restart
```

Change some saslauthd files so Couchbase user can access them
```
sudo chmod 755 /var/run/saslauthd
sudo chmod 755 /var/run/saslauthd/mux
```

Restart saslauthd service a second time
```
sudo service saslauthd restart
```

Change Couchbase configuration to support LDAP
```
/opt/couchbase/bin/couchbase-cli setting-ldap -c localhost:8091 -u <Username> -p <Password> --ldap-enabled 1
```

Restart Couchbase server service
```
sudo service couchbase-server restart
```

## Testing
To check if the user used by the Couchbase service can use the SASL configuration, run the following command:
```
sudo -u couchbase /usr/sbin/testsaslauthd -u <Domain user> -p <Password> -f /var/run/saslauthd/mux
```
If you get a permission denied error, review SASL configuration above.

To authenticate on Couchbase with your domain user, you need to create an external user.
Connect on Couchbase UI, usually `http://<location>:8091` with your regular Couchbase administrator account. Then, click *Security* and *Add User*.

![Add User](/img/Couchbase1.png "Add User")

Pick the *Authentication Domain* as *External* and add the *domain user* as the *Username*. You may inform a *Full Name* and *Roles* as well, finally click *Save*.

![Add New User](/img/Couchbase2.png "Add New User")

Logoff from the Couchbase UI and login with the domain user.

![Login as Domain User](/img/Couchbase3.png "Login as Domain User")

If everything worked, you should see the domain user name on the top right of the screen.

![Logged as Domain User](/img/Couchbase4.png "Logged as Domain User")
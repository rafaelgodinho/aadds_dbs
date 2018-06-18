# Azure Active Directory With Databases
Azure Active Directory can synchronize its credential database with on premises Windows Server Active Directory, allowing the integration of on premises credentials and cloud applications.

Azure Active Directory Domain Services is a Managed Active Directory on Azure, exposing traditional LDAP and Kerberos endpoints applications can use. This configuration easily enables traditional applications, like databases, to authenticate and authorize users against Azure Active Directory, allowing companies to manage their user credentials in a single location.

![Architecture](/img/arch.png "Architecture")

## Requirements
First, you need to enable Azure Active Directory Domain Services on your subscription. You can follow the [step-by-step](https://docs.microsoft.com/en-us/azure/active-directory-domain-services/active-directory-ds-getting-started) from Azure documentation.

## Databases
Then, you need to configure the database to authenticate and/or authorize users from Azure Active Directory Domain Services. Below a list with the configuration for some well known databases:

- [Couchbase Enterprise](couchbase.md)
- [Microsoft SQL Server](sqlserver.md)
- [MongoDB Enterprise](mongodb.md)
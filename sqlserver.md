# Microsoft SQL Server
## Configuration
Microsoft SQL Server supports authentication and authorization of users against Active Directory. You will need to configure the Virtual Machine and the SQL Server logins to allow/deny access to the databases.

### Virtual Machine
The easiest way to use Microsoft SQL Server in Azure is to create a VM using the image available in the [Azure Gallery](https://portal.azure.com/?feature.customportal=false#create/Microsoft.SQLServer2016SP1StandardWindowsServer2016-ARM). The VM must be created on the same VNet Azure Active Directory Domain Services was previously enabled.

Once the VM is running, you can add it to the domain. Just start following [Step 2: Connect to the Windows Server virtual machine by using the local administrator account](https://docs.microsoft.com/en-us/azure/active-directory-domain-services/active-directory-ds-admin-guide-join-windows-vm-portal#step-2-connect-to-the-windows-server-virtual-machine-by-using-the-local-administrator-account). Do not follow Step 1, since the VM was already created using the SQL Server image from Azure Gallery.

### SQL Server Configuration

Now you need to configure the SQL Login for the domain user.

Connect to SQL Server, either using the SQL Server authentication or Windows Authentication with a local user account, and execute the script below. Just remember to replace the placeholders between <> with the right information for your system.

```
USE [master]
GO

CREATE LOGIN [<domain name>\<user name>] FROM WINDOWS WITH DEFAULT_DATABASE=[<default dabase>], DEFAULT_LANGUAGE=[us_english]
GO

ALTER SERVER ROLE [<role name>] ADD MEMBER [<domain name>\<user name>]
GO
```

For example:
```
USE [master]
GO

CREATE LOGIN [rgodinhoad\domainadmin] FROM WINDOWS WITH DEFAULT_DATABASE=[master], DEFAULT_LANGUAGE=[us_english]
GO

ALTER SERVER ROLE [sysadmin] ADD MEMBER [rgodinhoad\domainadmin]
GO
```

## Testing
At this point, SQL Server should already have been configured to support domain authentication. To test the configuration, you first need to connect to the Windows VM using the domain user that was given permission to access the SQL Server on the previous step.

Open Microsoft SQL Server Management Studio and select Windows Authentication on the *Connect to Server* window, then click the *Connect* button.

![Connect To Server](/img/SQLServer_ConnectToServer.png "Connect To Server")

It is possible to check the user connected to SQL Server running the following command:

```
SELECT ORIGINAL_LOGIN()
```
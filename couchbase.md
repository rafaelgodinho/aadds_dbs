# Couchbase Enterprise
## Configuration
Couchbase Enterprise supports authentication and authorization of users against LDAP, including support to Active Directory.

### Virtual Machine
The easiest way to use Couchbase Enterprise in Azure is to deploy an enviroment using one of their offers on Azure Marketplace:

- [BYOL](https://portal.azure.com/?feature.customportal=false#create/couchbase.couchbase-enterprisecouchbase-enterprise-byol)
- [Hourly Pricing](https://portal.azure.com/?feature.customportal=false#create/couchbase.couchbase-enterprisecouchbase-enterprise-hourly-pricing).

It's a requirement the VMs must communicate with the LDAP endpoints from Active Directory Domain Services, but since Couchbase offer on Azure Marketplace does not support deploying the cluster to an existing VNet, you have some options regarding this configuration:

- First deploy the Couchbase Cluster using an Azure Marketplace offer, and then configure Domain Services on the VNet that was created by the Couchbase deployment;
- Create a VNet peering between the VNet Domain Services was enabled and the Couchbase VNet;
- Not use the Azure Marketplace offer and manually deploy Couchbase Enterprise on VMs; 

### Couchbase Configuration

## Testing
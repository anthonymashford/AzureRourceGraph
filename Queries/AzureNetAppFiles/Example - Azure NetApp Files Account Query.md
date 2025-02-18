# Azure Resource Graph Query

## Example - Azure NetApp Files Account

### List all Azure NetApp Files Accounts

This code is written in Kusto Query Language (KQL) and is used to query Azure resources. Here's a brief explanation of what it does:

**Filter Resources:** It filters resources of type microsoft.NetApp/netAppAccounts.

**Select Fields:** It selects specific fields: name, type (renamed to "NetApp account"), location, resourceGroup, and subscriptionId.

**Join with Subscriptions:** It performs an inner join with resourcecontainers to match subscriptions. It selects subscriptionId and renames name to subscriptionName in the joined table.

**Project Final Fields:** Finally, it projects the following fields in the output: Name, Type, Location, Resource Group, and Subscription Name.

So, this query essentially retrieves a list of NetApp accounts along with their relevant details and the corresponding subscription names.

```OQL
// Example - ANF Account Query
resources
| where type =~ 'microsoft.NetApp/netAppAccounts'
| project 
    name, 
    type = "NetApp account", 
    location, 
    resourceGroup, 
    subscriptionId
| join kind=inner (
    resourcecontainers
    | where type == 'microsoft.resources/subscriptions'
    | project subscriptionId, subscriptionName = name
) on subscriptionId
| project 
    ['Name'] = name, 
    ['Type'] = type, 
    ['Location'] = location, 
    ['Resource Group'] = resourceGroup, 
    ['Subscription Name'] = subscriptionName

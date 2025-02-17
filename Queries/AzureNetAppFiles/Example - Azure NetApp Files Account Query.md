# Azure Resource Graph Query

## Example - Azure NetApp Files Account

### List all Azure NetApp Files Accounts

```OQL
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

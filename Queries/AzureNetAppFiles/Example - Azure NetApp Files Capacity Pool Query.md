# Azure Resource Graph Query

## Example - Azure NetApp Files Capacity Pool

### List all Azure NetApp Files Capacity Pools

```OQL
// ANF Capacity Pool Example Query
// Click the "Run query" command above to execute the query and see results.
resources
| where type =~ 'microsoft.NetApp/netAppAccounts/capacityPools'
| extend PoolName = extract(@"capacityPools/([^/]+)", 1, id)
| extend PoolSize = properties.size
| extend CoolAccessEnabled = iif(tobool(properties.coolAccess), "Enabled", "Disabled")
| project
    Name = PoolName,
    Location = location, 
    Size = case(
        PoolSize <= 1023 * 1024 * 1024 * 1024, strcat((PoolSize / (1024 * 1024 * 1024)), " GiB"),
        PoolSize > 1023 * 1024 * 1024 * 1024, strcat((PoolSize / (1024 * 1024 * 1024 * 1024)), " TiB"),
        "Unknown"
    ),
    ['Total Throughput'] = strcat(properties.totalThroughputMibps, " MiB/s"),
    ['Utilised Throughput'] = strcat(properties.utilizedThroughputMibps, " MiB/s"),
    ['Service Tier'] = properties.serviceLevel,
    ['QoS'] = properties.qosType,    
    ['Encryption Type'] = properties.encryptionType,
    ['Cool Access'] = CoolAccessEnabled

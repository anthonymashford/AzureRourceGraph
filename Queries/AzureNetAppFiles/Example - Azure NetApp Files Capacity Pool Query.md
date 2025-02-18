# Azure Resource Graph Query

## Example - Azure NetApp Files Capacity Pool

### List all Azure NetApp Files Capacity Pools

This code is a Kusto query that retrieves information about NetApp capacity pools in your Azure environment. Here's a brief description of what it does:

**Filters** resources to include only those of type microsoft.NetApp/netAppAccounts/capacityPools.

**Extracts** the capacity pool name from the resource ID and assigns it to PoolName.

**Extracts** the size of the pool from the properties and assigns it to PoolSize.

**Determines** if the cool access feature is enabled and assigns the result to CoolAccessEnabled.

**Projects** the results into a new table with the following columns:

- **Name**: Capacity pool name (PoolName)

- **Location**: Location of the capacity pool

- **Size**: Size of the capacity pool in GiB or TiB based on the value of PoolSize

- **Total Throughput**: Total throughput in MiB/s

- **Utilised Throughput**: Utilized throughput in MiB/s

- **Service Tier**: Service level of the capacity pool

- **QoS**: Quality of service type

- **Encryption Type**: Type of encryption used

- **Cool Access**: Indicates if cool access is enabled or disabled

This query provides a detailed overview of the capacity pools, including their sizes, throughput metrics, and other important properties.

```OQL
// Example - ANF Capacity Pool Query
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

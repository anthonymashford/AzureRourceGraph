# Azure Resource Graph Query

## Example - Azure NetApp Files Volume

### List all Azure NetApp Files Volumes

This code is a Kusto query that retrieves detailed information about NetApp volumes in your Azure environment. Here's a brief description of what it does:

**Filters** resources to include only those of type microsoft.NetApp/netAppAccounts/CapacityPools/volumes.

**Extracts** the capacity pool name from the resource ID and assigns it to PoolName.

**Extracts** the quota limit from the properties and assigns it to Quota.

**Determines** if the backup feature is enabled and assigns the result to BackupEnabled.

**Determines** if cool access is enabled and assigns the result to CoolAccessEnabled.

**Extracts** the coolness period from the properties, defaulting to "None" if not available.

**Determines** the encryption type and assigns the result to EncryptionType.

**Expands** the mount targets for the volume and extracts the IP address and FQDN, assigning them to ip and fqdn.

**Splits** the volume name and assigns the relevant part to volNameSplit.

**Cleans** the protocol types and zones into a comma-separated string.

**Expands** the data protection properties and determines if data replication is enabled, cleaning the values to show "Source" or "Destination" as appropriate.

**Projects** the results into a new table with the following columns:

- **Name**: Volume name (volNameSplit)

- **Location**: Location of the volume

- **Quota**: Quota of the volume in GiB or TiB based on the value of Quota

- **Throughput**: Throughput in MiB/s

- **Service Level**: Service level of the volume

- **Capacity Pool**: Name of the capacity pool

- **Zone**: Cleaned zone information

- **Protocol Type**: Cleaned protocol type information

- **FQDN**: Fully Qualified Domain Name

- **IP Address**: IP address of the mount target

- **Network Features**: Network features of the volume

- **Security Style**: Security style of the volume

- **Encryption Type**: Encryption type used

- **Backup Enabled**: Indicates if backup is enabled

- **Replication Enabled**: Indicates if replication is enabled

- **Cool Access Enabled**: Indicates if cool access is enabled

- **Coolness Period**: Coolness period of the volume

- **AVS Datastore**: AVS datastore property

This query provides a comprehensive overview of the volumes, including their quotas, throughput metrics, network and security features, and data protection settings.

```OQL
// Example - ANF Volume Query
resources
| where type =~ 'microsoft.NetApp/netAppAccounts/CapacityPools/volumes'
| extend PoolName = extract(@"capacityPools/([^/]+)", 1, id)
| extend Quota = properties.usageThreshold
| extend BackupEnabled = iif ((properties.dataProtection.backup.backupEnabled) == "true","Yes","No")
| extend CoolAccessEnabled = iif ((properties.coolAccess) == "true","Yes","No")
| extend CoolnessPeriod = iif (isnull (properties.coolnessPeriod), "None", tostring(properties.coolnessPeriod))
| extend EncryptionType = iif ((properties.encryptionKeySource) == "Microsoft.NetApp","Microsoft Managed","Customer Managed")
| mv-expand properties.mountTargets
| extend ip = properties_mountTargets.ipAddress
| extend fqdn = iif (isnull (properties_mountTargets.smbServerFqdn), "N/A", tostring(properties_mountTargets.smbServerFqdn))
| extend volNameSplit = split(name,"/").[2]
| extend cleanProtocol = strcat_array(properties.protocolTypes, ", ")
| extend cleanZone = strcat_array(zones, ", ")
| mv-expand properties.dataProtection
| extend datareplication = iif (isnull (properties_dataProtection.replication.endpointType), "No", tostring(properties_dataProtection.replication.endpointType))
| extend cleanReplication = replace("Src", "Source", datareplication)
| extend cleanReplication = replace("Dst", "Destination", cleanReplication)
| project 
    // properties,
    Name = volNameSplit,
    Location = location,
    Quota = case(
        Quota <= 1023 * 1024 * 1024 * 1024, strcat((Quota / (1024 * 1024 * 1024)), " GiB"),
        Quota > 1023 * 1024 * 1024 * 1024, strcat((Quota / (1024 * 1024 * 1024 * 1024)), " TiB"),
        "Unknown"
    ),
    ['Throughput'] = strcat(properties.throughputMibps, " MiB/s"),
    ['Service Level'] = properties.serviceLevel,
    ["Capacity Pool"] = PoolName,
    ['Zone'] = cleanZone,
    ['Protocol Type'] = cleanProtocol,
    ['FQDN'] = fqdn,
    ['IP Address'] = ip,
    ['Network Features'] = properties.networkFeatures,
    ['Security Style'] = properties.securityStyle,
    ['Encryption Type'] = EncryptionType,
    ['Backup Enabled'] = BackupEnabled,
    ['Replication Enabled'] = cleanReplication,
    ['Cool-Acccess Enabled'] = CoolAccessEnabled,
    ['Coolness Period'] = CoolnessPeriod,
    ['AVS Datastore'] = properties.avsDataStore

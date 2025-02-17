# Azure Resource Graph Query

## Example - Azure NetApp Files Volume

### List all Azure NetApp Files Volumes

```OQL
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

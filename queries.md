# KQL Queries

# Table of contents
- [Azure Arc enabled resource count](#azure-arc-enabled-resource-count)
- [Azure Arc enabled resource count map](#azure-arc-enabled-resource-count-map)
- [Azure Arc enabled server and SQL information](#azure-arc-enabled-server-and-sql-information)

### Azure Arc enabled resource count

This query looks at all the Azure Arc type of resources in your environment and counts them.

```bash
resources
| where type =~ 'microsoft.hybridcompute/machines' or  type=~ 'microsoft.kubernetes/connectedclusters' or  type=~ 'microsoft.azurearcdata/postgresinstances' or  type=~ 'microsoft.azurearcdata/sqlmanagedinstances' or  type=~ 'microsoft.azurearcdata/datacontrollers' or type=~ 'microsoft.azurearcdata/sqlserverinstances' and kind !contains "Azure Arc-enabled"
| summarize count() by type
```

### Azure Arc enabled resource count map

This query looks at all the Azure Arc type of resources in your environment and counts them.  It then displays the results based on location.  You can visualise this using the Map chart. 

```bash
resources
| where type =~ 'microsoft.hybridcompute/machines' or  type=~ 'microsoft.kubernetes/connectedclusters' or  type=~ 'microsoft.azurearcdata/postgresinstances' or  type=~ 'microsoft.azurearcdata/sqlmanagedinstances' or  type=~ 'microsoft.azurearcdata/datacontrollers' or type=~ 'microsoft.azurearcdata/sqlserverinstances' and kind !contains "Azure Arc-enabled"
| summarize count() by location
```

### Azure Arc enabled server and SQL information

This query gives an overview of the servers that have an Arc agent installed. It also looks to see if there is a SQL instance installed on the server and lists out the version if there is.  It displays the name of the server, Arc agent version installed, current status of the agent, the SQL version identified, SQL edition the Azure location, Azure Resource Group name and Azure subscription ID.

```bash
resources
| where type =~ "microsoft.hybridcompute/machines" and kind !contains "Azure Arc-enabled"
| extend agentversion = properties.agentVersion
| extend state = properties.status
| extend status = case(
    state =~ 'Connected', 'Connected',
    state =~ 'Disconnected', 'Offline',
    state =~ 'Error', 'Error',
    state =~ 'Expired', 'Expired',
    '')
| project name, agentversion, status, location, resourceGroup, subscriptionId
| join kind=leftouter ( 
resources
| where type == 'microsoft.azurearcdata/sqlserverinstances'
| extend status = properties.status
| extend sqlversion = properties.version
| extend edition = properties.edition
| project name, sqlversion, edition
) on $left.name == $right.name
| project name, agentversion, status, resourceGroup, location, sqlversion, edition, subscriptionId
```
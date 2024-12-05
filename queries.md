# KQL Queries

# Table of contents
- [Azure Arc enabled resource count](#azure-arc-enabled-resource-count)
- [Azure Arc enabled resource count map](#azure-arc-enabled-resource-count-map)
- [Azure Arc enabled server and SQL information](#azure-arc-enabled-server-and-sql-information)
- [Azure Arc-enabled agent status count](#azure-arc-enabled-server-agent-status-count)
- [Azure Arc-enabled server agent status](#azure-arc-enabled-server-agent-status)

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

### Azure Arc-enabled server agent status count

There are five different status that an Azure Arc agent can have.  These each have different meanings and troubleshooting steps to be followed. 

This query looks at all the Arc-enabled servers in the environment and counts how many are sitting in each status.  The 5 statuses are:

1. **Connected**: The agent communicates successfully with the Azure Arc service, actively sending data and receiving instructions. This is the desired state for a healthy agent.

2. **Disconnected**: Indicates loss of communication with the Azure Arc service. Investigate network issues, misconfiguration, or interruptions to restore connectivity.

3. **Offline**: The agent is not currently running or unable to communicate with Azure Arc. Unlike "Disconnected," this suggests a more prolonged interruption. Troubleshoot, check health, and ensure the agent is running.

4. **Error**: Something has gone wrong with the agent (e.g., misconfiguration, incompatible environment). Investigate error messages or logs for specifics and take corrective actions.

5. **Expired**: Applies to certificates or tokens used for authentication. Renew or update credentials promptly to resolve this status.

```bash
resources 
| where type =~ 'microsoft.hybridcompute/machines' 
| summarize count() by tostring(properties.status)
```

### Azure Arc-enabled server agent status

There are five different status that an Azure Arc agent can have.  These each have different meanings and troubleshooting steps to be followed.

This query specifically looks for the Connected agents, however if you change the properties.status field to one of the other statuses it will find those for you.

The 5 statuses are:

1. **Connected**: The agent communicates successfully with the Azure Arc service, actively sending data and receiving instructions. This is the desired state for a healthy agent.

2. **Disconnected**: Indicates loss of communication with the Azure Arc service. Investigate network issues, misconfiguration, or interruptions to restore connectivity.

3. **Offline**: The agent is not currently running or unable to communicate with Azure Arc. Unlike "Disconnected," this suggests a more prolonged interruption. Troubleshoot, check health, and ensure the agent is running.

4. **Error**: Something has gone wrong with the agent (e.g., misconfiguration, incompatible environment). Investigate error messages or logs for specifics and take corrective actions.

5. **Expired**: Applies to certificates or tokens used for authentication. Renew or update credentials promptly to resolve this status.

```bash
resources
| where type == 'microsoft.hybridcompute/machines' and properties.status=='Connected'
| extend agentversion = properties.agentVersion
| extend state = properties.status
| project name, agentversion, state, location, resourceGroup, subscriptionId
| order by name
```

### Azure Arc-enabled SQL server compatibility level

Show the level of each SQL database from all the Azure Arc-enabled SQL servers. 

```bash
resources
resources
| where type == "microsoft.azurearcdata/sqlserverinstances/databases"
| extend state = properties.state, compat = properties.compatibilityLevel, recoveryMode = properties.recoveryMode
| project name, state, compat, recoveryMode
| summarize  count() by tostring(compat)
```

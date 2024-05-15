# KQL Queries

# Table of contents
- [Azure Arc enabled resource count](#azure-arc-enabled-resource-count)

### Azure Arc enabled resource count

This query looks at all the Azure Arc type of resources in your environment and counts them. 

```bash
resources
| where type =~ 'microsoft.hybridcompute/machines' or  type=~ 'microsoft.kubernetes/connectedclusters' or  type=~ 'microsoft.azurearcdata/postgresinstances' or  type=~ 'microsoft.azurearcdata/sqlmanagedinstances' or  type=~ 'microsoft.azurearcdata/datacontrollers' or type=~ 'microsoft.azurearcdata/sqlserverinstances' and kind !contains "Azure Arc-enabled"
| summarize count() by type
```
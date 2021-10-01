
# Dynamics Data Toolkit
## Overview
This repository contains Base project `DynamicsDataToolkit` to Export, Import, Remove and Confirm data between Dynamics 365 instances. `DynamicsDataToolkitPowerShell` uses `DynamicsDataToolkit` base project for the data operations and provides ***Cmdlets*** you can use to simplify your build & release procedures to easily migrate reference(*configuration*) data with same unique identifier between dynamics instances.


## Goals
> - Easy to use and configure.
> - Automate data export & import between Dynamics 365 instances.
> - Manage configuration data to get consistence record ID(s) between Dynamics 365 instances.
> - Bulk Data delete in Dynamics 365 instance.

## Main Concepts

### Cmdlets
List of Cmdlets `DynamicsDataToolkitPowerShell.dll` provides

- Export-DynamicsData
- Import-DynamicsData
- Remove-DynamicsData
- Confirm-DynamicsData

### Job

The tool takes a **Job** `.xml` file as an input, Job element has a required ***name*** attribute to specify a name for the Job. A job is a list of **Steps** that dynamics data toolkit is going to execute. Each **Step** contains fetch xml query for execution the output result of the query determines the data to be Exported, Imported, Deleted and Verified between Dynamics 365 instances. Every **Step** has two required attributes ***name*** to specify step name and the ***order*** in which step must execute.

Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Job name="Export Job">
  <Steps>
    <Step name="step 1" order="0001">
      <FetchXml>
        <fetch>
          <entity name="test">
            <attribute name="testid" />
            <attribute name="name" />
          </entity>
        </fetch>
      </FetchXml>
    </Step>
  </Steps>
</Job>
```

### Transformations

The Transformation `.json` file is an **array** of **object(s)** that allows to modify target attribute value with the new replacement value during the import process between source to target environment. There are four key attributes to the **object**. Dynamics data import must have transformations for Root **businessunit** and **transactioncurrency** if these transformations are not provided toolkit will try to resolve them by default.

**Minimum recommended transformations**

Example:

```yaml
[
  {
    "TargetEntity": "businessunit",
    "TargetAttribute": "parentbusinessunitid",
    "TargetValue": "380DB065-F6C3-EA11-997E-00155D011205",
    "ReplacementValue": "A15005E9-A0C0-E911-995C-00155D52BA06"
  },
  {
    "TargetEntity": "businessunit",
    "TargetAttribute": "businessunitid",
    "TargetValue": "380DB065-F6C3-EA11-997E-00155D011205",
    "ReplacementValue": "A15005E9-A0C0-E911-995C-00155D52BA06"
  },
  {
    "TargetEntity": "transactioncurrency",
    "TargetAttribute": "transactioncurrencyid",
    "TargetValue": "E5801196-F6C3-EA11-997E-00155D011205",
    "ReplacementValue": "5563F35E-A1C0-E911-995C-00155D52BA06"
  }
]

```

- TargetEntity - Entity logical name where this transformation should apply during import.
- TargetAttribute - Attribute logical name to which attribute to target during import for the `TargetEntity`.
- TargetRecord - `Guid` of the source record to target the transformation for specific record only.
- TargetValue - Current value of the `TargetAttribute` that need to be replaced during import.
- ReplacementValue - Replacement value for the `TargetValue`.  For EntityReference `ReplacementValue` [Guid + pipe`(|)` + EntityLogicalName] can be passed to change the type of EntityReference. For example if source record owner is SystemUser and during transformation we want to change it to team the replacement value will look like **teamid|team** `"ReplacementValue": "5563F35E-A1C0-E911-995C-00155D52BA06|team"`

Example:

```yaml
[
  {
    "TargetEntity": "new_configurationsettings",
    "TargetAttribute": "new_value",
    "TargetValue": "http:/dev/urlForDevInstance",
    "ReplacementValue": "http:/test/urlForTestInstance"
  }
]
```

Transformation file also supports **asterisk** `(*)` Wildcard for `TargetEntity` and `TargetValue` attributes to target any entity or value. 

Example 1: Below example of transformation will be applied on all the entities and will replace the ownerid with the replacement value ownerid record `guid` in the target instance.

```yaml
[
  {
    "TargetEntity": "*",
    "TargetAttribute": "ownerid",
    "TargetValue": "*",
    "ReplacementValue": "7E29126B-5E13-426B-B158-ABE028F74512"
  }
]
```


Example 2: Below example of transformation will be applied on all the entities and will only replace the ownerid if the current ownerid guid is `9B29126B-5E13-426B-B158-ABE028F74512` with the replacement value in the target instance ownerid guid `7E29126B-5E13-426B-B158-ABE028F74512`.

```yaml
[
  {
    "TargetEntity": "*",
    "TargetAttribute": "ownerid",
    "TargetValue": "9B29126B-5E13-426B-B158-ABE028F74512",
    "ReplacementValue": "7E29126B-5E13-426B-B158-ABE028F74512"
  }
]
```


Example 3: Below example of transformation will be applied only on `new_message` entity and will replace all the ownerid(s) to new ownerid guid `7E29126B-5E13-426B-B158-ABE028F74512`.

```yaml
[
  {
    "TargetEntity": "new_message",
    "TargetAttribute": "ownerid",
    "TargetValue": "*",
    "ReplacementValue": "7E29126B-5E13-426B-B158-ABE028F74512"
  }
]
```

Example 4: Below example of transformation will be applied only on `new_message` entity and will replace the ownerid to new ownerid guid `7E29126B-5E13-426B-B158-ABE028F74512` `team` where target record id is `5563F35E-A1C0-E911-995C-00155D52BA06`.

```yaml
[
  {
    "TargetEntity": "new_message",
    "TargetAttribute": "ownerid",
	"TargetRecord" : "5563F35E-A1C0-E911-995C-00155D52BA06",
    "TargetValue": "*",
    "ReplacementValue": "7E29126B-5E13-426B-B158-ABE028F74512|team"
  }
]
```
### Connection string
DynamicsDataToolkit support following [connection strings](https://docs.microsoft.com/en-us/powerapps/developer/data-platform/xrm-tooling/use-connection-strings-xrm-tooling-connect) to connect to dynamics instances.

## Getting started

### Installation
Download and install package [https://www.nuget.org/packages/DynamicsDataToolkitPowerShell/](https://www.nuget.org/packages/DynamicsDataToolkitPowerShell/).

``` 
Install-Package DynamicsDataToolkitPowerShell
```

### PowerShell Cmdlets

Note: All the cmdlets supports `-TargetTimeout` or `-SourceTimeout` parameters to change the organisation service timeouts.

**Cmdlet** `Export-DynamicsData` exports data from Dynamics 365 instance based on the fetch xml steps provided in the job xml file. EntityDataFilesPath is the path to directory where `.xml` data files will be exported.

Example:

```powershell
Import-Module "C:\DynamicsDataToolkit\DynamicsDataToolkitPowerShell.dll"

$connectionString = "AuthType=ClientSecret; url=https://contosotest.crm.dynamics.com;  ClientId={AppId};  ClientSecret={ClientSecret}"
$jobPath = "C:\Jobs\ExportTeamsJob.xml"
$edPath = "C:\JobsOutput\EntityData\"
 
Export-DynamicsData `
	-SourceConnectionString $connectionString `
	-JobFilePath $jobPath `
	-EntityDataFilesPath $edPath
```

**Cmdlet** `Import-DynamicsData` import data from Dynamics 365 instance based on the fetch xml steps provided in the job xml file or it can import data from the entity data files which is the output of the `Export-DynamicsData`. EntityDataFilesPath is the path to directory where `.xml` data files are exported. Optional Switch Parameter DrawTree `-DrawTree` will draw batch in tree structure.  

Example:

```powershell
Import-Module "C:\DynamicsDataToolkit\DynamicsDataToolkitPowerShell.dll"

$sourceConn = "AuthType=ClientSecret; url=https://contosodev.crm.dynamics.com;  ClientId={AppId};  ClientSecret={ClientSecret}"
$targetConn = "AuthType=ClientSecret; url=https://contosotest.crm.dynamics.com;  ClientId={AppId};  ClientSecret={ClientSecret}"

$jobTrans = "C:\Jobs\JobTransformations.json"
$jobPath = "C:\Jobs\ImportTeamsJob.xml"
$edPath = "C:\JobsOutput\EntityData\"

# Ex. Set 1 - Import Data directly from the Job.xml
Import-DynamicsData `
	-SourceConnectionString $sourceConn `
	-TargetConnectionString $targetConn `
	-TransformFilePath $jobTrans `
	-JobFilePath $jobPath `
	-DrawTree

# Ex. Set 2 - Import Data from the EntityDataFilePath
Import-DynamicsData `
	-SourceConnectionString $sourceConn `
	-TargetConnectionString $targetConn `
	-TransformFilePath $jobTrans `
	-EntityDataFilesPath $edPath
```

**Cmdlet** `Remove-DynamicsData` deletes data from Dynamics 365 instance based on the fetch xml steps provided in the job xml file.

Example:

```powershell
Import-Module "C:\DynamicsDataToolkit\DynamicsDataToolkitPowerShell.dll"

$connectionString = "AuthType=ClientSecret; url=https://contosotest.crm.dynamics.com;  ClientId={AppId};  ClientSecret={ClientSecret}"
$jobPath = "C:\Jobs\DeleteTeamsJob.xml"
 
Remove-DynamicsData `
	-TargetConnectionString $connectionString `
	-JobFilePath $jobPath `
	-BatchSize 1000
```

**Cmdlet** `Confirm-DynamicsData` verifies job data exported from source to target Dynamics 365 instance based on the fetch xml steps provided in the job xml file. It verifies if counts of the records matches in source and target based on the id(s) that has been imported.

Example:

```powershell
Import-Module "C:\DynamicsDataToolkit\DynamicsDataToolkitPowerShell.dll"

$sourceConn = "AuthType=ClientSecret; url=https://contosodev.crm.dynamics.com;  ClientId={AppId};  ClientSecret={ClientSecret}"
$targetConn = "AuthType=ClientSecret; url=https://contosotest.crm.dynamics.com;  ClientId={AppId};  ClientSecret={ClientSecret}"

$jobPath = "C:\Jobs\ImportTeamsJob.xml"
 
Confirm-DynamicsData `
	-SourceConnectionString $sourceConn `
	-TargetConnectionString $targetConn `
	-JobFilePath $jobPath 
```
---
title: Create Datasets in Azure Data Factory | Microsoft Docs
description: Learn how to create datasets in Azure Data Factory with examples that use properties such as offset and anchorDateTime.
keywords: create dataset, dataset example, offset example
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: jhubbard
editor: monicar

ms.assetid: 0614cd24-2ff0-49d3-9301-06052fd4f92a
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/05/2016
ms.author: shlo

---
# Datasets in Azure Data Factory
This article describes datasets in Azure Data Factory and includes examples such as offset, anchorDateTime, and offset/style databases.

When you create a dataset, you’re creating a pointer to the data that you want to process. Data is processed (input/output) in an activity and an activity is contained in a pipeline. An input dataset represents the input for an activity in the pipeline and an output dataset represents the output for the activity.

Datasets identify data within different data stores, such as tables, files, folders, and documents. After you create a dataset, you can use it with activities in a pipeline. For example, a dataset can be an input/output dataset of a Copy Activity or an HDInsightHive Activity. The Azure portal gives you a visual layout of all your pipelines and data inputs and outputs. At a glance, you see all the relationships and dependencies of your pipelines across all your sources so you always know where data is coming from and where it is going.

In Azure Data Factory, you can get data from a dataset by using copy activity in a pipeline.

> [!NOTE]
> If you are new to Azure Data Factory, see [Introduction to Azure Data Factory](data-factory-introduction.md) for an overview of Azure Data Factory service. See [Build your first data factory](data-factory-build-your-first-pipeline.md) for a tutorial to create your first data factory. These two articles provide you background information you need to understand this article better.
>
>

## Define datasets
A dataset in Azure Data Factory is defined as follows:

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": <boolean flag to indicate external data. only for input datasets>,
        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        },
        "availability": {
            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
        },
       "policy":
        {      
        }
    }
}
```

The following table describes properties in the above JSON:   

| Property | Description | Required | Default |
| --- | --- | --- | --- |
| name |Name of the dataset. See [Azure Data Factory - Naming rules](data-factory-naming-rules.md) for naming rules. |Yes |NA |
| type |Type of the dataset. Specify one of the types supported by Azure Data Factory (for example: AzureBlob, AzureSqlTable). <br/><br/>See [Dataset Type](#Type) for details. |Yes |NA |
| structure |Schema of the dataset<br/><br/>For details, see [Dataset Structure](#Structure) section. |No. |NA |
| typeProperties |Properties corresponding to the selected type. See [Dataset Type](#Type) section for details on the supported types and their properties. |Yes |NA |
| external |Boolean flag to specify whether a dataset is explicitly produced by a data factory pipeline or not. |No |false |
| availability |Defines the processing window or the slicing model for the dataset production. <br/><br/>For details, see [Dataset Availability](#Availability) section. <br/><br/>For details on the dataset slicing model, see [Scheduling and Execution](data-factory-scheduling-and-execution.md) article. |Yes |NA |
| policy |Defines the criteria or the condition that the dataset slices must fulfill. <br/><br/>For details, see [Dataset Policy](#Policy) section. |No |NA |

## Dataset example
In the following example, the dataset represents a table named **MyTable** in an **Azure SQL database**.

```json
{
    "name": "DatasetSample",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties":
        {
            "tableName": "MyTable"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Note the following points:

* type is set to AzureSqlTable.
* tableName type property (specific to AzureSqlTable type) is set to MyTable.
* linkedServiceName refers to a linked service of type AzureSqlDatabase. See the definition of the following linked service.
* availability frequency is set to Day and interval is set to 1, which means that the slice is produced daily.  

AzureSqlLinkedService is defined as follows:

```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>@<servername>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

In the above JSON:

* type is set to AzureSqlDatabase
* connectionString type property specifies information to connect to an Azure SQL database.  

As you can see, the linked service defines how to connect to an Azure SQL database. The dataset defines what table is used as an input/output for the activity in a pipeline. The activity section in your [pipeline](data-factory-create-pipelines.md) JSON specifies whether the dataset is used as an input or output dataset.

> [!IMPORTANT]
> Unless a dataset is being produced by Azure Data Factory, it should be marked as **external**. This setting generally applies to inputs of first activity in a pipeline.   
>
>

## <a name="Type"></a> Dataset Type
The supported data sources and dataset types are aligned. See topics referenced in the [Data Movement Activities](data-factory-data-movement-activities.md#supported-data-stores-and-formats) article for information on types and configuration of datasets. For example, if you are using data from an Azure SQL database, click Azure SQL Database in the list of supported data stores to see detailed information.  

## <a name="Structure"></a>Dataset Structure
The **structure** section defines the schema of the dataset. It contains a collection of names and data types of columns.  In the following example, the dataset has three columns slicetimestamp, projectname, and pageviews and they are of type: String, String, and Decimal respectively.

```json
structure:  
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

## <a name="Availability"></a> Dataset Availability
The **availability** section in a dataset defines the processing window (hourly, daily, weekly etc.) or the slicing model for the dataset. See [Scheduling and Execution](data-factory-scheduling-and-execution.md) article for more details on the dataset slicing and dependency model.

The following availability section specifies that the output dataset is either produced hourly (or) input dataset is available hourly:

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 1    
}
```

The following table describes properties you can use in the availability section:

| Property | Description | Required | Default |
| --- | --- | --- | --- |
| frequency |Specifies the time unit for dataset slice production.<br/><br/>**Supported frequency**: Minute, Hour, Day, Week, Month |Yes |NA |
| interval |Specifies a multiplier for frequency<br/><br/>”Frequency x interval” determines how often the slice is produced.<br/><br/>If you need the dataset to be sliced on an hourly basis, you set **Frequency** to **Hour**, and **interval** to **1**.<br/><br/>**Note:** If you specify Frequency as Minute, we recommend that you set the interval to no less than 15 |Yes |NA |
| style |Specifies whether the slice should be produced at the start/end of the interval.<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>If Frequency is set to Month and style is set to EndOfInterval, the slice is produced on the last day of month. If the style is set to StartOfInterval, the slice is produced on the first day of month.<br/><br/>If Frequency is set to Day and style is set to EndOfInterval, the slice is produced in the last hour of the day.<br/><br/>If Frequency is set to Hour and style is set to EndOfInterval, the slice is produced at the end of the hour. For example, for a slice for 1 PM – 2 PM period, the slice is produced at 2 PM. |No |EndOfInterval |
| anchorDateTime |Defines the absolute position in time used by scheduler to compute dataset slice boundaries. <br/><br/>**Note:** If the AnchorDateTime has date parts that are more granular than the frequency then the more granular parts are ignored. <br/><br/>For example, if the **interval** is **hourly** (frequency: hour and interval: 1) and the **AnchorDateTime** contains **minutes and seconds**, then the **minutes and seconds** parts of the AnchorDateTime are ignored. |No |01/01/0001 |
| offset |Timespan by which the start and end of all dataset slices are shifted. <br/><br/>**Note:** If both anchorDateTime and offset are specified, the result is the combined shift. |No |NA |

### offset example
Daily slices that start at 6 AM instead of the default midnight.

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```

The **frequency** is set to **Day** and **interval** is set to **1** (once a day):
If you want the slice to be produced at 6 AM instead of at the default time: 12 AM. Remember that this time is an UTC time.

## anchorDateTime example
**Example:** 23 hours dataset slices that start on 2007-04-19T08:00:00

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 23,    
    "anchorDateTime":"2007-04-19T08:00:00"    
}
```

## offset/style Example
If you need dataset on monthly basis on specific date and time (suppose on 3rd of every month at 8:00 AM), use the **offset** tag to set the date and time it should run.

```json
{
  "name": "MyDataset",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "availability": {
      "frequency": "Month",
      "interval": 1,
      "offset": "3.08:00:00",
      "style": "StartOfInterval"
    }
  }
}
```

## <a name="Policy"></a>Dataset Policy
The **policy** section in dataset definition defines the criteria or the condition that the dataset slices must fulfill.

### Validation policies
| Policy Name | Description | Applied To | Required | Default |
| --- | --- | --- | --- | --- |
| minimumSizeMB |Validates that the data in an **Azure blob** meets the minimum size requirements (in megabytes). |Azure Blob |No |NA |
| minimumRows |Validates that the data in an **Azure SQL database** or an **Azure table** contains the minimum number of rows. |<ul><li>Azure SQL Database</li><li>Azure Table</li></ul> |No |NA |

#### Examples
**minimumSizeMB:**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

**minimumRows**

```json
"policy":
{
    "validation":
    {
        "minimumRows": 100
    }
}
```

### External datasets
External datasets are the ones that are not produced by a running pipeline in the data factory. If the dataset is marked as **external**, the **ExternalData** policy may be defined to influence the behavior of the dataset slice availability.

Unless a dataset is being produced by Azure Data Factory, it should be marked as **external**. This setting generally applies to the inputs of first activity in a pipeline unless activity or pipeline chaining is being used.

| Name | Description | Required | Default Value |
| --- | --- | --- | --- |
| dataDelay |Time to delay the check on the availability of the external data for the given slice. For example, if the data is supposed to be available hourly, the check to see the external data is available and the corresponding slice is Ready can be delayed by using dataDelay.<br/><br/>Only applies to the present time.  For example, if it is 1:00 PM right now and this value is 10 minutes, the validation starts at 1:10 PM.<br/><br/>This setting does not affect slices in the past (slices with Slice End Time + dataDelay < Now) are processed without any delay.<br/><br/>Time greater than 23:59 hours need to specified using the day.hours:minutes:seconds format. For example, to specify 24 hours, don't use 24:00:00; instead, use 1.00:00:00. If you use 24:00:00, it is treated as 24 days (24.00:00:00). For 1 day and 4 hours, specify 1:04:00:00. |No |0 |
| retryInterval |The wait time between a failure and the next retry attempt. Applies to present time; if the previous try failed, we wait this long after the last try. <br/><br/>If it is 1:00pm right now, we begin the first try. If the duration to complete the first validation check is 1 minute and the operation failed, the next retry is at 1:00 + 1min (duration) + 1min (retry interval) = 1:02pm. <br/><br/>For slices in the past, there is no delay. The retry happens immediately. |No |00:01:00 (1 minute) |
| retryTimeout |The timeout for each retry attempt.<br/><br/>If this property is set to 10 minutes, the validation needs to be completed within 10 minutes. If it takes longer than 10 minutes to perform the validation, the retry times out.<br/><br/>If all attempts for the validation times out, the slice is marked as TimedOut. |No |00:10:00 (10 minutes) |
| maximumRetry |Number of times to check for the availability of the external data. The allowed maximum value is 10. |No |3 |

## Scoped datasets
You can create datasets that are scoped to a pipeline by using the **datasets** property. These datasets can only be used by activities within this pipeline but not by activities in other pipelines. The following example defines a pipeline with two datasets - InputDataset-rdc and OutputDataset-rdc - to be used within the pipeline:  

> [!IMPORTANT]
> Scoped datasets are supported only with one-time pipelines (**pipelineMode** set to **OneTime**). See [Onetime pipeline](data-factory-scheduling-and-execution.md#onetime-pipeline) for details.
>
>

```json
{
    "name": "CopyPipeline-rdc",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-rdc"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-rdc"
                    }
                ],
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1,
                    "style": "StartOfInterval"
                },
                "name": "CopyActivity-0"
            }
        ],
        "start": "2016-02-28T00:00:00Z",
        "end": "2016-02-28T00:00:00Z",
        "isPaused": false,
        "pipelineMode": "OneTime",
        "expirationTime": "15.00:00:00",
        "datasets": [
            {
                "name": "InputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "InputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/input",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": true,
                    "policy": {}
                }
            },
            {
                "name": "OutputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "OutputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/output",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": false,
                    "policy": {}
                }
            }
        ]
    }
}
```

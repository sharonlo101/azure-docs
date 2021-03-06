﻿---
title: Get started with SQL Database Dynamic Data Masking (Azure Portal)
description: How to get started with SQL Database Dynamic Data Masking in the Azure Portal
services: sql-database
documentationcenter: ''
author: ronitr
manager: jhubbard
editor: v-romcal

ms.assetid: 4b36d78e-7749-4f26-9774-eed1120a9182
ms.service: sql-database
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 07/10/2016
ms.author: ronitr; ronmat; v-romcal; sstein

---
# Get started with SQL Database Dynamic Data Masking (Azure Portal)
> [!div class="op_single_selector"]
> * [Dynamic Data Masking - Azure Classic Portal](sql-database-dynamic-data-masking-get-started-portal.md)
> 
> 

## Overview
SQL Database Dynamic Data Masking limits sensitive data exposure by masking it to non-privileged users. Dynamic data masking is supported for the V12 version of Azure SQL Database.

Dynamic data masking helps prevent unauthorized access to sensitive data by enabling customers to designate how much of the sensitive data to reveal with minimal impact on the application layer. It’s a policy-based security feature that hides the sensitive data in the result set of a query over designated database fields, while the data in the database is not changed.

For example, a service representative at a call center may identify callers by several digits of their social security number or credit card number, but those data items should not be fully exposed to the service representative. A masking rule can be defined that masks all but the last four digits of any social security number or credit card number in the result set of any query. As another example, an appropriate data mask can be defined to protect personally identifiable information (PII) data, so that a developer can query production environments for troubleshooting purposes without violating compliance regulations.

## SQL Database Dynamic Data Masking basics
You set up a dynamic data masking policy in the Azure Portal by selecting the Dynamic Data Masking operation in your SQL Database configuration blade or settings blade.

### Dynamic data masking permissions
Dynamic data masking can be configured by the Azure Database admin, server admin, or security officer roles.

### Dynamic data masking policy
* **SQL users excluded from masking** - A set of SQL users or AAD identities that will get unmasked data in the SQL query results. Note that users with administrator privileges will always be excluded from masking, and will see the original data without any mask.
* **Masking rules** - A set of rules that define the designated fields to be masked and the masking function that will be used. The designated fields can be defined using a database schema name, table name and column name.
* **Masking functions** - A set of methods that control the exposure of data for different scenarios.

| Masking Function | Masking Logic |
| --- | --- |
| **Default** |**Full masking according to the data types of the designated fields**<br/><br/>• Use XXXX or fewer Xs if the size of the field is less than 4 characters for string data types (nchar, ntext, nvarchar).<br/>• Use a zero value for numeric data types (bigint, bit, decimal, int, money, numeric, smallint, smallmoney, tinyint, float, real).<br/>• Use 01-01-1900 for date/time data types (date, datetime2, datetime, datetimeoffset, smalldatetime, time).<br/>• For SQL variant, the default value of the current type is used.<br/>• For XML the document <masked/> is used.<br/>• Use an empty value for special data types (timestamp  table, hierarchyid, GUID, binary, image, varbinary spatial types). |
| **Credit card** |**Masking method which exposes the last four digits of the designated fields** and adds a constant string as a prefix in the form of a credit card.<br/><br/>XXXX-XXXX-XXXX-1234 |
| **Social security number** |**Masking method which exposes the last four digits of the designated fields** and adds a constant string as a prefix in the form of an American social security number.<br/><br/>XXX-XX-1234 |
| **Email** |**Masking method which exposes the first letter and replaces the domain with XXX.com** using a constant string prefix in the form of an email address.<br/><br/>aXX@XXXX.com |
| **Random number** |**Masking method which generates a random number** according to the selected boundaries and actual data types. If the designated boundaries are equal, then the masking function will be a constant number.<br/><br/>![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/1_DDM_Random_number.png) |
| **Custom text** |**Masking method which exposes the first and last characters** and adds a custom padding string in the middle. If the original string is shorter than the exposed prefix and suffix, only the padding string will be used. <br/>prefix[padding]suffix<br/><br/>![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/2_DDM_Custom_text.png) |

<a name="Anchor1"></a>

### Recommended fields to mask
The DDM recommendations engine flags certain fields from your database as potentially sensitive fields, which may be good candidates for masking. In the Dynamic Data Masking blade in the portal, you will see the recommended columns for your database. All you need to do is click **Add Mask** for one or more columns and then **Save** in order to apply a mask for these fields.

## Set up dynamic data masking for your database using the Azure Portal
1. Launch the Azure Portal at [https://portal.azure.com](https://portal.azure.com).
2. Navigate to the settings blade of the database that includes the sensitive data you want to mask.
3. Click the **Dynamic Data Masking** tile which launches the **Dynamic Data Masking** configuration blade.
   
   * Alternatively, you can scroll down to the **Operations** section and click **Dynamic Data Masking**.
     
     ![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/4_ddm_settings_tile.png)<br/><br/>
4. In the **Dynamic Data Masking** configuration blade you may see some database columns that the recommendations engine has flagged for masking. In order to accept the recommendations, just click **Add Mask** for one or more columns and a mask will be created based on the default type for this column. You can change the masking function by clicking on the masking rule and editing the masking field format to a different format of your choice. Be sure to click **Save** to save your settings.
   
    ![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/5_ddm_recommendations.png)<br/><br/>
5. To add a mask for any column in your database, at the top of the **Dynamic Data Masking** configuration blade click **Add Mask** to open the **Add Masking Rule** configuration blade
   
    ![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/6_ddm_add_mask.png)<br/><br/>
6. Select the **Schema**, **Table** and **Column** to define the designated field that will be masked.
7. Choose a **Masking Field Format** from the list of sensitive data masking categories.
   
    ![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/7_ddm_mask_field_format.png)<br/><br/>        
8. Click **Save** in the data masking rule blade to update the set of masking rules in the dynamic data masking policy.
9. Type the SQL users or AAD identities that should be excluded from masking, and have access to the unmasked sensitive data. This should be a semicolon-separated list of users. Note that users with administrator privileges always have access to the original unmasked data.
   
    ![Navigation pane](./media/sql-database-dynamic-data-masking-get-started/8_ddm_excluded_users.png)
   
   > [!TIP]
   > To make it so the application layer can display sensitive data for application privileged users, add the SQL user or AAD identity the application uses to query the database. It is highly recommended that this list contain a minimal number of privileged users to minimize exposure of the sensitive data.
   > 
   > 
10. Click **Save** in the data masking configuration blade to save the new or updated masking policy.

## Set up dynamic data masking for your database using Powershell cmdlets
See [Azure SQL Database Cmdlets](https://msdn.microsoft.com/library/azure/mt574084.aspx).

## Set up dynamic data masking for your database using REST API
See [Operations for Azure SQL Databases](https://msdn.microsoft.com/library/dn505719.aspx).


# Viewing instance status and recommendations<a name="accessing-monitoring"></a>

Using the Amazon RDS console, you can quickly access the status of your DB instance and respond to Amazon RDS recommendations\.

**Topics**
+ [Viewing Amazon RDS DB instance status](#Overview.DBInstance.Status)
+ [Viewing Amazon RDS recommendations](#USER_Recommendations)

## Viewing Amazon RDS DB instance status<a name="Overview.DBInstance.Status"></a>

The status of a DB instance indicates the health of the DB instance\. You can use the following procedures to view the DB instance status in the Amazon RDS console, the AWS CLI command, or the API operation\.

**Note**  
Amazon RDS also uses another status called *maintenance status*, which is shown in the **Maintenance** column of the Amazon RDS console\. This value indicates the status of any maintenance patches that need to be applied to a DB instance\. Maintenance status is independent of DB instance status\. For more information about maintenance status, see [Applying updates for a DB instance](USER_UpgradeDBInstance.Maintenance.md#USER_UpgradeDBInstance.OSUpgrades)\. 

Find the possible status values for DB instances in the following table\. This table also shows whether you will be billed for the DB instance and storage, billed only for storage, or not billed\. For all DB instance statuses, you are always billed for backup usage\.


| DB instance status | Billed  | Description | 
| --- | --- | --- | 
|  **Available**  | Billed |  The DB instance is healthy and available\.  | 
|  **Backing\-up**  | Billed |  The DB instance is currently being backed up\.  | 
|  **Configuring\-enhanced\-monitoring**  | Billed |  Enhanced Monitoring is being enabled or disabled for this DB instance\.  | 
|  **Configuring\-iam\-database\-auth**  | Billed |  AWS Identity and Access Management \(IAM\) database authentication is being enabled or disabled for this DB instance\.  | 
|  **Configuring\-log\-exports**  | Billed |  Publishing log files to Amazon CloudWatch Logs is being enabled or disabled for this DB instance\.  | 
|  **Converting\-to\-vpc**  | Billed |  The DB instance is being converted from a DB instance that is not in an Amazon Virtual Private Cloud \(Amazon VPC\) to a DB instance that is in an Amazon VPC\.  | 
|  **Creating**  | Not billed |  The DB instance is being created\. The DB instance is inaccessible while it is being created\.   | 
|  **Delete\-precheck**  | Not billed |  Amazon RDS is validating that read replicas are healthy and are safe to delete\.  | 
|  **Deleting**  | Not billed |  The DB instance is being deleted\.  | 
|  **Failed**  | Not billed |  The DB instance has failed and Amazon RDS can't recover it\. Perform a point\-in\-time restore to the latest restorable time of the DB instance to recover the data\.   | 
|  **Inaccessible\-encryption\-credentials**  | Not billed |  The AWS KMS key used to encrypt or decrypt the DB instance can't be accessed or recovered\.   | 
|  **Inaccessible\-encryption\-credentials\-recoverable**  | Billed for storage |  The KMS key used to encrypt or decrypt the DB instance can't be accessed\. However, if the KMS key is active, restarting the DB instance can recover it\. For more information, see [Encrypting a DB instance](Overview.Encryption.md#Overview.Encryption.Enabling)\.  | 
|  **Incompatible\-network**  | Not billed |  Amazon RDS is attempting to perform a recovery action on a DB instance but can't do so because the VPC is in a state that prevents the action from being completed\. This status can occur if, for example, all available IP addresses in a subnet are in use and Amazon RDS can't get an IP address for the DB instance\.   | 
|  **Incompatible\-option\-group**  | Billed |  Amazon RDS attempted to apply an option group change but can't do so, and Amazon RDS can't roll back to the previous option group state\. For more information, check the **Recent Events** list for the DB instance\. This status can occur if, for example, the option group contains an option such as TDE and the DB instance doesn't contain encrypted information\.   | 
|  **Incompatible\-parameters**  | Billed |  Amazon RDS can't start the DB instance because the parameters specified in the DB instance's DB parameter group aren't compatible with the DB instance\. Revert the parameter changes or make them compatible with the DB instance to regain access to your DB instance\. For more information about the incompatible parameters, check the **Recent Events** list for the DB instance\.   | 
|  **Incompatible\-restore**  | Not billed |  Amazon RDS can't do a point\-in\-time restore\. Common causes for this status include using temp tables, using MyISAM tables with MySQL, or using Aria tables with MariaDB\.  | 
| Insufficient\-capacity | Not billed |  Amazon RDS can’t create your instance because sufficient capacity isn’t currently available\. To create your DB instance in the same AZ with the same instance type, delete your DB instance, wait a few hours, and try to create again\. Alternatively, create a new instance using a different instance class or AZ\.  | 
|  **Maintenance**  | Billed |  Amazon RDS is applying a maintenance update to the DB instance\. This status is used for instance\-level maintenance that RDS schedules well in advance\.   | 
|  **Modifying**  | Billed |  The DB instance is being modified because of a customer request to modify the DB instance\.   | 
|  **Moving\-to\-vpc**  | Billed |  The DB instance is being moved to a new Amazon Virtual Private Cloud \(Amazon VPC\)\.  | 
|  **Rebooting**  | Billed |  The DB instance is being rebooted because of a customer request or an Amazon RDS process that requires the rebooting of the DB instance\.  | 
|  **Resetting\-master\-credentials**  | Billed |  The master credentials for the DB instance are being reset because of a customer request to reset them\.  | 
|  **Renaming**  | Billed |  The DB instance is being renamed because of a customer request to rename it\.   | 
|  **Restore\-error**  | Billed |  The DB instance encountered an error attempting to restore to a point\-in\-time or from a snapshot\.  | 
|  **Starting**  | Billed for storage |  The DB instance is starting\.  | 
|  **Stopped**  | Billed for storage |  The DB instance is stopped\.  | 
|  **Stopping**  | Billed for storage |  The DB instance is being stopped\.  | 
|  **Storage\-full**  | Billed |  The DB instance has reached its storage capacity allocation\. This is a critical status, and we recommend that you fix this issue immediately\. To do so, scale up your storage by modifying the DB instance\. To avoid this situation, set Amazon CloudWatch alarms to warn you when storage space is getting low\.   | 
|  **Storage\-optimization**  | Billed |  Amazon RDS is optimizing the storage of your DB instance\. The DB instance is fully operational\. The storage optimization process is usually short, but can sometimes take up to and even beyond 24 hours\.  | 
|  **Upgrading**  | Billed |  The database engine version is being upgraded\.   | 

### Console<a name="DBinstance.Status.Console"></a>

**To view the status of a DB instance**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**\.

   The **Databases page** appears with the list of DB instances\. For each DB instance , the status value is displayed\.   
![\[View the status of a DB instance\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/RDS_instance_status.png)

### CLI<a name="DBinstance.Status.Cli"></a>

To view DB instance and its status information by using the AWS CLI, use the [describe\-db\-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) command\. For example, the following AWS CLI command lists all the DB instances information \.

```
aws rds describe-db-instances
```

To view a specific DB instance and its status, call the [describe\-db\-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) command with the following option:
+ `DBInstanceIdentifier` – The name of the DB instance\. 

```
aws rds describe-db-instances --db-instance-identifier mydbinstance
```

To view just the status of all the DB instances, use the following query in AWS CLI\.

```
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]' --output table
```

### API<a name="DBinstance.Status.Api"></a>

To view the status of the DB instance using the Amazon RDS API, call the [DescribeDBInstances](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBInstances.html) operation\.

## Viewing Amazon RDS recommendations<a name="USER_Recommendations"></a>

Amazon RDS provides automated recommendations for database resources, such as DB instances, read replicas, and DB parameter groups\. These recommendations provide best practice guidance by analyzing DB instance configuration, usage, and performance data\.

You can find examples of these recommendations in the following table\.


| Type | Description | Recommendation | Additional information | 
| --- | --- | --- | --- | 
|  DB instance isn't a Multi\-AZ DB instance  |  Your DB instance isn't using the Multi\-AZ deployment\.  |  We recommend that you use Multi\-AZ deployment\. The Multi\-AZ deployments enhance the availability and durability of the DB instance\. For information about Amazon RDS Multi\-AZ pricing, see [Pricing](https://aws.amazon.com/rds/features/multi-az/#Pricing)\.  |  [ Amazon RDS Multi\-AZ](https://aws.amazon.com/rds/features/multi-az/)  | 
|   |  Your DB cluster has all the reader instances in the same Availability Zone\.  |  We recommend that you distribute the Reader instances across multiple Availability Zones\. Distribution increases the availability of the database, and improves the response time by reducing network latency between clients and database\.  |   | 
|  Storage autoscaling isn't turned on  |  Your DB instance doesn't have Amazon RDS storage autoscaling turned on\. Storage autoscaling automatically scales the storage capacity when there is an increase in the database size, with zero downtime\.  |  We recommend that you turn on the storage autoscaling with a maximum allocated storage of `{{MaxAllocatedStorage}}` GB for your DB instance `{{DBInstanceIdentifier}}`\.  | [Managing capacity automatically with Amazon RDS storage autoscaling](USER_PIOPS.StorageTypes.md#USER_PIOPS.Autoscaling) | 
|  Engine version outdated  |  Your DB instance is not running the latest minor engine version\.  |  We recommend that you upgrade to the latest version because it contains the latest security fixes and other improvements\.  |  [Upgrading a DB instance engine version](USER_UpgradeDBInstance.Upgrading.md)  | 
|  Pending maintenance available  |  You have pending maintenance available on your DB instance\.  |  We recommend that you perform the pending maintenance available on your DB instance\. Updates to the operating system most often occur for security issues and should be done as soon as possible\.  |  [Maintaining a DB instance](USER_UpgradeDBInstance.Maintenance.md)  | 
|  Automated backups disabled  |  Your DB instance has automated backups disabled\.  |  We recommend that you enable automated backups on your DB instance\. Automated backups enable point\-in\-time recovery of your DB instance\. You receive backup storage up to the storage size of your DB instance at no additional charge\.  |  [Working with backups](USER_WorkingWithAutomatedBackups.md)  | 
|  Magnetic volumes in use  |  Your DB instance is using magnetic storage\.  |  Magnetic storage is not recommended for most DB instances\. We recommend switching to General Purpose \(SSD\) storage or provisioned IOPS storage\.  |  [Amazon RDS DB instance storage](CHAP_Storage.md)  | 
|  Enhanced Monitoring disabled  |  Your DB instance doesn't have Enhanced Monitoring enabled\.  |  We recommend enabling Enhanced Monitoring\. Enhanced Monitoring provides real\-time operating system metrics for monitoring and troubleshooting\.  |  [Monitoring OS metrics with Enhanced Monitoring](USER_Monitoring.OS.md)  | 
|  Performance Insights disabled  |  Your DB instance doesn't have Performance Insights enabled\.  |  We recommend enabling Performance Insights\. Performance Insights monitors your database load for better analysis and troubleshooting\.  |  [Overview of Performance Insights on Amazon RDS](USER_PerfInsights.Overview.md)  | 
|  Encryption disabled  |  Your DB instance doesn't have encryption enabled\.  |  We recommend enabling encryption\. You can encrypt your existing Amazon RDS DB instances by restoring from an encrypted snapshot\.  |  [Encrypting Amazon RDS resources](Overview.Encryption.md)  | 
|  Previous generation DB instance class in use  |  Your DB instance is running on a previous\-generation DB instance class\.  |  Previous\-generation DB instance classes have been replaced by DB instance classes with better price, better performance, or both\. We recommend running your DB instance on a later generation DB instance class\.  |  [DB instance classes](Concepts.DBInstanceClass.md)  | 
|  Huge pages not used for an Oracle DB instance  |  The `use_large_pages` parameter is not set to `ONLY` in the DB parameter group used by your DB instance\.  |  For increased database scalability, we recommend setting `use_large_pages` to `ONLY` in the DB parameter group used by your DB instance\.  |  [Turning on HugePages for an RDS for Oracle instance](Oracle.Concepts.HugePages.md)  | 
|  Nondefault custom memory parameters  |  Your DB parameter group sets memory parameters that diverge too much from the default values\.  |  Settings that diverge too much from the default values can cause poor performance and errors\. We recommend setting custom memory parameters to their default values in the DB parameter group used by the DB instance\.  |  [Working with parameter groups](USER_WorkingWithParamGroups.md)  | 
|  Found an unsafe durability parameter value for a MySQL DB instance  |  Your DB instance has an unsafe value for the `innodb_flush_log_at_trx_commit` parameter\. This parameter controls the persistence of commit operations to disk\.  |  We recommend that you set the value of the `innodb_flush_log_at_trx_commit` parameter to `1`\. The current value might improve performance but transactions can be lost if the database crashes\.  |  [ Best practices for configuring parameters for Amazon RDS for MySQL, part 1: Parameters related to performance](http://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-1-parameters-related-to-performance/) on the AWS Database Blog  | 
|  Optimizer statistics aren't persisted to the disk for a MySQL DB instance  |  Your DB instance isn't configured to persist the InnoDB statistics to the disk\. When it isn't configured, the statistics may recalculate frequently, which leads to variations in query execution plan\. You can modify the value of this global parameter at the table level\.  |  Global statistics persistence is disabled\. We recommend that you set the `innodb_stats_persistent` parameter to `ON`\.  |  [ Best practices for configuring parameters for Amazon RDS for MySQL, part 1: Parameters related to performance](http://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-1-parameters-related-to-performance/) on the AWS Database Blog  | 
|  General logging is enabled for a MySQL DB instance  |  Your DB instance has the general logging turned on\. Turning on general logging increases the amount of I/O operations and allocated storage space,which can lead to contention and performance degradation\.  |  Evaluate your required general logging usage\. General logging can increase the amount of I/O operations and allocated storage space, and lead to contention and performance degradation\.  |  [Managing table\-based MySQL logs](Appendix.MySQL.CommonDBATasks.Logs.md)  | 
|  Maximum InnoDB open files setting is misconfigured for a MySQL DB instance  |  Your DB instance has a low value for the maximum number of files InnoDB can open at one time\.  |  We recommend that you set the `innodb_open_files` parameter to a minimum value of `65`\.  |  [innodb\_open\_files](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_open_files)  | 
|  Number of allowed simultaneous connections for a given database user is misconfigured for a MySQL DB instance  |  Your DB instance has a low value for the maximum number of simultaneous connections for each database account\.  |  We recommend that you increase the setting of the `max_user_connections` parameter to a number greater than 5\. The current `max_user_connections` value is low which impacts the database health checks and regular operations\.  |  [Setting Account Resource Limits](https://dev.mysql.com/doc/refman/8.0/en/user-resources.html)  | 
|  Read replica is open in writable mode for a MySQL DB instance  |  Your DB instance has the Read replica in writable mode, which allows updates from clients\.  |  We recommend that you don't change MySQL read replicas to writable mode for a long duration\. This setting can cause replication errors and data consistency issues\.  |  [ Best practices for configuring parameters for Amazon RDS for MySQL, part 2: Parameters related to replication](http://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-2-parameters-related-to-replication/) on the AWS Database Blog  | 
|  Found an unsafe durability parameter value for a MySQL DB instance  |  The synchronization of the binary log to disk isn't enforced before the acknowledgement of the transactions commit in your DB instance\.  |  We recommend that you set the `sync_binlog` parameter to `1`\. Currently, the synchronization of the binary log to disk isn't enforced before acknowledgement of the transactions commit\. If there is a power failure or the operating system crashes, the committed transactions can be lost\.  |  [ Best practices for configuring parameters for Amazon RDS for MySQL, part 2: Parameters related to replication](http://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-2-parameters-related-to-replication/) on the AWS Database Blog  | 
|  Found an unsafe setting of the `innodb_default_row_format` parameter for a MySQL DB instance  |  Your DB instance has the the following known issue: A table created in a MySQL version lower than 8\.0\.26 with row\_format COMPACT or REDUNDANT will be inaccessible and unrecoverable when the index exceeds 767 bytes\.  |  We recommend that you change the current value of the `innodb_default_row_format` parameter to `DYNAMIC`\.  |  [Changes in MySQL 8\.0\.26 \(2021\-07\-20, General Availability\)](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-26.html)  | 
|  Change buffering enabled for a MySQL DB instance  |  Your DB parameter group has change buffering enabled\.  |  Change buffering allows a MySQL DB instance to defer some writes necessary to maintain secondary indexes\. This configuration can improve performance slightly, but it can create a large delay in crash recovery\. During crash recovery, the secondary index must be brought up to date\. So, the benefits of change buffering are outweighed by the potentially very long crash recovery events\. We recommend disabling change buffering\.  |  [ Best practices for configuring parameters for Amazon RDS for MySQL, part 1: Parameters related to performance](http://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-1-parameters-related-to-performance/) on the AWS Database Blog  | 
|  Query cache enabled for a MySQL DB instance  |  Your DB parameter group has query cache parameter enabled\.  |  The query cache can cause the DB instance to appear to stall when changes require the cache to be purged\. Most workloads don't benefit from a query cache\. The query cache was removed from MySQL version 8\.0\. We recommend that you disable the query cache parameter\.  |  [ Best practices for configuring parameters for Amazon RDS for MySQL, part 1: Parameters related to performance](http://aws.amazon.com/blogs/database/best-practices-for-configuring-parameters-for-amazon-rds-for-mysql-part-1-parameters-related-to-performance/) on the AWS Database Blog  | 
|  Autovacuum is disabled for a PostgreSQL DB instance  |  Your DB instance has autovacuum turned off\. Turning off autovacuum increases table and index bloat and impacts performance\.  |  We recommend that you set the autovacuum parameter to on\.  |  [Understanding autovacuum in Amazon RDS for PostgreSQL environments](https://aws.amazon.com/blogs/database/understanding-autovacuum-in-amazon-rds-for-postgresql-environments/)  | 
|  Synchronous commit is turned off for a PostgreSQL DB instance  |  When the `synchronous_commit` parameter is set to `OFF`, it causes data loss when the database crashes, which can impact the durability of the database\.  |  We recommend that you turn on the `synchronous_commit` parameter\.  |  [Asynchronous Commit](https://www.postgresql.org/docs/current/wal-async-commit.html)  | 
|  `track_counts` parameter is disabled for a PostgreSQL DB instance  |  If the track\_counts parameter is turned off, the database doesn't collect the database activity statistics\. Autovacuum requires these statistics to work correctly\.  |  We recommend that you set `track_counts` parameter to `ON`\.  |  [track\_counts \(boolean\)](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-COUNTS)  | 
|  Index only scan plan type is disabled for a PostgreSQL DB instance  |  The query planner or optimizer can't use the index only scan plan when it is disabled\.   |  We recommend that you set the parameter `enable_indexonlyscan` to `ON`\.  |  [enable\_indexonlyscan \(boolean\)](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-INDEXONLYSCAN)  | 
|  index\-scan plan type is disabled for a PostgreSQL DB instance  |  The query planner or optimizer can't use the index scan plan types when it is disabled\.  |  We recommend that you set the parameter `enable_indexscan` to `ON`\.  |  [enable\_indexscan \(boolean\) ](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-INDEXSCAN)  | 
|  Logging to table  |  Your DB parameter group sets logging output to `TABLE`\.  |  Setting logging output to `TABLE` uses more storage than setting this parameter to `FILE`\. To avoid reaching the storage limit, we recommend setting the logging output parameter to `FILE`\.  |  [MySQL database log files](USER_LogAccess.Concepts.MySQL.md)  | 

Amazon RDS generates recommendations for a resource when the resource is created or modified\. Amazon RDS also periodically scans your resources and generates recommendations\.

### <a name="USER_Recommendations.Responding"></a>

**To view Amazon RDS recommendations**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Recommendations**\.  
![\[Select Recommendations in the console\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/recommendations-select.png)

   The Recommendations page appears\.  
![\[Main Recommendations page in the console\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/recommendations-main.png)

1. On the **Recommendations** page, choose one of the following:
   + **Active** – Shows the current recommendations that you can apply, dismiss, or schedule\.
   + **Dismissed** – Shows the recommendations that have been dismissed\. When you choose **Dismissed**, you can apply these dismissed recommendations\.
   + **Scheduled** – Shows the recommendations that are scheduled but not yet applied\. These recommendations will be applied in the next scheduled maintenance window\.
   + **Applied** – Shows the recommendations that are currently applied\.

   From any list of recommendations, you can open a section to view the recommendations in that section\.  
![\[Take action on recommendations in the console\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/recommendations-active.png)

   To configure preferences for displaying recommendations in each section, choose the **Preferences** icon\.  
![\[Preferences icon for Recommendations in the console\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/recommendations-preferences.png)

   From the **Preferences** window that appears, you can set display options\. These options include the visible columns and the number of recommendations to display on the page\.

1. \(optional\) Respond to your active recommendations as follows:

   1. Choose **Active** and open one or more sections to view the recommendations in them\.

   1. Choose one or more recommendations and choose **Apply now** \(to apply them immediately\), **Schedule** \(to apply them in next maintenance window\), or **Dismiss**\.

      If the **Apply now** button appears for a recommendation but is unavailable \(grayed out\), the DB instance is not available\. You can apply recommendations immediately only if the DB instance status is **available**\. For example, you can't apply recommendations immediately to the DB instance if its status is **modifying**\. In this case, wait for the DB instance to be available and then apply the recommendation\.

      If the **Apply now** button doesn't appear for a recommendation, you can't apply the recommendation using the **Recommendations** page\. You can modify the DB instance to apply the recommendation manually\.

      For more information about modifying a DB instance, see [Modifying an Amazon RDS DB instance](Overview.DBInstance.Modifying.md)\.
**Note**  
When you choose **Apply now**, a brief DB instance outage might result\.
# Improving write performance with Amazon RDS Optimized Writes for MariaDB<a name="rds-optimized-writes-mariadb"></a>

You can improve the performance of write transactions with Amazon RDS Optimized Writes for MariaDB\. When your RDS for MariaDB database uses RDS Optimized Writes, it can achieve up to two times higher write transaction throughput\.

**Topics**
+ [Overview of RDS Optimized Writes](#rds-optimized-writes-overview)
+ [Using RDS Optimized Writes](#rds-optimized-writes-using-mariadb)
+ [Limitations for RDS Optimized Writes](#rds-optimized-writes-limitations-mariadb)

## Overview of RDS Optimized Writes<a name="rds-optimized-writes-overview"></a>

When you turn on Amazon RDS Optimized Writes, your RDS for MariaDB databases write only once when flushing data to durable storage without the need for the doublewrite buffer\. The databases continue to provide ACID property protections for reliable database transactions, along with improved performance\.

Relational databases, like MariaDB, provide the *ACID properties* of atomicity, consistency, isolation, and durability for reliable database transactions\. To help provide these properties, MariaDB uses a data storage area called the *doublewrite buffer* that prevents partial page write errors\. These errors occur when there is a hardware failure while the database is updating a page, such as in the case of a power outage\. A MariaDB database can detect partial page writes and recover with a copy of the page in the doublewrite buffer\. While this technique provides protection, it also results in extra write operations\. For more information about the MariaDB doublewrite buffer, see [InnoDB Doublewrite Buffer](https://mariadb.com/kb/en/innodb-doublewrite-buffer/) in the MariaDB documentation\.

With RDS Optimized Writes turned on, RDS for MariaDB databases write only once when flushing data to durable storage without using the doublewrite buffer\. RDS Optimized Writes is useful if you run write\-heavy workloads on your RDS for MariaDB databases\. Examples of databases with write\-heavy workloads include ones that support digital payments, financial trading, and gaming applications\.

These databases run on DB instance classes that use the AWS Nitro System\. Because of the hardware configuration in these systems, the database can write 16\-KiB pages directly to data files reliably and durably in one step\. The AWS Nitro System makes RDS Optimized Writes possible\.

You can set the new database parameter `rds.optimized_writes` to control the RDS Optimized Writes feature for RDS for MariaDB databases\. Access this parameter in the DB parameter groups of RDS for MariaDB version 10\.6\.10 or higher\. Set the parameter using the following values:
+ `AUTO` – Turn on RDS Optimized Writes if the database supports it\. Turn off RDS Optimized Writes if the database doesn't support it\. This setting is the default\.
+ `OFF` – Turn off RDS Optimized Writes even if the database supports it\.

If you migrate an RDS for MariaDB database that is configured to use RDS Optimized Writes to a DB instance class that doesn't support the feature, RDS automatically turns off RDS Optimized Writes for the database\.

When RDS Optimized Writes is turned off, the database uses the MariaDB doublewrite buffer\.

To determine whether an RDS for MariaDB database is using RDS Optimized Writes, view the current value of the `innodb_doublewrite` parameter for the database\. If the database is using RDS Optimized Writes, this parameter is set to `FALSE` \(`0`\)\.

## Using RDS Optimized Writes<a name="rds-optimized-writes-using-mariadb"></a>

You can turn on RDS Optimized Writes when you create an RDS for MariaDB database with the RDS console, the AWS CLI, or the RDS API\. RDS Optimized Writes is turned on automatically when both of the following conditions apply during database creation:
+ You specify a DB engine version and DB instance class that support RDS Optimized Writes\.
  + RDS Optimized Writes is supported for RDS for MariaDB version 10\.6\.10 and higher\. For information about RDS for MariaDB versions, see [MariaDB on Amazon RDS versions](MariaDB.Concepts.VersionMgmt.md)\.
  + RDS Optimized Writes is supported for RDS for MariaDB databases that use the following DB instance classes: 
    + db\.x2idn
    + db\.x2iedn
    + db\.m7g
    + db\.r7g
    + db\.r6g
    + db\.r6gd
    + db\.r6i
    + db\.r5b

    For information about DB instance classes, see [DB instance classes](Concepts.DBInstanceClass.md)\.

    DB instance class availability differs for AWS Regions\. To determine whether a DB instance class is supported in a specific AWS Region, see [Determining DB instance class support in AWS Regions](Concepts.DBInstanceClass.md#Concepts.DBInstanceClass.RegionSupport)\.
+ In the parameter group associated with the database, the `rds.optimized_writes` parameter is set to `AUTO`\. In default parameter groups, this parameter is always set to `AUTO`\.

If you want to use a DB engine version and DB instance class that support RDS Optimized Writes, but you don't want to use this feature, then specify a custom parameter group when you create the database\. In this parameter group, set the `rds.optimized_writes` parameter to `OFF`\. If you want the database to use RDS Optimized Writes later, you can set the parameter to `AUTO` to turn it on\. For information about creating custom parameter groups and setting parameters, see [Working with parameter groups](USER_WorkingWithParamGroups.md)\.

For information about creating a DB instance, see [Creating an Amazon RDS DB instance](USER_CreateDBInstance.md)\.

### Console<a name="rds-optimized-writes-using-console"></a>

When you use the RDS console to create an RDS for MariaDB database, you can filter for the DB engine versions and DB instance classes that support RDS Optimized Writes\. After you turn on the filters, you can choose from the available DB engine versions and DB instance classes\.

To choose a DB engine version that supports RDS Optimized Writes, filter for the RDS for MariaDB DB engine versions that support it in **Engine version**, and then choose a version\.

![\[DB engine version filter for RDS Optimized Writes.\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/rds-optimized-writes-version-filter-mariadb.png)

In the **Instance configuration** section, filter for the DB instance classes that support RDS Optimized Writes, and then choose a DB instance class\.

![\[DB instance class filter for RDS Optimized Writes.\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/rds-optimized-writes-class-filter.png)

After you make these selections, you can choose other settings that meet your requirements and finish creating the RDS for MariaDB database with the console\.

### AWS CLI<a name="rds-optimized-writes-using-cli"></a>

To create a DB instance by using the AWS CLI, use the [create\-db\-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) command\. Make sure the `--engine-version` and `--db-instance-class` values support RDS Optimized Writes\. In addition, make sure the parameter group associated with the DB instance has the `rds.optimized_writes` parameter set to `AUTO`\. This example associates the default parameter group with the DB instance\.

**Example Creating a DB instance that uses RDS Optimized Writes**  
For Linux, macOS, or Unix:  

```
1. aws rds create-db-instance \
2.     --db-instance-identifier mydbinstance \
3.     --engine mariadb \
4.     --engine-version 10.6.10 \
5.     --db-instance-class db.r5b.large \
6.     --master-user-password password \
7.     --master-username admin \
8.     --allocated-storage 200
```
For Windows:  

```
1. aws rds create-db-instance ^
2.     --db-instance-identifier mydbinstance ^
3.     --engine mariadb ^
4.     --engine-version 10.6.10 ^
5.     --db-instance-class db.r5b.large ^
6.     --master-user-password password ^
7.     --master-username admin ^
8.     --allocated-storage 200
```

### RDS API<a name="rds-optimized-writes-using-api"></a>

You can create a DB instance using the [ CreateDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html) operation\. When you use this operation, make sure the `EngineVersion` and `DBInstanceClass` values support RDS Optimized Writes\. In addition, make sure the parameter group associated with the DB instance has the `rds.optimized_writes` parameter set to `AUTO`\. 

## Limitations for RDS Optimized Writes<a name="rds-optimized-writes-limitations-mariadb"></a>

The following limitations apply to RDS Optimized Writes: 
+ You can only modify a database to turn on RDS Optimized Writes if the database was created with a DB engine version and DB instance class that support the feature\. In this case, if RDS Optimized Writes is turned off for the database, you can turn it on by setting the `rds.optimized_writes` parameter to `AUTO`\. For more information, see [Using RDS Optimized Writes](#rds-optimized-writes-using-mariadb)\.
+ You can only modify a database to turn on RDS Optimized Writes if the database was created *after* the feature was released\. The underlying file system format and organization that RDS Optimized Writes needs is incompatible with the file system format of databases created before the feature was released\. By extension, you can't use any snapshots of previously created instances with this feature because the snapshots use the older, incompatible file system\. 
**Important**  
To convert from the old format to the new format, you need to perform a full database migration\. If you want to use this feature on DB instances that were created *before* the feature was released, create a new empty DB instance and manually migrate your older DB instance to the newer DB instance\. You can migrate your older DB instance using the native `mysqldump` tool, replication, or AWS Database Migration Service\. For more information, see [mariadb\-dump/mysqldump](https://mariadb.com/kb/en/mariadb-dumpmysqldump/) in the MariaDB documentation, [Working with MariaDB replication in Amazon RDS](USER_MariaDB.Replication.md), and the [https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)\. For help with migrating using AWS tools, contact support\.
+ When you are restoring an RDS for MariaDB database from a snapshot, you can only turn on RDS Optimized Writes for the database if all of the following conditions apply:
  + The snapshot was created from a database that supports RDS Optimized Writes\.
  + The snapshot was created from a database that was created *after* RDS Optimized Writes was released\.
  + The snapshot is restored to a database that supports RDS Optimized Writes\.
  + The restored database is associated with a parameter group that has the `rds.optimized_writes` parameter set to `AUTO`\.
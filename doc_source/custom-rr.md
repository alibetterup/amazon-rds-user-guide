# Working with Oracle replicas for RDS Custom for Oracle<a name="custom-rr"></a>

You can create Oracle replicas for RDS Custom for Oracle DB instances\. Both container databases \(CDBs\) and non\-CDBs are supported\. Creating an RDS Custom for Oracle replica is similar to creating an RDS for Oracle replica, but with important differences\. For general information about creating and managing Oracle replicas, see [Working with DB instance read replicas](USER_ReadRepl.md) and [Working with read replicas for Amazon RDS for Oracle](oracle-read-replicas.md)\.

**Topics**
+ [Overview of RDS Custom for Oracle replication](#custom-rr.overview)
+ [Guidelines and limitations for RDS Custom for Oracle replication](#custom-rr.reqs-limitations)
+ [Promoting an RDS Custom for Oracle replica to a standalone DB instance](#custom-rr.promoting)

## Overview of RDS Custom for Oracle replication<a name="custom-rr.overview"></a>

The architecture of RDS Custom for Oracle replication is analogous to RDS for Oracle replication\. A primary DB instance replicates asynchronously to one or more Oracle replicas\.

![\[RDS Custom for Oracle supports Oracle replicas\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/read-replica-custom-oracle.png)

### Maximum number of replicas<a name="custom-rr.overview.number"></a>

As with RDS for Oracle, you can create up to five managed Oracle replicas of your RDS Custom for Oracle primary DB instance\. You can also create your own manually configured \(external\) Oracle replicas\. External replicas don't count toward your DB instance limit\. They also lie outside the RDS Custom support perimeter\. For more information about the support perimeter, see [RDS Custom support perimeter and unsupported configurations](custom-troubleshooting.md#custom-troubleshooting.support-perimeter)\.

### Replica naming convention<a name="custom-rr.overview.names"></a>

Oracle replica names are based on the database unique name\. The format is `DB_UNIQUE_NAME_X`, with letters appended sequentially\. For example, if your database unique name is `ORCL`, the first two replicas are named `ORCL_A` and `ORCL_B`\. The first six letters, A–F, are reserved for RDS Custom\. RDS Custom copies database parameters from your primary DB instance to the replicas\. For more information, see [DB\_UNIQUE\_NAME](https://docs.oracle.com/database/121/REFRN/GUID-3547C937-5DDA-49FF-A9F9-14FF306545D8.htm#REFRN10242) in the Oracle documentation\.

### Replica backup retention<a name="custom-rr.overview.backups"></a>

By default, RDS Custom Oracle replicas use the same backup retention period as your primary DB instance\. You can modify the backup retention period to 1–35 days\. RDS Custom supports backing up, restoring, and point\-in\-time recovery \(PITR\)\. For more information about backing up and restoring RDS Custom DB instances, see [Backing up and restoring an Amazon RDS Custom for Oracle DB instance](custom-backup.md)\.

**Note**  
While creating a Oracle replica, RDS Custom temporarily pauses the cleanup of redo log files\. In this way, RDS Custom ensures that it can apply these logs to the new Oracle replica after it becomes available\.

### Replica promotion<a name="custom-rr.overview.promotion"></a>

You can promote managed Oracle replicas in RDS Custom for Oracle using the console, `promote-read-replica` AWS CLI command, or `PromoteReadReplica` API\. If you delete your primary DB instance, and all replicas are healthy, RDS Custom for Oracle promotes your managed replicas to standalone instances automatically\. If a replica has paused automation or is outside the support perimeter, you must fix the replica before RDS Custom can promote it automatically\. You can only promote external Oracle replicas manually\.

## Guidelines and limitations for RDS Custom for Oracle replication<a name="custom-rr.reqs-limitations"></a>

When you create RDS Custom for Oracle replicas, not all RDS Oracle replica options are supported\.

**Topics**
+ [General guidelines for RDS Custom for Oracle replication](#custom-rr.guidelines)
+ [General limitations for RDS Custom for Oracle replication](#custom-rr.limitations)
+ [Networking requirements and limitations for RDS Custom for Oracle replication](#custom-rr.network)
+ [External replica limitations for RDS Custom for Oracle](#custom-rr.external-replica-reqs)
+ [Replica promotion limitations for RDS Custom for Oracle](#custom-rr.promotion-reqs)
+ [Replica promotion guidelines for RDS Custom for Oracle](#custom-rr.promotion-guidelines)

### General guidelines for RDS Custom for Oracle replication<a name="custom-rr.guidelines"></a>

When working with RDS Custom for Oracle, follow these guidelines:
+ Don't modify the `RDS_DATAGUARD` user\. This user is reserved for RDS Custom for Oracle automation\. Modifying this user can result in undesired outcomes, such as an inability to create Oracle replicas for your RDS Custom for Oracle DB instance\.
+ Don't change the replication user password\. It is required to administer the Oracle Data Guard configuration on the RDS Custom host\. If you change the password, RDS Custom for Oracle might put your Oracle replica outside the support perimeter\. For more information, see [RDS Custom support perimeter and unsupported configurations](custom-troubleshooting.md#custom-troubleshooting.support-perimeter)\.

  The password is stored in AWS Secrets Manager, tagged with the DB resource ID\. Each Oracle replica has its own secret in Secrets Manager\. The format for the secret is the following\.

  ```
  do-not-delete-rds-custom-db-DB_resource_id-6-digit_UUID-dg
  ```
+ Don't change the `DB_UNIQUE_NAME` for the primary DB instance\. Changing the name causes any restore operation to become stuck\.
+ Don't specify the clause `STANDBYS=NONE` in a `CREATE PLUGGABLE DATABASE` command in an RDS Custom CDB\. This way, if a failover occurs, your standby CDB contains all PDBs\.

### General limitations for RDS Custom for Oracle replication<a name="custom-rr.limitations"></a>

RDS Custom for Oracle replicas have the following limitations:
+ You can't create RDS Custom for Oracle replicas in read\-only mode\. However, you can manually change the mode of mounted replicas to read\-only, and from read\-only to mounted\. For more information, see the documentation for the [create\-db\-instance\-read\-replica](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance-read-replica.html) AWS CLI command\.
+ You can't create cross\-Region RDS Custom for Oracle replicas\.
+ You can't change the value of the Oracle Data Guard `CommunicationTimeout` parameter\. This parameter is set to 15 seconds for RDS Custom for Oracle DB instances\.

### Networking requirements and limitations for RDS Custom for Oracle replication<a name="custom-rr.network"></a>

Make sure that your network configuration supports RDS Custom for Oracle replicas\. Consider the following:
+ Make sure to enable port 1140 for both inbound and outbound communication within your virtual private cloud \(VPC\) for the primary DB instance and all of its replicas\. This is required for Oracle Data Guard communication between read replicas\.
+ RDS Custom for Oracle validates the network while creating a Oracle replica\. If the primary DB instance and the new replica can't connect over the network, RDS Custom for Oracle doesn't create the replica and places it in the `INCOMPATIBLE_NETWORK` state\.
+ For external Oracle replicas, such as those you create on Amazon EC2 or on\-premises, use another port and listener for Oracle Data Guard replication\. Trying to use port 1140 could cause conflicts with RDS Custom automation\.
+ The `/rdsdbdata/config/tnsnames.ora` file contains network service names mapped to listener protocol addresses\. Note the following requirements and recommendations:
  + Entries in `tnsnames.ora` prefixed with `rds_custom_` are reserved for RDS Custom when handling Oracle replica operations\.

    When creating manual entries in `tnsnames.ora`, don't use this prefix\.
  + In some cases, you might want to switch over or fail over manually, or use failover technologies such as Fast\-Start Failover \(FSFO\)\. If so, make sure to manually synchronize `tnsnames.ora` entries from the primary DB instance to all of the standby instances\. This recommendation applies to both Oracle replicas managed by RDS Custom and to external Oracle replicas\.

    RDS Custom automation updates `tnsnames.ora` entries on only the primary DB instance\. Make sure also to synchronize when you add or remove a Oracle replica\.

    If you don't synchronize the `tnsnames.ora` files and switch over or fail over manually, Oracle Data Guard on the primary DB instance might not be able to communicate with the Oracle replicas\.

### External replica limitations for RDS Custom for Oracle<a name="custom-rr.external-replica-reqs"></a>

 RDS Custom for Oracle external replicas, which include on\-premises replicas, have the following limitations:
+ RDS Custom for Oracle doesn't detect instance role changes upon manual failover, such as FSFO, for external Oracle replicas\.

  RDS Custom for Oracle does detect changes for managed replicas\. The role change is noted in the event log\. You can also see the new state by using the [describe\-db\-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) AWS CLI command\.
+ RDS Custom for Oracle doesn't detect high replication lag for external Oracle replicas\.

  RDS Custom for Oracle does detect lag for managed replicas\. High replication lag produces the `Replication has stopped` event\. You can also see the replication status by using the [describe\-db\-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html) AWS CLI command, but there might be a delay for it to be updated\.
+ RDS Custom for Oracle doesn't promote external Oracle replicas automatically if you delete your primary DB instance\. 

  The automatic promotion feature is available only for managed Oracle replicas\. For information about promoting Oracle replicas manually, see the white paper [Enabling high availability with Data Guard on Amazon RDS Custom for Oracle](https://d1.awsstatic.com/whitepapers/enabling-high-availability-with-data-guard-on-amazon-rds-custom-for-oracle.pdf)\.

### Replica promotion limitations for RDS Custom for Oracle<a name="custom-rr.promotion-reqs"></a>

Promoting RDS Custom for Oracle managed Oracle replicas is the same as promoting RDS managed replicas, with some differences\. Note the following limitations for RDS Custom for Oracle replicas:
+ You can't promote a replica while RDS Custom for Oracle is backing it up\.
+ You can't change the backup retention period to `0` when you promote your Oracle replica\.
+ You can't promote your replica when it isn't in a healthy state\.

  If you issue `delete-db-instance` on the primary DB instance, RDS Custom for Oracle validates that each managed Oracle replica is healthy and available for promotion\. A replica might be ineligible for promotion because automation is paused or it is outside the support perimeter\. In such cases, RDS Custom for Oracle publishes an event explaining the issue so that you can repair your Oracle replica manually\.

### Replica promotion guidelines for RDS Custom for Oracle<a name="custom-rr.promotion-guidelines"></a>

When promoting a replica, note the following guidelines:
+ Don't initiate a failover while RDS Custom for Oracle is promoting your replica\. Otherwise, the promotion workflow could become stuck\.
+ Don't switch over your primary DB instance while RDS Custom for Oracle is promoting your Oracle replica\. Otherwise, the promotion workflow could become stuck\.
+ Don't shut down your primary DB instance while RDS Custom for Oracle is promoting your Oracle replica\. Otherwise, the promotion workflow could become stuck\.
+ Don't try to restart replication with your newly promoted DB instance as a target\. After RDS Custom for Oracle promotes your Oracle replica, it becomes a standalone DB instance and no longer has the replica role\.

For more information, see [Troubleshooting replica promotion for RDS Custom for Oracle](custom-troubleshooting.md#custom-troubleshooting-promote)\.

## Promoting an RDS Custom for Oracle replica to a standalone DB instance<a name="custom-rr.promoting"></a>

Just as with RDS for Oracle, you can promote an RDS Custom for Oracle replica to a standalone DB instance\. When you promote a Oracle replica, RDS Custom for Oracle reboots the DB instance before it becomes available\. For more information about promoting Oracle replicas, see [Promoting a read replica to be a standalone DB instance](USER_ReadRepl.md#USER_ReadRepl.Promote)\.

The following steps show the general process for promoting a Oracle replica to a DB instance:

1. Stop any transactions from being written to the primary DB instance\. 

1. Wait for RDS Custom for Oracle to apply all updates to your Oracle replica\.

1. Promote your Oracle replica by choosing the **Promote** option on the Amazon RDS console, the AWS CLI command [https://docs.aws.amazon.com/cli/latest/reference/rds/promote-read-replica.html](https://docs.aws.amazon.com/cli/latest/reference/rds/promote-read-replica.html), or the [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_PromoteReadReplica.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_PromoteReadReplica.html) Amazon RDS API operation\.

Promoting a Oracle replica takes a few minutes to complete\. During the process, RDS Custom for Oracle stops replication and reboots your replica\. When the reboot completes, the Oracle replica is available as a standalone DB instance\.

### Console<a name="USER_ReadRepl.Promote.Console"></a>

**To promote an RDS Custom for Oracle replica to a standalone DB instance**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the Amazon RDS console, choose **Databases**\.

   The **Databases** pane appears\. Each Oracle replica shows **Replica** in the **Role** column\.

1. Choose the RDS Custom for Oracle replica that you want to promote\.

1. For **Actions**, choose **Promote**\.

1. On the **Promote Oracle replica** page, enter the backup retention period and the backup window for the newly promoted DB instance\. You can't set this value to **0**\.

1. When the settings are as you want them, choose **Promote Oracle replica**\.

### AWS CLI<a name="USER_ReadRepl.Promote.CLI"></a>

To promote your RDS Custom for Oracle replica to a standalone DB instance, use the AWS CLI [https://docs.aws.amazon.com/cli/latest/reference/rds/promote-read-replica.html](https://docs.aws.amazon.com/cli/latest/reference/rds/promote-read-replica.html) command\.

**Example**  
For Linux, macOS, or Unix:  

```
aws rds promote-read-replica \
--db-instance-identifier my-custom-read-replica \
--backup-retention-period 2 \
--preferred-backup-window 23:00-24:00
```
For Windows:  

```
aws rds promote-read-replica ^
--db-instance-identifier my-custom-read-replica ^
--backup-retention-period 2 ^
--preferred-backup-window 23:00-24:00
```

### RDS API<a name="USER_ReadRepl.Promote.API"></a>

To promote your RDS Custom for Oracle replica to be a standalone DB instance, call the Amazon RDS API [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_PromoteReadReplica.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_PromoteReadReplica.html) operation with the required parameter `DBInstanceIdentifier`\.
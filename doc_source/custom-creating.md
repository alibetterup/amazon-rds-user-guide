# Configuring a DB instance for Amazon RDS Custom for Oracle<a name="custom-creating"></a>

You can create an RDS Custom DB instance, and then connect to it using Secure Shell \(SSH\) or AWS Systems Manager\.

**Topics**
+ [Overview of Amazon RDS Custom for Oracle architecture](#custom-creating.overview)
+ [Creating an RDS Custom for Oracle DB instance](#custom-creating.create)
+ [RDS Custom service\-linked role](#custom-creating.slr)
+ [Connecting to your RDS Custom DB instance using SSH](#custom-creating.ssh)
+ [Connecting to your RDS Custom DB instance using AWS Systems Manager](#custom-creating.ssm)
+ [Logging in to your RDS Custom for Oracle database as SYS](#custom-creating.sysdba)

## Overview of Amazon RDS Custom for Oracle architecture<a name="custom-creating.overview"></a>

If you create an Amazon RDS Custom for Oracle DB instance with the Oracle Multitenant architecture \(`custom-oracle-ee-cdb` engine type\), your database is a container database \(CDB\)\. If you don't specify the Oracle Multitenant architecture, your database is a traditional non\-CDB that uses the `custom-oracle-ee` engine type\. A non\-CDB can't contain pluggable databases \(PDBs\)\.

An RDS Custom for Oracle CDB supports the following features:
+ Backups
+ Restoring and point\-time\-restore \(PITR\) from backups
+ Read replicas
+ Minor version upgrades

When you create a CDB instance using the Oracle Multitenant architecture, your CDB includes the following:
+ CDB root \(`CDB$ROOT`\)
+ PDB seed \(`PDB$SEED`\)
+ PDB

By default, your CDB is named `RDSCDB`, which is also the name of the Oracle System ID \(Oracle SID\), and your initial PDB is named `ORCL`\. You can choose a different name for your PDB, but the Oracle SID and the PDB name can’t be the same\.

RDS Custom for Oracle doesn't supply APIs for PDBs\. If you want additional PDBs, create them manually using Oracle SQL\. RDS Custom for Oracle doesn't restrict the number of PDBs that you can create\. In general, you are responsible for creating and managing PDBs, as in an on\-premises deployment\.

**Note**  
If you create a PDB, we recommend that you take a manual snapshot afterward in case you need to perform point\-in\-time recovery \(PITR\)\.

You can't rename existing PDBs using Amazon RDS APIs\. You also can't rename the CDB using the `modify-db-instance` command\.

The open mode for the CDB root is `READ WRITE` on the primary and `MOUNTED` on a mounted standby database\. RDS Custom for Oracle attempts to open all PDBs when opening the CDB\. If RDS Custom for Oracle can’t open all PDBs, it issues the event `tenant database shutdown`\.

## Creating an RDS Custom for Oracle DB instance<a name="custom-creating.create"></a>

Create an Amazon RDS Custom for Oracle DB instance using either the AWS Management Console or the AWS CLI\. The procedure is similar to the procedure for creating an Amazon RDS DB instance\. For more information, see [Creating an Amazon RDS DB instance](USER_CreateDBInstance.md)\.

If you included installation parameters in your CEV manifest, then your DB instance uses the Oracle base, Oracle home, and the ID and name of the UNIX/Linux user and group that you specified\. The `oratab` file, which is created by Oracle Database during installation, points to the real installation location rather than to a symbolic link\. When RDS Custom runs commands, it runs as the configured OS user rather than the default user `rdsdb`\. For more information, see [Step 5: Prepare the CEV manifest](custom-cev.preparing.md#custom-cev.preparing.manifest)\.

Before you attempt to create or connect to an RDS Custom DB instance, complete the tasks in [Setting up your environment for Amazon RDS Custom for Oracle](custom-setup-orcl.md)\.

### Console<a name="custom-creating.console"></a>

**To create an RDS Custom for Oracle DB instance**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**\.

1. Choose **Create database**\.

1. In **Choose a database creation method**, select **Standard create**\.

1. In the **Engine options** section, do the following:

   1. For **Engine type**, choose **Oracle**\.

   1. For **Database management type**, choose **Amazon RDS Custom**\.

   1. For **Architecture settings**, do one of the following:
      + Select **Multitenant architecture** to create a container database \(CDB\)\. At creation, your CDB contains one PDB and one PDB seed\.
      + Clear **Multitenant architecture** to create a non\-CDB\. A non\-CDB can't contain PDBs\.

   1. For **Edition**, choose **Oracle Enterprise Edition**\.

   1. For **Custom engine version**, choose an existing RDS Custom custom engine version \(CEV\)\. A CEV has the following format: `major-engine-version.customized_string`\. An example identifier is `19.cdb_cev1`\.

      If you chose **Multitenant architecture** in the previous step, you can only specify a Multitenant CEV\. The console filters out CEVs that were created as non\-CDBs\.

1. In **Templates**, choose **Production**\.

1. In the **Settings** section, do the following:

   1. For **DB instance identifier**, enter a unique name for your DB instance\.

   1. For **Master username**, enter a username\. You can retrieve this value from the console later\. 

      When you connect to a non\-CDB, the master user is the user for the non\-CDB\. When you connect to a CDB, the master user is the user for the PDB\. To connect to the CDB root, log in to the host, start a SQL client, and create an administrative user with SQL commands\. 

   1. Clear **Auto generate a password**\.

1. Choose a **DB instance class**\.

   For supported classes, see [DB instance class support for RDS Custom for Oracle](custom-reqs-limits.md#custom-reqs-limits.instances)\.

1. In the **Storage** section, do the following:

   1. For **Storage type**, choose an SSD type: io1, gp2, or gp3\. If you choose the io1 or gp3 storage types, also choose a rate for **Provisioned IOPS**\.

   1. For **Allocated storage**, choose a storage size\. The default is 40 GiB\.

1. For **Connectivity**, specify your **Virtual private cloud \(VPC\)**, **DB subnet group**, and **VPC security group \(firewall\)**\.

1. For **RDS Custom security**, do the following:

   1. For **IAM instance profile**, choose the instance profile for your RDS Custom for Oracle DB instance\.

      The IAM instance profile must begin with `AWSRDSCustom`, for example *AWSRDSCustomInstanceProfileForRdsCustomInstance*\.

   1. For **Encryption**, choose **Enter a key ARN** to list the available AWS KMS keys\. Then choose your key from the list\. 

      An AWS KMS key is required for RDS Custom\. For more information, see [Step 1: Create or reuse a symmetric encryption AWS KMS key](custom-setup-orcl.md#custom-setup-orcl.cmk)\.

1. For **Database options**, do the following:

   1. \(Optional\) For **Initial database name**, enter a name\. The default value is **ORCL**\. In the multitenant architecture, the initial database name is the PDB name\.

      The **System ID \(SID\)** value of `RDSCDB` is the name of the Oracle database instance that manages your database files\. In this context, the term "Oracle database instance" refers exclusively to the system global area \(SGA\) and Oracle background processes\. The Oracle SID is also the name of your CDB\. You can't change this value\.

   1. For **Backup retention period** choose a value\. You can't choose **0 days**\.

   1. For the remaining sections, specify your preferred RDS Custom DB instance settings\. For information about each setting, see [Settings for DB instances](USER_CreateDBInstance.md#USER_CreateDBInstance.Settings)\. The following settings don't appear in the console and aren't supported:
      + **Processor features**
      + **Storage autoscaling**
      + **Availability & durability**
      + **Password and Kerberos authentication** option in **Database authentication** \(only **Password authentication** is supported\)
      + **Database options** group in **Additional configuration**
      + **Performance Insights**
      + **Log exports**
      + **Enable auto minor version upgrade**
      + **Deletion protection**

1. Choose **Create database**\.
**Important**  
When you create an RDS Custom for Oracle DB instance, you might receive the following error: The service\-linked role is in the process of being created\. Try again later\. If you do, wait a few minutes and then try again to create the DB instance\.

   The **View credential details** button appears on the **Databases** page\.

   To view the master user name and password for the RDS Custom DB instance, choose **View credential details**\.

   To connect to the DB instance as the master user, use the user name and password that appear\.
**Important**  
You can't view the master user password again\. If you don't record it, you might have to change it\. If you need to change the master user password after the RDS Custom DB instance is available, modify the DB instance to do so\. For more information about modifying a DB instance, see [Managing an Amazon RDS Custom for Oracle DB instance](custom-managing.md)\.

1. Choose **Databases** to view the list of RDS Custom DB instances\.

1. Choose the RDS Custom DB instance that you just created\.

   On the RDS console, the details for the new RDS Custom DB instance appear:
   + The DB instance has a status of **creating** until the RDS Custom DB instance is created and ready for use\. When the state changes to **available**, you can connect to the DB instance\. Depending on the instance class and storage allocated, it can take several minutes for the new DB instance to be available\.
   + **Role** has the value **Instance \(RDS Custom\)**\.
   + **RDS Custom automation mode** has the value **Full automation**\. This setting means that the DB instance provides automatic monitoring and instance recovery\.

### AWS CLI<a name="custom-creating.CLI"></a>

You create an RDS Custom DB instance by using the [create\-db\-instance](https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html) AWS CLI command\.

The following options are required:
+ `--db-instance-identifier`
+ `--db-instance-class` \(for a list of supported instance classes, see [DB instance class support for RDS Custom for Oracle](custom-reqs-limits.md#custom-reqs-limits.instances)\)
+ `--engine engine_type` \(where *`engine-type`* is `custom-oracle-ee-cdb` for a CDB and `custom-oracle-ee` for a non\-CDB\)
+ `--engine-version cev` \(where *`cev`* is the name of the custom engine version that you specified in [Creating a CEV](custom-cev.create.md)\)
+ `--kms-key-id`
+ `--no-auto-minor-version-upgrade`
+ `--custom-iam-instance-profile`

The following example creates an RDS Custom DB instance named `my-cdb-instance`\. The database is a CDB\. The PDB name is *my\-pdb*\. The backup retention period is three days\.

**Example**  
For Linux, macOS, or Unix:  

```
 1. aws rds create-db-instance \
 2.     --engine custom-oracle-ee-cdb \
 3.     --db-instance-identifier my-cdb-instance \
 4.     --engine-version 19.cdb_cev1 \
 5.     --db-name my-pdb \
 6.     --allocated-storage 250 \
 7.     --db-instance-class db.m5.xlarge \
 8.     --db-subnet-group mydbsubnetgroup \
 9.     --master-username myawsuser \
10.     --master-user-password mypassword \
11.     --backup-retention-period 3 \
12.     --no-multi-az \
13.     --port 8200 \
14.     --license-model bring-your-own-license \
15.     --kms-key-id my-kms-key \
16.     --no-auto-minor-version-upgrade \
17.     --custom-iam-instance-profile AWSRDSCustomInstanceProfileForRdsCustomInstance
```
For Windows:  

```
 1. aws rds create-db-instance ^
 2.     --engine custom-oracle-ee-cdb ^
 3.     --db-instance-identifier my-cdb-instance ^
 4.     --engine-version 19.cdb_cev1 ^
 5.     --db-name my-pdb ^
 6.     --allocated-storage 250 ^
 7.     --db-instance-class db.m5.xlarge ^
 8.     --db-subnet-group mydbsubnetgroup ^
 9.     --master-username myawsuser ^
10.     --master-user-password mypassword ^
11.     --backup-retention-period 3 ^
12.     --no-multi-az ^
13.     --port 8200 ^
14.     --license-model bring-your-own-license ^
15.     --kms-key-id my-kms-key ^
16.     --no-auto-minor-version-upgrade ^
17.     --custom-iam-instance-profile AWSRDSCustomInstanceProfileForRdsCustomInstance
```

Get details about your instance by using the `describe-db-instances` command\.

**Example**  

```
1. aws rds describe-db-instances --db-instance-identifier my-cdb-instance
```
The following partial output shows the engine, parameter groups, and other information\.  

```
 1. {
 2.     "DBInstances": [
 3.         {
 4.             "PendingModifiedValues": {},
 5.             "Engine": "custom-oracle-ee-cdb",
 6.             "MultiAZ": false,
 7.             "DBSecurityGroups": [],
 8.             "DBParameterGroups": [
 9.                 {
10.                     "DBParameterGroupName": "default.custom-oracle-ee-cdb-19",
11.                     "ParameterApplyStatus": "in-sync"
12.                 }
13.             ],
14.             "AutomationMode": "full",
15.             "DBInstanceIdentifier": "my-cdb-instance",
16.             ...
17.             "TagList": [
18.                 {
19.                     "Key": "AWSRDSCustom",
20.                     "Value": "custom-oracle"
21.                 }
22.             ]
23.         }
24.     ]
25. }
```

## RDS Custom service\-linked role<a name="custom-creating.slr"></a>

A *service\-linked role* gives Amazon RDS Custom access to resources in your AWS account\. It makes using RDS Custom easier because you don't have to manually add the necessary permissions\. RDS Custom defines the permissions of its service\-linked roles, and unless defined otherwise, only RDS Custom can assume its roles\. The defined permissions include the trust policy and the permissions policy, and that permissions policy can't be attached to any other IAM entity\.

When you create an RDS Custom DB instance, both the Amazon RDS and RDS Custom service\-linked roles are created \(if they don't already exist\) and used\. For more information, see [Using service\-linked roles for Amazon RDS](UsingWithRDS.IAM.ServiceLinkedRoles.md)\.

The first time that you create an RDS Custom for Oracle DB instance, you might receive the following error: The service\-linked role is in the process of being created\. Try again later\. If you do, wait a few minutes and then try again to create the DB instance\.

## Connecting to your RDS Custom DB instance using SSH<a name="custom-creating.ssh"></a>

After you create your RDS Custom DB instance, you can connect to this instance using an SSH client\. The procedure is the same as for connecting to an Amazon EC2 instance\. For more information, see [Connecting to your Linux instance using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)\.

To connect to the DB instance, you need the key pair associated with the instance\. RDS Custom creates the key pair on your behalf\. The pair name uses the prefix `do-not-delete-rds-custom-ssh-privatekey-db-`\. AWS Secrets Manager stores your private key as a secret\.

Complete the task in the following steps:

1. [Configure your DB instance to allow SSH connections](#custom-managing.ssh.port-22)

1. [Retrieve your secret key](#custom-managing.ssh.obtaining-key)

1. [Connect to your EC2 instance using the ssh utility](#custom-managing.ssh.connecting)

### Configure your DB instance to allow SSH connections<a name="custom-managing.ssh.port-22"></a>

Make sure that your DB instance security group permits inbound connections on port 22 for TCP\.

### Retrieve your secret key<a name="custom-managing.ssh.obtaining-key"></a>

Retrieve the secret key using either AWS Management Console or the AWS CLI\.

#### Console<a name="custom-managing.ssh.obtaining-key.console"></a>

**To retrieve the secret key**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**, and then choose the RDS Custom DB instance to which you want to connect\.

1. Choose **Configuration**\.

1. Note the **Resource ID** value\. For example, the resource ID might be `db-ABCDEFGHIJKLMNOPQRS0123456`\.

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Find the name of your EC2 instance, and choose the instance ID associated with it\. For example, the EC2 instance ID might be `i-abcdefghijklm01234`\.

1. In **Details**, find **Key pair name**\. The pair name includes the resource ID\. For example, the pair name might be `do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c`\.

1. In the instance summary, find **Public IPv4 DNS**\. For the example, the public Domain Name System \(DNS\) address might be `ec2-12-345-678-901.us-east-2.compute.amazonaws.com`\.

1. Open the AWS Secrets Manager console at [https://console\.aws\.amazon\.com/secretsmanager/](https://console.aws.amazon.com/secretsmanager/)\.

1. Choose the secret that has the same name as your key pair\.

1. Choose **Retrieve secret value**\.

1. Copy the private key into a text file, and then save the file with the `.pem` extension\. For example, save the file as `/tmp/do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c.pem`\.

#### AWS CLI<a name="custom-managing.ssh.obtaining-key.CLI"></a>

To retrieve the private key, use the AWS CLI\.

**Example**  
To find the DB resource ID of your RDS Custom DB instance, use `aws rds [describe\-db\-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html)`\.  

```
aws rds describe-db-instances \
    --query 'DBInstances[*].[DBInstanceIdentifier,DbiResourceId]' \
    --output text
```
The following sample output shows the resource ID for your RDS Custom instance\. The prefix is `db-`\.  

```
db-ABCDEFGHIJKLMNOPQRS0123456
```
To find the EC2 instance ID of your DB instance, use `aws ec2 describe-instances`\. The following example uses `db-ABCDEFGHIJKLMNOPQRS0123456` for the resource ID\.  

```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=db-ABCDEFGHIJKLMNOPQRS0123456" \
    --output text \
    --query 'Reservations[*].Instances[*].InstanceId'
```
The following sample output shows the EC2 instance ID\.  

```
i-abcdefghijklm01234
```
To find the key name, specify the EC2 instance ID\.  

```
aws ec2 describe-instances \
    --instance-ids i-0bdc4219e66944afa \
    --output text \
    --query 'Reservations[*].Instances[*].KeyName'
```
The following sample output shows the key name, which uses the prefix `do-not-delete-rds-custom-ssh-privatekey-`\.  

```
do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c
```
To save the private key in a \.pem file named after the key, use `aws secretsmanager`\. The following example saves the file in your `/tmp` directory\.  

```
aws secretsmanager get-secret-value \
    --secret-id do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c \
    --query SecretString \
    --output text >/tmp/do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c.pem
```

### Connect to your EC2 instance using the ssh utility<a name="custom-managing.ssh.connecting"></a>

The following example assumes that you created a \.pem file that contains your private key\.

Change to the directory that contains your \.pem file\. Using `chmod`, set the permissions to `400`\.

```
cd /tmp
chmod 400 do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c.pem
```

To obtain your public DNS name, use the command `ec2 describe-instances`\.

```
aws ec2 describe-instances \
    --instance-ids i-abcdefghijklm01234 \
    --output text \
    --query 'Reservations[*].Instances[*].PublicDnsName'
```

The following sample output shows the public DNS name\.

```
ec2-12-345-678-901.us-east-2.compute.amazonaws.com
```

In the ssh utility, specify the \.pem file and the public DNS name of the instance\.

```
ssh -i \
    "do-not-delete-rds-custom-ssh-privatekey-db-ABCDEFGHIJKLMNOPQRS0123456-0d726c.pem" \
    ec2-user@ec2-12-345-678-901.us-east-2.compute.amazonaws.com
```

## Connecting to your RDS Custom DB instance using AWS Systems Manager<a name="custom-creating.ssm"></a>

After you create your RDS Custom DB instance, you can connect to it using AWS Systems Manager Session Manager\. Session Manager is an Systems Manager capability that you can use to manage Amazon EC2 instances through a browser\-based shell or through the AWS CLI\. For more information, see [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)\.

### Console<a name="custom-managing.ssm.console"></a>

**To connect to your DB instance using Session Manager**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**, and then choose the RDS Custom DB instance to which you want to connect\.

1. Choose **Configuration**\.

1. Note the **Resource ID** for your DB instance\. For example, the resource ID might be `db-ABCDEFGHIJKLMNOPQRS0123456`\.

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Look for the name of your EC2 instance, and then click the instance ID associated with it\. For example, the instance ID might be `i-abcdefghijklm01234`\.

1. Choose **Connect**\.

1. Choose **Session Manager**\.

1. Choose **Connect**\.

   A window opens for your session\.

### AWS CLI<a name="custom-managing.ssm.CLI"></a>

You can connect to your RDS Custom DB instance using the AWS CLI\. This technique requires the Session Manager plugin for the AWS CLI\. To learn how to install the plugin, see [Install the Session Manager plugin for the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)\.

To find the DB resource ID of your RDS Custom DB instance, use `aws rds [describe\-db\-instances](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-instances.html)`\.

```
aws rds describe-db-instances \
    --query 'DBInstances[*].[DBInstanceIdentifier,DbiResourceId]' \
    --output text
```

The following sample output shows the resource ID for your RDS Custom instance\. The prefix is `db-`\.

```
db-ABCDEFGHIJKLMNOPQRS0123456
```

To find the EC2 instance ID of your DB instance, use `aws ec2 describe-instances`\. The following example uses `db-ABCDEFGHIJKLMNOPQRS0123456` for the resource ID\.

```
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=db-ABCDEFGHIJKLMNOPQRS0123456" \
    --output text \
    --query 'Reservations[*].Instances[*].InstanceId'
```

The following sample output shows the EC2 instance ID\.

```
i-abcdefghijklm01234
```

Use the `aws ssm start-session` command, supplying the EC2 instance ID in the `--target` parameter\.

```
aws ssm start-session --target "i-abcdefghijklm01234"
```

A successful connection looks like the following\.

```
Starting session with SessionId: yourid-abcdefghijklm1234
[ssm-user@ip-123-45-67-89 bin]$
```

## Logging in to your RDS Custom for Oracle database as SYS<a name="custom-creating.sysdba"></a>

After you create your RDS Custom DB instance, you can log in to your Oracle database as user `SYS`, which gives you `SYSDBA` privileges\. You have the following login options:
+ Get the `SYS` password from Secrets Manager, and specify this password in your SQL client\.
+ Use OS authentication to log in to your database\. In this case, you don't need a password\.

### Finding the SYS password for your RDS Custom for Oracle database<a name="custom-creating.sysdba.pwd"></a>

Your can log in to your Oracle database as `SYS` or `SYSTEM` or by specifying the master user name in an API call\. The password for `SYS` and `SYSTEM` is stored in Secrets Manager\. The secret uses the naming format do\-not\-delete\-rds\-custom\-*resource\_id*\-*uuid*\. You can find the password using the AWS Management Console\.

#### Console<a name="custom-creating.sysdba.pwd.console"></a>

**To find the SYS password for your database in Secrets Manager**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the RDS console, complete the following steps:

   1. In the navigation pane, choose **Databases**\.

   1. Choose the name of your RDS Custom for Oracle DB instance\.

   1. Choose **Configuration**\.

   1. Copy the value underneath **Resource ID**\. For example, you resource ID might be **db\-ABC12CDE3FGH4I5JKLMNO6PQR7**\.

1. Open the Secrets Manager console at [https://console\.aws\.amazon\.com/secretsmanager/](https://console.aws.amazon.com/secretsmanager/)\.

1. In the Secrets Manager console, complete the following steps:

   1. In the left navigation pane, choose **Secrets**\.

   1. Filter the secrets by the resource ID that you copied in step 5\.

   1. Choose the secret named **do\-not\-delete\-rds\-custom\-*resource\_id*\-*uuid***, where *resource\_id* is the resource ID that you copied in step 5\. For example, if your resource ID is **db\-ABC12CDE3FGH4I5JKLMNO6PQR7**, your secret will be named **do\-not\-delete\-rds\-custom\-db\-ABC12CDE3FGH4I5JKLMNO6PQR7**\.

   1. In **Secret value**, choose **Retrieve secret value**\.

   1. In **Key/value**, copy the value for **password**\.

1. Install SQL\*Plus on your DB instance and log in to your database as `SYS`\. For more information, see [Step 4: Connect your SQL client to an Oracle DB instance](CHAP_GettingStarted.CreatingConnecting.Oracle.md#CHAP_GettingStarted.Connecting.Oracle)\.

### Logging in to your RDS Custom for Oracle database using OS authentication<a name="custom-creating.sysdba.pwd"></a>

The OS user `rdsdb` owns the Oracle database binaries\. You can switch to the `rdsdb` user and log in to your RDS Custom for Oracle database without a password\.

1. Connect to your DB instance with AWS Systems Manager\. For more information, see [Connecting to your RDS Custom DB instance using AWS Systems Manager](#custom-creating.ssm)\.

1. In a web browser, go to [https://www\.oracle\.com/database/technologies/instant\-client/linux\-x86\-64\-downloads\.html](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html)\.

1. For the latest database version that appears on the web page, copy the \.rpm links \(not the \.zip links\) for the Instant Client Basic Package and SQL\*Plus Package\. For example, the following links are for Oracle Database version 21\.9:
   + https://download\.oracle\.com/otn\_software/linux/instantclient/219000/oracle\-instantclient\-basic\-21\.9\.0\.0\.0\-1\.el8\.x86\_64\.rpm
   + https://download\.oracle\.com/otn\_software/linux/instantclient/219000/oracle\-instantclient\-sqlplus\-21\.9\.0\.0\.0\-1\.el8\.x86\_64\.rpm

1. In your SSH session, run the `wget` command to the download the \.rpm files from the links that you obtained in the previous step\. The following example downloads the \.rpm files for Oracle Database version 21\.9:

   ```
   wget https://download.oracle.com/otn_software/linux/instantclient/219000/oracle-instantclient-basic-21.9.0.0.0-1.el8.x86_64.rpm
   wget https://download.oracle.com/otn_software/linux/instantclient/219000/oracle-instantclient-sqlplus-21.9.0.0.0-1.el8.x86_64.rpm
   ```

1. Install the packages by running the `yum` command as follows:

   ```
   sudo yum install oracle-instantclient-*.rpm
   ```

1. Switch to the `rdsdb` user\.

   ```
   sudo su - rdsdb
   ```

1. Log in to your database using OS authentication\.

   ```
   $ sqlplus / as sysdba
   
   SQL*Plus: Release 21.0.0.0.0 - Production on Wed Apr 12 20:11:08 2023
   Version 21.9.0.0.0
   
   Copyright (c) 1982, 2020, Oracle.  All rights reserved.
   
   
   Connected to:
   Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
   Version 19.10.0.0.0
   ```
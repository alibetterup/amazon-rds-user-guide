# Connecting an EC2 instance and a Multi\-AZ DB cluster automatically<a name="multiaz-ec2-rds-connect"></a>

You can use the RDS console to simplify setting up a connection between an EC2 instance and a Multi\-AZ DB cluster\.You can use the RDS console to simplify setting up a connection between an EC2 instance and a Multi\-AZ DB cluster\. Often, your Multi\-AZ DB cluster is in a private subnet and your EC2 instance is in a public subnet within a VPC\. You can use a SQL client on your EC2 instance to connect to your Multi\-AZ DB cluster\. The EC2 instance can also run web servers or applications that access your private Multi\-AZ DB cluster\. 

![\[Automatically connect a Multi-AZ DB cluster with an EC2 instance.\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/multi-az-ec2-connect-overview.png)

If you want to connect to an EC2 instance that isn't in the same VPC as the Multi\-AZ DB cluster, see the scenarios in [Scenarios for accessing a DB instance in a VPC](USER_VPC.Scenarios.md)\.

**Topics**
+ [Overview of automatic connectivity with an EC2 instance](#multiaz-ec2-rds-connect-overview)
+ [Connecting an EC2 instance and a Multi\-AZ DB cluster automatically](#multiaz-ec2-rds-connect-connecting)
+ [Viewing connected compute resources](#multiaz-ec2-rds-connect-viewing)

## Overview of automatic connectivity with an EC2 instance<a name="multiaz-ec2-rds-connect-overview"></a>

When you set up a connection between an EC2 instance and a Multi\-AZ DB cluster automatically, RDS configures the VPC security group for your EC2 instance and for your DB cluster\.

The following are requirements for connecting an EC2 instance with a Multi\-AZ DB cluster:
+ The EC2 instance must exist in the same VPC as the Multi\-AZ DB cluster\.

  If no EC2 instances exist in the same VPC, the console provides a link to create one\.
+ The user who is setting up connectivity must have permissions to perform the following EC2 operations:
  + `ec2:AuthorizeSecurityGroupEgress` 
  + `ec2:AuthorizeSecurityGroupIngress` 
  + `ec2:CreateSecurityGroup` 
  + `ec2:DescribeInstances` 
  + `ec2:DescribeNetworkInterfaces` 
  + `ec2:DescribeSecurityGroups` 
  + `ec2:ModifyNetworkInterfaceAttribute` 
  + `ec2:RevokeSecurityGroupEgress` 

When you set up a connection to an EC2 instance, RDS takes an action based on the current configuration of the security groups associated with the Multi\-AZ DB cluster and EC2 instance, as described in the following table\.


| Current RDS security group configuration | Current EC2 security group configuration | RDS action | 
| --- | --- | --- | 
|  There are one or more security groups associated with the Multi\-AZ DB cluster with a name that matches the pattern `rds-ec2-n` \(where `n` is a number\)\. A security group that matches the pattern hasn't been modified\. This security group has only one inbound rule with the VPC security group of the EC2 instance as the source\.  |  There are one or more security groups associated with the EC2 instance with a name that matches the pattern `rds-ec2-n` \(where `n` is a number\)\. A security group that matches the pattern hasn't been modified\. This security group has only one outbound rule with the VPC security group of the Multi\-AZ DB cluster as the source\.  |  RDS takes no action\. A connection was already configured automatically between the EC2 instance and Multi\-AZ DB cluster\. Because a connection already exists between the EC2 instance and the RDS database, the security groups aren't modified\.  | 
|  Either of the following conditions apply: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multiaz-ec2-rds-connect.html)  |  Either of the following conditions apply: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multiaz-ec2-rds-connect.html)  |  [RDS action: create new security groups](ec2-rds-connect.md#rds-action-create-new-security-groups)  | 
|  There are one or more security groups associated with the Multi\-AZ DB cluster with a name that matches the pattern `rds-ec2-n`\. A security group that matches the pattern hasn't been modified\. This security group has only one inbound rule with the VPC security group of the EC2 instance as the source\.  |  There are one or more security groups associated with the EC2 instance with a name that matches the pattern `ec2-rds-n`\. However, none of these security groups can be used for the connection with the Multi\-AZ DB cluster\. A security group can't be used if it doesn't have one outbound rule with the VPC security group of the Multi\-AZ DB cluster as the source\. A security group also can't be used if it has been modified\.  |  [RDS action: create new security groups](ec2-rds-connect.md#rds-action-create-new-security-groups)  | 
|  There are one or more security groups associated with the Multi\-AZ DB cluster with a name that matches the pattern `rds-ec2-n`\. A security group that matches the pattern hasn't been modified\. This security group has only one inbound rule with the VPC security group of the EC2 instance as the source\.  |  A valid EC2 security group for the connection exists, but it is not associated with the EC2 instance\. This security group has a name that matches the pattern `rds-ec2-n`\. It hasn't been modified\. It has only one outbound rule with the VPC security group of the Multi\-AZ DB cluster as the source\.  |  [RDS action: associate EC2 security group](ec2-rds-connect.md#rds-action-associate-ec2-security-group)  | 
|  Either of the following conditions apply: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multiaz-ec2-rds-connect.html)  |  There are one or more security groups associated with the EC2 instance with a name that matches the pattern `rds-ec2-n`\. A security group that matches the pattern hasn't been modified\. This security group has only one outbound rule with the VPC security group of the Multi\-AZ DB cluster as the source\.  |  [RDS action: create new security groups](ec2-rds-connect.md#rds-action-create-new-security-groups)  | 

**RDS action: create new security groups**  
RDS takes the following actions:
+ Creates a new security group that matches the pattern `rds-ec2-n`\. This security group has an inbound rule with the VPC security group of the EC2 instance as the source\. This security group is associated with the Multi\-AZ DB cluster and allows the EC2 instance to access the Multi\-AZ DB cluster\.
+ Creates a new security group that matches the pattern `ec2-rds-n`\. This security group has an outbound rule with the VPC security group of the Multi\-AZ DB cluster as the source\. This security group is associated with the EC2 instance and allows the EC2 instance to send traffic to the Multi\-AZ DB cluster\.

**RDS action: associate EC2 security group**  
RDS associates the valid, existing EC2 security group with the EC2 instance\. This security group allows the EC2 instance to send traffic to the Multi\-AZ DB cluster\.

## Connecting an EC2 instance and a Multi\-AZ DB cluster automatically<a name="multiaz-ec2-rds-connect-connecting"></a>

Before setting up a connection between an EC2 instance and an RDS database, make sure you meet the requirements described in [Overview of automatic connectivity with an EC2 instance](ec2-rds-connect.md#ec2-rds-connect-overview)\.

If you make changes to security groups after you configure connectivity, the changes might affect the connection between the EC2 instance and the RDS database\.

**Note**  
You can only set up a connection between an EC2 instance and an RDS database automatically by using the AWS Management Console\. You can't set up a connection automatically with the AWS CLI or RDS API\.

**To connect an EC2 instance and an RDS database automatically**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**, and then choose the RDS database\.

1. For **Actions**, choose **Set up EC2 connection**\.

   The **Set up EC2 connection** page appears\.

1. On the **Set up EC2 connection** page, choose the EC2 instance\.  
![\[Set up EC2 connection page.\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/auto-connect-rds-ec2-set-up.png)

   If no EC2 instances exist in the same VPC, choose **Create EC2 instance** to create one\. In this case, make sure the new EC2 instance is in the same VPC as the RDS database\.

1. Choose **Continue**\.

   The **Review and confirm** page appears\.  
![\[EC2 connection review and confirmation page.\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/auto-connect-rds-ec2-confirm.png)

1. On the **Review and confirm** page, review the changes that RDS will make to set up connectivity with the EC2 instance\.

   If the changes are correct, choose **Confirm and set up**\.

   If the changes aren't correct, choose **Previous** or **Cancel**\.

## Viewing connected compute resources<a name="multiaz-ec2-rds-connect-viewing"></a>

You can use the AWS Management Console to view the compute resources that are connected to an RDS database\. The resources shown include compute resource connections that were set up automatically\. You can set up connectivity with compute resources automatically in the following ways:
+ You can select the compute resource when you create the database\.

  For more information, see [Creating an Amazon RDS DB instance](USER_CreateDBInstance.md) and [Creating a Multi\-AZ DB cluster](create-multi-az-db-cluster.md)\.
+ You can set up connectivity between an existing database and a compute resource\.

  For more information, see [Connecting an EC2 instance and an RDS database automatically](ec2-rds-connect.md#ec2-rds-connect-connecting)\.

The listed compute resources don't include ones that were connected to the database manually\. For example, you can allow a compute resource to access a database manually by adding a rule to the VPC security group associated with the database\.

For a compute resource to be listed, the following conditions must apply:
+ The name of the security group associated with the compute resource matches the pattern `ec2-rds-n` \(where `n` is a number\)\.
+ The security group associated with the compute resource has an outbound rule with the port range set to the port used by the RDS database\.
+ The security group associated with the compute resource has an outbound rule with the source set to a security group associated with the RDS database\.
+ The name of the security group associated with the RDS database matches the pattern `rds-ec2-n` \(where `n` is a number\)\.
+ The security group associated with the RDS database has an inbound rule with the port range set to the port used by the RDS database\.
+ The security group associated with the RDS database has an inbound rule with the source set to a security group associated with the compute resource\.

**To connect an EC2 instance and an RDS database automatically**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**, and then choose the name of the RDS database\.

1. On the **Connectivity & security** tab, view the compute resources in the **Connected compute resources**\.  
![\[Connected compute resources.\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/ec2-connected-compute-resources.png)
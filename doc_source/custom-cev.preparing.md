# Preparing to create a CEV<a name="custom-cev.preparing"></a>

To create a CEV, access the installation files and patches that are stored in your Amazon S3 bucket for any of the following releases:
+ Oracle Database 19c
+ Oracle Database 18c
+ Oracle Database 12c Release 2 \(12\.2\)
+ Oracle Database 12c Release 1 \(12\.1\)

For example, you can use the April 2021 RU/RUR for Oracle Database 19c, or any valid combination of installation files and patches\. For more information on the versions and Regions supported by RDS Custom for Oracle, see [RDS Custom with RDS for Oracle](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RDS_Fea_Regions_DB-eng.Feature.RDSCustom.html#Concepts.RDS_Fea_Regions_DB-eng.Feature.RDSCustom.ora)\.

**Topics**
+ [Step 1 \(Optional\): Download the manifest templates](#custom-cev.preparing.templates)
+ [Step 2: Download your database installation files and patches from Oracle Software Delivery Cloud](#custom-cev.preparing.download)
+ [Step 3: Upload your installation files to Amazon S3](#custom-cev.preparing.s3)
+ [Step 4 \(Optional\): Share your installation media in S3 across AWS accounts](#custom-cev.preparing.accounts)
+ [Step 5: Prepare the CEV manifest](#custom-cev.preparing.manifest)
+ [Step 6 \(Optional\): Validate the CEV manifest](#custom-cev.preparing.validating)
+ [Step 7: Add necessary IAM permissions](#custom-cev.preparing.iam)

## Step 1 \(Optional\): Download the manifest templates<a name="custom-cev.preparing.templates"></a>

A *CEV manifest* is a JSON document that includes the list of database installation \.zip files for your CEV\. To create a CEV, do the following:

1. Identify the Oracle database installation files that you want to include in your CEV\.

1. Download the installation files\.

1. Create a JSON manifest that lists the installation files\.

RDS Custom for Oracle provides JSON manifest templates with our recommended \.zip files for each supported Oracle Database release\. For example, the following template is for the 19\.17\.0\.0\.0 RU\.

```
{
    "mediaImportTemplateVersion": "2020-08-14",
    "databaseInstallationFileNames": [
        "V982063-01.zip"
    ],
    "opatchFileNames": [
        "p6880880_190000_Linux-x86-64.zip"
    ],
    "psuRuPatchFileNames": [
        "p34419443_190000_Linux-x86-64.zip",
        "p34411846_190000_Linux-x86-64.zip"
    ],
    "otherPatchFileNames": [
        "p28852325_190000_Linux-x86-64.zip",
        "p29997937_190000_Linux-x86-64.zip",
        "p31335037_190000_Linux-x86-64.zip",
        "p32327201_190000_Linux-x86-64.zip",
        "p33613829_190000_Linux-x86-64.zip",
        "p34006614_190000_Linux-x86-64.zip",
        "p34533061_190000_Linux-x86-64.zip",
        "p34533150_190000_Generic.zip",
        "p28730253_190000_Linux-x86-64.zip",
        "p29213893_1917000DBRU_Generic.zip",
        "p33125873_1917000DBRU_Linux-x86-64.zip",
        "p34446152_1917000DBRU_Linux-x86-64.zip"
    ]
}
```

Each template has an associated readme that includes instructions for downloading the patches, URLs for the \.zip files, and file checksums\. You can use these templates as they are or modify them with your own patches\. To review the templates, download [custom\-oracle\-manifest\.zip](samples/custom-oracle-manifest.zip) to your local disk and then open it with a file archiving application\. For more information, see [Step 5: Prepare the CEV manifest](#custom-cev.preparing.manifest)\.

## Step 2: Download your database installation files and patches from Oracle Software Delivery Cloud<a name="custom-cev.preparing.download"></a>

When you have identified the installation files that you want for your CEV, download them to your local system\. The Oracle Database installation files and patches are hosted on Oracle Software Delivery Cloud\. Each CEV requires a base release, such as Oracle Database 19c or Oracle Database 12c Release 2 \(12\.2\), and an optional list of patches\.

**To download the database installation files for Oracle Database**

1. Go to [https://edelivery.oracle.com/](https://edelivery.oracle.com/) and sign in\.

1. In the box, enter **Oracle Database Enterprise Edition** and choose **Search**\.

1. Choose one of the following base releases:
   + **DLP: Oracle Database Enterprise Edition 19\.3\.0\.0\.0 \( Oracle Database Enterprise Edition \)**\.
   + Choose **DLP: Oracle Database 12c Enterprise Edition 18\.0\.0\.0\.0 \( Oracle Database Enterprise Edition \)**\.
   + Choose **DLP: Oracle Database 12c Enterprise Edition 12\.2\.0\.1\.0 \( Oracle Database Enterprise Edition \)**\.
   + Choose **DLP: Oracle Database 12c Enterprise Edition 12\.1\.0\.2\.0 \( Oracle Database Enterprise Edition \)**\.

1. Choose **Continue**\.

1. Clear the **Download Queue** check box\.

1. Choose the option that corresponds to your base release:
   + **Oracle Database 19\.3\.0\.0\.0 \- Long Term Release**\.
   + **Oracle Database 18\.0\.0\.0\.0**
   + **Oracle Database 12\.2\.0\.1\.0**\.
   + **Oracle Database 12\.1\.0\.2\.0**\.

1. Choose **Linux x86\-64** in **Platform/Languages**\.

1. Choose **Continue**, and then sign the waiver\.

1. Choose the \.zip file that corresponds to your database release:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/custom-cev.preparing.html)

1. Download your desired Oracle patches from `updates.oracle.com` or `support.oracle.com` to your local system\. You can find the URLs for the patches in the following locations:
   + The readme files in the \.zip file that you downloaded in [Step 1 \(Optional\): Download the manifest templates](#custom-cev.preparing.templates)
   + The patches listed in each Release Update \(RU\) in [Release notes for Amazon Relational Database Service \(Amazon RDS\) for Oracle](https://docs.aws.amazon.com/AmazonRDS/latest/OracleReleaseNotes)

## Step 3: Upload your installation files to Amazon S3<a name="custom-cev.preparing.s3"></a>

Upload your Oracle installation and patch files to Amazon S3 using the AWS CLI\. The S3 bucket that contains your installation files must be in the same AWS Region as your CEV\.

Choose either of the following options:
+ Use `aws s3 cp` to upload a single \.zip file\.

  Upload each installation \.zip file separately\. Don't combine the \.zip files into a single \.zip file\.
+ Use `aws s3 sync` to upload a directory\.

List your installation files using either the AWS Management Console or the AWS CLI\.

Examples in this section use the following placeholders:
+ `install-or-patch-file.zip` – Oracle installation media file\. For example, p32126828\_190000\_Linux\-x86\-64\.zip is a patch\.
+ `my-custom-installation-files` – Your Amazon S3 bucket designated for your uploaded installation files\.
+ `123456789012/cev1` – An optional prefix in your Amazon S3 bucket\.
+ `source-bucket` – An Amazon S3 bucket where you can optionally stage files\.

The following example uploads `install-or-patch-file.zip` to the `123456789012/cev1` folder in the RDS Custom Amazon S3 bucket\. Run a separate `aws s3` command for each \.zip that you want to upload\.

For Linux, macOS, or Unix:

```
1. aws s3 cp install-or-patch-file.zip \
2.     s3://my-custom-installation-files/123456789012/cev1/
```

For Windows:

```
1. aws s3 cp install-or-patch-file.zip ^
2.     s3://my-custom-installation-files/123456789012/cev1/
```

Verify that your S3 bucket is in the AWS Region where you plan to run the `create-custom-db-engine-version` command\.

```
aws s3api get-bucket-location --bucket my-custom-installation-files
```

List the files in your RDS Custom Amazon S3 bucket as follows\.

```
aws s3 ls \
    s3://my-custom-installation-files/123456789012/cev1/
```

The following example uploads the files in your local *cev1* folder to the *123456789012/cev1* folder in your Amazon S3 bucket\.

For Linux, macOS, or Unix:

```
aws s3 sync cev1 \
    s3://my-custom-installation-files/123456789012/cev1/
```

For Windows:

```
aws s3 sync cev1 ^
    s3://my-custom-installation-files/123456789012/cev1/
```

The following example uploads all files in `source-bucket` to the **`123456789012/cev1`** folder in your Amazon S3 bucket\.

For Linux, macOS, or Unix:

```
aws s3 sync s3://source-bucket/ \
    s3://my-custom-installation-files/123456789012/cev1/
```

For Windows:

```
aws s3 sync s3://source-bucket/ ^
    s3://my-custom-installation-files/123456789012/cev1/
```

## Step 4 \(Optional\): Share your installation media in S3 across AWS accounts<a name="custom-cev.preparing.accounts"></a>

For the purposes of this section, the Amazon S3 bucket that contains your uploaded Oracle installation files is your *media bucket*\. Your organization might use multiple AWS accounts in an AWS Region\. If so, you might want to use one AWS account to populate your media bucket and a different AWS account to create CEVs\. If you don't intend to share your media bucket, skip to the next section\.

This section assumes the following: 
+ You can access the account that created your media bucket and a different account in which you intend to create CEVs\.
+ You intend to create CEVs in only one AWS Region\. If you intend to use multiple Regions, create a media bucket in each Region\.
+ You're using the CLI\. If you're using the Amazon S3 console, adapt the following steps\.

**To configure your media bucket for sharing across AWS accounts**

1. Log in to the AWS account that contains the S3 bucket into which you uploaded your installation media\.

1. Start with either a blank JSON policy template or an existing policy that you can adapt\.

   The following command retrieves an existing policy and saves it as *my\-policy\.json*\. In this example, the S3 bucket containing your installation files is named *oracle\-media\-bucket*\.

   ```
   aws s3api get-bucket-policy \ 
       --bucket oracle-media-bucket \
       --query Policy \
       --output text > my-policy.json
   ```

1. Edit the media bucket permissions as follows:
   + In the `Resource` element of your template, specify the S3 bucket into which you uploaded your Oracle Database installation files\.
   + In the `Principal` element, specify the ARNs for all AWS accounts that you intend to use to create CEVs\. You can add the root, a user, or a role to the S3 bucket allow list\. For more information, see [IAM identifiers](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html) in the *AWS Identity and Access Management User Guide*\.

   ```
   {
       "Version": "2008-10-17",
       "Statement": [
           {
               "Sid": "GrantAccountsAccess",
               "Effect": "Allow",
               "Principal": {
                   "AWS": [
                       "arn:aws:iam::account-1:root",
                       "arn:aws:iam::account-2:user/user-name-with-path",
                       "arn:aws:iam::account-3:role/role-name-with-path",
                       ...
                   ]
               },
               "Action": [
                   "s3:GetObject",
                   "s3:GetObjectAcl",
                   "s3:GetObjectTagging",
                   "s3:ListBucket",
                   "s3:GetBucketLocation"
               ],
               "Resource": [
                   "arn:aws:s3:::oracle-media-bucket",
                   "arn:aws:s3:::oracle-media-bucket/*"
               ]
           }
       ]
   }
   ```

1. Attach the policy to your media bucket\.

   In the following example, *oracle\-media\-bucket* is the name of the S3 bucket that contains your installation files, and *my\-policy\.json* is the name of your JSON file\.

   ```
   aws s3api put-bucket-policy \
       --bucket oracle-media-bucket \
       --policy file://my-policy.json
   ```

1. Log in to an AWS account in which you intend to create CEVs\.

1. Verify that this account can access the media bucket in the AWS account that created it\.

   ```
   aws s3 ls --query "Buckets[].Name"
   ```

   For more information, see [aws s3 ls](https://docs.aws.amazon.com/cli/latest/reference/s3/ls.html) in the *AWS CLI Command Reference*\.

1. Create a CEV by following the steps in [Creating a CEV](custom-cev.create.md)\.

## Step 5: Prepare the CEV manifest<a name="custom-cev.preparing.manifest"></a>

A CEV manifest is a JSON document that includes the following:
+ \(Required\) The list of installation \.zip files that you uploaded to Amazon S3\. RDS Custom applies the patches in the order in which they're listed in the manifest\.
+ \(Optional\) Installation parameters that set nondefault values for the Oracle base, Oracle home, and the ID and name of the UNIX/Linux user and group\. Be aware that you can’t modify the installation parameters for an existing CEV or an existing DB instance\. You also can’t upgrade from one CEV to another CEV when the installation parameters have different settings\.

For sample CEV manifests, see the JSON templates that you downloaded in [Step 1 \(Optional\): Download the manifest templates](#custom-cev.preparing.templates)\. You can also review the samples in [CEV manifest examples](#custom-cev.preparing.manifest.examples)\.

**Topics**
+ [JSON fields in the CEV manifest](#custom-cev.preparing.manifest.fields)
+ [Creating the CEV manifest](#custom-cev.preparing.manifest.creating)
+ [CEV manifest examples](#custom-cev.preparing.manifest.examples)

### JSON fields in the CEV manifest<a name="custom-cev.preparing.manifest.fields"></a>

The following table describes the JSON fields in the manifest\.


**JSON fields in the CEV manifest**  

| JSON field | Description | 
| --- | --- | 
|  `MediaImportTemplateVersion`  |  Version of the CEV manifest\. The date is in the format `YYYY-MM-DD`\.  | 
|  `databaseInstallationFileNames`  |  Ordered list of installation files for the database\.  | 
|  `opatchFileNames`  |  Ordered list of OPatch installers used for the Oracle DB engine\. Only one value is valid\. Values for `opatchFileNames` must start with `p6880880_`\.  | 
|  `psuRuPatchFileNames`  |  The PSU and RU patches for this database\.  If you include `psuRuPatchFileNames`, `opatchFileNames` is required\. Values for `opatchFileNames` must start with `p6880880_`\.   | 
|  `OtherPatchFileNames`  |  The patches that aren't in the list of PSU and RU patches\. RDS Custom applies these patches after applying the PSU and RU patches\.  If you include `OtherPatchFileNames`, `opatchFileNames` is required\. Values for `opatchFileNames` must start with `p6880880_`\.    | 
|  `installationParameters`  |  Nondefault settings for the Oracle base, Oracle home, and the ID and name of the UNIX/Linux user and group\. You can set the following parameters: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/custom-cev.preparing.html)  | 

Each Oracle Database release has a different list of supported installation files\. When you create your CEV manifest, make sure to specify only files that are supported by RDS Custom for Oracle\. Otherwise, CEV creation fails with an error\. All patches listed in [Release notes for Amazon Relational Database Service \(Amazon RDS\) for Oracle](https://docs.aws.amazon.com/AmazonRDS/latest/OracleReleaseNotes) are supported\.

### Creating the CEV manifest<a name="custom-cev.preparing.manifest.creating"></a>

**To create a CEV manifest**

1. List all installation files that you plan to apply, in the order that you want to apply them\.

1. Correlate the installation files with the JSON fields described in [JSON fields in the CEV manifest](#custom-cev.preparing.manifest.fields)\.

1. Do either of the following:
   + Create the CEV manifest as a JSON text file\.
   + Edit the CEV manifest template when you create the CEV in the console\. For more information, see [Creating a CEV](custom-cev.create.md)\.

### CEV manifest examples<a name="custom-cev.preparing.manifest.examples"></a>

The following examples show CEV manifest files for different Oracle Database releases\. If you include a JSON field in your manifest, make sure that it isn't empty\. For example, the following CEV manifest isn't valid because `otherPatchFileNames` is empty\.

```
{
    "mediaImportTemplateVersion": "2020-08-14",
    "databaseInstallationFileNames": [
        "V982063-01.zip"
    ],
    "opatchFileNames": [
        "p6880880_190000_Linux-x86-64.zip"
    ],
    "psuRuPatchFileNames": [
        "p32126828_190000_Linux-x86-64.zip"
    ],
    "otherPatchFileNames": [
    ]
}
```

**Topics**
+ [Sample CEV manifest for Oracle Database 12c Release 1 (12.1)](#oracle-cev-manifest-12.1)
+ [Sample CEV manifest for Oracle Database 12c Release 2 (12.2)](#oracle-cev-manifest-12.2)
+ [Sample CEV manifest for Oracle Database 18c](#oracle-cev-manifest-18c)
+ [Sample CEV manifest for Oracle Database 19c](#oracle-cev-manifest-19c)

**Example Sample CEV manifest for Oracle Database 12c Release 1 \(12\.1\)**  
In the following example for the July 2021 PSU for Oracle Database 12c Release 1 \(12\.1\), RDS Custom applies the patches in the order specified\. Thus, RDS Custom applies p32768233, then p32876425, then p18759211, and so on\. The example sets new values for the UNIX user and group, and the Oracle home and Oracle base\.  

```
{
    "mediaImportTemplateVersion":"2020-08-14",
    "databaseInstallationFileNames":[
        "V46095-01_1of2.zip",
        "V46095-01_2of2.zip"
    ],
    "opatchFileNames":[
        "p6880880_121010_Linux-x86-64.zip"
    ],
    "psuRuPatchFileNames":[
        "p32768233_121020_Linux-x86-64.zip"
    ],
    "otherPatchFileNames":[
        "p32876425_121020_Linux-x86-64.zip",
        "p18759211_121020_Linux-x86-64.zip",
        "p19396455_121020_Linux-x86-64.zip",
        "p20875898_121020_Linux-x86-64.zip",
        "p22037014_121020_Linux-x86-64.zip",
        "p22873635_121020_Linux-x86-64.zip",
        "p23614158_121020_Linux-x86-64.zip",
        "p24701840_121020_Linux-x86-64.zip",
        "p25881255_121020_Linux-x86-64.zip",
        "p27015449_121020_Linux-x86-64.zip",
        "p28125601_121020_Linux-x86-64.zip",
        "p28852325_121020_Linux-x86-64.zip",
        "p29997937_121020_Linux-x86-64.zip",
        "p31335037_121020_Linux-x86-64.zip",
        "p32327201_121020_Linux-x86-64.zip",
        "p32327208_121020_Generic.zip",
        "p17969866_12102210119_Linux-x86-64.zip",
        "p20394750_12102210119_Linux-x86-64.zip",
        "p24835919_121020_Linux-x86-64.zip",
        "p23262847_12102201020_Linux-x86-64.zip",
        "p21171382_12102201020_Generic.zip",
        "p21091901_12102210720_Linux-x86-64.zip",
        "p33013352_12102210720_Linux-x86-64.zip",
        "p25031502_12102210720_Linux-x86-64.zip",
        "p23711335_12102191015_Generic.zip",
        "p19504946_121020_Linux-x86-64.zip"
    ],
    "installationParameters": {
        "unixGroupName": "dba",
        "unixGroupId": 12345,
        "unixUname": "oracle",
        "unixUid": 12345,
        "oracleHome": "/home/oracle/oracle.12.1.0.2",
        "oracleBase": "/home/oracle"
    }
}
```

**Example Sample CEV manifest for Oracle Database 12c Release 2 \(12\.2\)**  
In following example for the October 2021 PSU for Oracle Database 12c Release 2 \(12\.2\), RDS Custom applies p33261817, then p33192662, then p29213893, and so on\. The example sets new values for the UNIX user and group, and the Oracle home and Oracle base\.  

```
{
    "mediaImportTemplateVersion":"2020-08-14",
    "databaseInstallationFileNames":[
        "V839960-01.zip"
    ],
    "opatchFileNames":[
        "p6880880_122010_Linux-x86-64.zip"
    ],
    "psuRuPatchFileNames":[
        "p33261817_122010_Linux-x86-64.zip"
    ],
    "otherPatchFileNames":[
        "p33192662_122010_Linux-x86-64.zip",
        "p29213893_122010_Generic.zip",
        "p28730253_122010_Linux-x86-64.zip",
        "p26352615_12201211019DBOCT2021RU_Linux-x86-64.zip",
        "p23614158_122010_Linux-x86-64.zip",
        "p24701840_122010_Linux-x86-64.zip",
        "p25173124_122010_Linux-x86-64.zip",
        "p25881255_122010_Linux-x86-64.zip",
        "p27015449_122010_Linux-x86-64.zip",
        "p28125601_122010_Linux-x86-64.zip",
        "p28852325_122010_Linux-x86-64.zip",
        "p29997937_122010_Linux-x86-64.zip",
        "p31335037_122010_Linux-x86-64.zip",
        "p32327201_122010_Linux-x86-64.zip",
        "p32327208_122010_Generic.zip"
    ],
    "installationParameters": {
        "unixGroupName": "dba",
        "unixGroupId": 12345,
        "unixUname": "oracle",
        "unixUid": 12345,
        "oracleHome": "/home/oracle/oracle.12.2.0.1",
        "oracleBase": "/home/oracle"
    }
}
```

**Example Sample CEV manifest for Oracle Database 18c**  
In following example for the October 2021 PSU for Oracle Database 18c, RDS Custom applies p32126855, then p28730253, then p27539475, and so on\. The example sets new values for the UNIX user and group, and the Oracle home and Oracle base\.  

```
{
    "mediaImportTemplateVersion":"2020-08-14",
    "databaseInstallationFileNames":[
        "V978967-01.zip"
    ],
    "opatchFileNames":[
        "p6880880_180000_Linux-x86-64.zip"
    ],
    "psuRuPatchFileNames":[
        "p32126855_180000_Linux-x86-64.zip"
    ],
    "otherPatchFileNames":[
        "p28730253_180000_Linux-x86-64.zip",
        "p27539475_1813000DBRU_Linux-x86-64.zip",
        "p29213893_180000_Generic.zip",
        "p29374604_1813000DBRU_Linux-x86-64.zip",
        "p29782284_180000_Generic.zip",
        "p28125601_180000_Linux-x86-64.zip",
        "p28852325_180000_Linux-x86-64.zip",
        "p29997937_180000_Linux-x86-64.zip",
        "p31335037_180000_Linux-x86-64.zip",
        "p31335142_180000_Generic.zip"
    ]
    "installationParameters": {
        "unixGroupName": "dba",
        "unixGroupId": 12345,
        "unixUname": "oracle",
        "unixUid": 12345,
        "oracleHome": "/home/oracle/18.0.0.0.ru-2020-10.rur-2020-10.r1",
        "oracleBase": "/home/oracle/"
    }
}
```

**Example Sample CEV manifest for Oracle Database 19c**  
In the following example for Oracle Database 19c, RDS Custom applies p32126828, then p29213893, then p29782284, and so on\. The example sets new values for the UNIX user and group, and the Oracle home and Oracle base\.  

```
{
    "mediaImportTemplateVersion": "2020-08-14",
    "databaseInstallationFileNames": [
        "V982063-01.zip"
    ],
    "opatchFileNames": [
        "p6880880_190000_Linux-x86-64.zip"
    ],
    "psuRuPatchFileNames": [
        "p32126828_190000_Linux-x86-64.zip"
    ],
    "otherPatchFileNames": [
        "p29213893_1910000DBRU_Generic.zip",
        "p29782284_1910000DBRU_Generic.zip",
        "p28730253_190000_Linux-x86-64.zip",
        "p29374604_1910000DBRU_Linux-x86-64.zip",
        "p28852325_190000_Linux-x86-64.zip",
        "p29997937_190000_Linux-x86-64.zip",
        "p31335037_190000_Linux-x86-64.zip",
        "p31335142_190000_Generic.zip"
    ],
    "installationParameters": {
        "unixGroupName": "dba",
        "unixGroupId": 12345,
        "unixUname": "oracle",
        "unixUid": 12345,
        "oracleHome": "/home/oracle/oracle.19.0.0.0.ru-2020-04.rur-2020-04.r1.EE.1",
        "oracleBase": "/home/oracle"
    }
}
```

## Step 6 \(Optional\): Validate the CEV manifest<a name="custom-cev.preparing.validating"></a>

Optionally, verify that manifest is a valid JSON file by running the `json.tool` Python script\. For example, if you change into the directory containing a CEV manifest named `manifest.json`, run the following command\.

```
python -m json.tool < manifest.json
```

## Step 7: Add necessary IAM permissions<a name="custom-cev.preparing.iam"></a>

Make sure that the IAM principal that creates the CEV has the necessary policies described in [Step 4: Grant required permissions to your IAM principal](custom-setup-orcl.md#custom-setup-orcl.iam-user)\.
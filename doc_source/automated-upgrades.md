# Performing an Automated Upgrade<a name="automated-upgrades"></a>

You can perform an automated upgrade on your Windows Server 2008 R2 and SQL Server 2008 R2 with Service Pack 3 instances on AWS with AWS Systems Manager documents\. 

The Systems Manager automation documents provide two upgrade paths:
+ Windows Server 2008 R2 to Windows Server 2012 R2 
+ SQL Server 2008 R2 on Windows Server 2012 R2 to SQL Server 2016

**Topics**
+ [Related Services](#automated-related)
+ [Prerequisites](#automated-prereq)
+ [Upgrade Paths](#upgrade-paths)
+ [Steps for Performing an Automated Upgrade](#upgrade-steps-auto)

## Related Services<a name="automated-related"></a>

The following AWS services are used in the automated upgrade process:
+ ** AWS Systems Manager**\. AWS Systems Manager is a powerful, unified interface for centrally managing your AWS resources\. For more information, see [What is AWS Systems Manager?](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
+ **AWS Systems Manager Agent \(SSM Agent\)**\. The SSM Agent is software that runs on your EC2 instances and processes requests from the Systems Manager service in the cloud to configure your machine\. SSM Agent sends status and execution information back to the Systems Manager service with EC2 messaging\. For more information, see [Installing and Configuring SSM Agent\.](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)
+ **AWS Systems Manager documents**\. An SSM document defines the actions that Systems Manager performs on your managed instances\. Documents use JavaScript Object Notation \(JSON\) or YAML, and they include steps and parameters that you specify\. This topic uses two AWS Systems Manager automation documents\. For more information, see [AWS Systems Manager Documents](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-ssm-docs.html)\.

## Prerequisites<a name="automated-prereq"></a>

In order to automate your upgrade with AWS Systems Manager documents, you must perform the following tasks:
+ [Create an IAM role with the specified IAM policies](#automated-iam) to allow AWS Systems Manager to perform automation tasks on your Amazon EC2 instances and verify that you meet the prerequisites to use AWS Systems Manager\.
+ [Select the option for how you want the automation to be executed](#automated-execution-option)\. The options for execution are **Simple execution**, **Rate control**, **Multi\-account and Region**, and **Manual execution**\. 

### Create IAM Role with Specified Permissions<a name="automated-iam"></a>

For steps on how to create an IAM role in order to allow AWS Systems Manager to access resources on your behalf, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html)\. This topic also contains information on how to verify that your account meets the prerequisites to use AWS Systems Manager\.

### Select Execution Option<a name="automated-execution-option"></a>

When you select **Automation** on the Systems Manager console, select **Execute**\. You are then prompted to choose an automation execution option\. You choose from the following options\. In the steps for the paths provided in this topic, we use the Simple execution option\.

**Simple Execution**  
Choose this option if you want to update a single instance but do not want to go through each automation step to audit the results\. This option is explained in further detail in the upgrade steps that follow\.

**Rate control**

Choose this option if you want to apply the upgrade to more than one instance\. You define the following settings\.
+ **Parameter**

  This setting, which is also set in Multi\-Account and Region settings, defines how your automation branches out\.
+ **Targets**

  Select the target to which you want to apply the automation\. This setting is also set in Multi\-Account and Region settings\.
+ **Parameter Values**

  Use the values defined in the automation document parameters\.
+ **Resource Group**

  In AWS, a resource is an entity you can work with\. Examples include Amazon EC2 instances, AWS CloudFormation stacks, or Amazon S3 buckets\. If you work with multiple resources, it may be useful to manage them as a group as opposed to moving from one AWS service to another for every task\. In some cases, you may want to manage large numbers of related resources, such as EC2 instances that make up an application layer\. In this case, you will likely need to perform bulk actions on these resources at one time\.
+ **Tags**

  Tags help you categorize your AWS resources in different ways, for example, by purpose, owner, or environment\. This categorization is useful when you have many resources of the same type\. You can quickly identify a specific resource using the assigned tags\.
+ **Rate Control**

  Rate Control is also set in Multi\-Account and Region settings\. When you set the rate control parameters, you define how many of your fleet to apply the automation to, either by target count or by percentage of the fleet\.

 **Multi\-Account and Region**

In addition to the parameters specified under Rate Control that are also used in the Multi\-Account and Region settings, there are two additional settings: 
+ **Accounts and organizational units \(OUs\)**

  Specify multiple accounts on which you want to run the automation\.
+ **AWS Regions**

  Specify multiple AWS Regions where you want to run the automation\.

**Manual Execution**  
This option is similar to **Simple execution**, but allows you to step through each automation step and audit the results\.

## Upgrade Paths<a name="upgrade-paths"></a>

There are two upgrade paths, which use two different AWS Systems Manager Automation documents\.
+ **AWSEC2\-CloneInstanceAndUpgradeWindows**\. This script creates an Amazon Machine Image \(AMI\) from a Windows Server 2008 R2 instance in your account and upgrades this AMI to Windows Server 2012 R2\. This multi\-step process can take up to two hours to complete\. 

  In this workflow, the automation creates an AMI from the instance and then launches the new AMI in the VPC and subnet you provide\. The automation workflow performs an in\-place upgrade from Windows Server 2008 R2 to Windows Server 2012 R2\. It also updates or installs the AWS drivers required by the upgraded instance\. After the upgrade is complete, the workflow creates a new AMI and terminates the upgraded instance\.
+ **AWSEC2\-CloneInstanceAndUpgradeSQLServer**\. This script creates an AMI from an Amazon EC2 instance running SQL Server 2008 R2 SP3 in your account, and then upgrades the AMI to SQL Server 2016 SP2\. This multi\-step process can take up to two hours to complete\.

  In this workflow, the automation creates an AMI from the instance and then launches the new AMI in the subnet you provide\. The automation then performs an in\-place upgrade of SQL Server 2008 R2 to SQL Server 2016 SP2\. After the upgrade is complete, the automation creates a new AMI before terminating the upgraded instance\. 

  There are two AMIs included in the automated upgrade process:
  + **Current running instance**\. The first AMI is the current running instance, which is not upgraded\. This AMI is used to launch another instance to run the in\-place upgrade\. When the process is complete, this AMI is deleted from your account, unless you specifically request to keep the original instance\. This setting is handled by the parameter `KeepPreUpgradeImageBackUp` \(default value is `false`, which means the AMI is deleted by default\)\.
  + **Upgraded AMI**\. This AMI is the outcome of the automation process\. The second AMI includes SQL Server 2016 SP2 instead of SQL Server 2008 R2\. 

  The final result is one AMI, which is the upgraded instance of the AMI\.

  When the upgrade is complete, you can test your application functionality by launching the new AMI in your VPC\. After testing, and before you perform another upgrade, schedule application downtime before completely switching to the upgraded instance\.

## Steps for Performing an Automated Upgrade<a name="upgrade-steps-auto"></a>

**Topics**
+ [Upgrade Windows 2008 R2 to 2012 R2](#2008R2-2012R2)
+ [Upgrade SQL Server 2008 R2 to SQL Server 2016](#SQL2008R2-SQL2016)

### Upgrade Windows 2008 R2 to 2012 R2<a name="2008R2-2012R2"></a>

This upgrade path requires additional prerequisites to work successfully\. These prerequisites can be found in the automation document details for [AWSEC2\-CloneInstanceAndUpgradeWindows](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-awsec2-CloneInstanceAndUpgradeWindows.html) in the *Systems Manager User Guide*\. 

After you have verified the additional prerequisite tasks, follow these steps to upgrade your Windows 2008 R2 instance to Windows 2012 R2 by using the automation document on AWS Systems Manager\. 

1. Open AWS Systems Manager from the **AWS Management Console**\.

1. From the left navigation pane, choose **Automation**\.

1. Choose **Execute automation**\.

1. Search for the automation document called `AWSEC2-CloneInstanceAndUpgradeWindows`\.

1. When the document name appears, select it\. When you select it, the document details appear\. 

1. Select **Next** to input the parameters for this document\. Leave **Simple execution** selected at the top of the page\.

1. Enter the requested parameters based on the following guidance\.
   + `InstanceID`

     **Type:** String

     \(Required\) The instance running Windows Server 2012 R2\.
   + `InstanceProfile`\. 

     **Type:** String

     \(Required\) The IAM instance profile\. This is the IAM role used to perform the AWS Systems Manager automation against the Amazon EC2 instance and AWS AMIs\. For more information, see [ Create an Instance Profile for Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-configuring-access-role.html) in the *Systems Manager User Guide*\.
   + `SubnetId`

     **Type:** String

     \(Required\) This is the subnet for the upgrade process and where your source EC2 instance resides\. Verify that the subnet has outbound connectivity to AWS services, including Amazon S3, and also to Microsoft \(in order to download patches\)\. 
   + `KeepPreUpgradedBackUp`

     **Type:** String

     \(Optional\) If this parameter is set to `true`, the automation retains the image created from the instance\. The default setting is `false`\. 
   + `RebootInstanceBeforeTakingImage`

     **Type:** String

     \(Optional\) The default is `false` \(no reboot\)\. If this parameter is set to `true`, AWS Systems Manager reboots the instance before creating an AMI for the upgrade\.

1. After you have entered the parameters, select **Execute**\. When the automation begins, you can monitor the execution progress\.

1. When the automation completes, you will see the AMI ID\. You can launch the AMI to verify that the Windows OS is upgraded\.
**Note**  
It is not necessary for the automation to run all of the steps\. The steps are conditional based on the behavior of the automation and instance\. AWS Systems Manager might skip some steps that are not required\.  
Additionally, some steps may time out\. AWS Systems Manager attempts to upgrade and install all of the latest patches\. Sometimes, however, patches time out based on a definable timeout setting for the given step\. When this happens, the AWS Systems Manager automation continues to the next step to ensure that the internal OS is upgraded to Windows Server 2012 R2\.

1. After the automation completes, you can launch an Amazon EC2 instance using the AMI ID to review your upgrade\. For more information about how to create an Amazon EC2 instance from an AWS AMI, see [ How do I launch an EC2 instance from a custom Amazon Machine Image \(AMI\)?](https://aws.amazon.com/premiumsupport/knowledge-center/launch-instance-custom-ami/)

### Upgrade SQL Server 2008 R2 to SQL Server 2016<a name="SQL2008R2-SQL2016"></a>

This upgrade path requires additional prerequisites to work successfully\. These prerequisites can be found in the automation document details for [AWSEC2\-CloneInstanceAndUpgradeSQLServer](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-awsec2-CloneInstanceAndUpgradeSQLServer.html) in the *Systems Manager User Guide*\.

After you have verified the additional prerequisite tasks, follow these steps to upgrade your SQL Server 2008 R2 database engine to SQL Server 2016 using the automation document on AWS Systems Manager\. 

1. If you haven't already, download the SQL Server 2016 \.iso file and mount it to the source server\. 

1. After the \.iso file is mounted, copy all of the component files and place them on any volume of your choice\. 

1. Take an EBS snapshot of the volume and copy the snapshot ID onto a clipboard for later use\. For more information about creating an EBS snapshot, see [Creating an EBS Snapshot](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ebs-creating-snapshot.html) in the *Amazon Elastic Compute Cloud User Guide*\.

1. Attach the instance profile to the EC2 source instance\. This allows Systems Manager to communicate with the EC2 instance and run commands on it after it is added to the AWS Systems Manager service\. For this example, we named the role `SSM-EC2-Profile-Role` with the `AmazonEc2RoleSSM` policy attached to the role\. See [ Create an Instance Profile for Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-configuring-access-role.html)\.

1. From the Systems Manager console, select **Managed Instances** under **Shared Resources** in the left navigation pane\. Verify that your EC2 instance is visible\. If you don't see your instance, wait a few minutes and it should appear\. Otherwise, reconfirm that the AWS Systems Manager prerequisites have been met\.

1. In the left navigation pane under **Actions**, select **Automation**\.

1. In the upper right corner, select **Execute automations**\.

1. Search for the `AWSEC2-CloneInstanceAndUpgradeSQLServer` document\.

1. Select the document and choose **Next**\. 

1. Enter the requested parameters based on the following guidance\.
   + `InstanceId` 

     **Type:** String

     \(Required\) The instance running SQL Server 2008 R2 \(or later\)\. 
   + `IamInstanceProfile`

     **Type:** String

     \(Required\) The IAM instance profile\.
   + `SnapshotId`

     **Type:** String

     \(Required\) The Snapshot ID for SQL Server 2016 installation media\.
   + `SubnetId`

     **Type:** String

     \(Required\) This is the subnet for the upgrade process and where your source EC2 instance resides\. Verify that the subnet has outbound connectivity to AWS services, including Amazon S3, and also to Microsoft \(in order to download patches\)\. 
   + `KeepPreUpgradedBackUp`

     **Type:** String

     \(Optional\) If this parameter is set to `true`, the automation retains the image created from the instance\. The default setting is `false`\. 
   + `RebootInstanceBeforeTakingImage`

     **Type:** String

     \(Optional\) The default is `false` \(no reboot\)\. If this parameter is set to `true`, AWS Systems Manager reboots the instance before creating an AMI for the upgrade\.

1. After you have entered the parameters, select **Execute**\. When the automation begins, you can monitor the execution progress\.

1. When the **Execution Status** shows **Success**, choose the **Outputs** list to view the AMI information\. You can use the AMI ID to launch your SQL Server 2016 instance for the VPC of your choice\.

1. From the EC2 console, under **Images** in the left navigation pane, select **AMIs**\. You should see the new AMI\.

1. To verify that SQL Server 2016 has been successfully installed, select the new AMI and choose **Launch**\.

1. Choose the type of instance that you want for the AMI, the VPC and subnet that you want to deploy to, and the storage that you want to use\. Because you are launching the new instance from an AMI, the volumes are presented to you as an option to include within the new EC2 instance you are launching\. You can remove any of these volumes, or you can add volumes\.

1. Add a tag to help you identify your instance\.

1. Add the security group or groups to the instance\.

1. Select **Launch Instance**\.

1. Choose the tag name for the instance and select **Connect** under the **Actions** dropdown\. 

1. Verify that SQL Server 2016 is the new database engine on the new instance\.

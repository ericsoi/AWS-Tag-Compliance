## Checking System tags
## Overview
This soulution uses Lambda function to perform the following:

        1. Checks tag compliance and make changes accordingly.
        2. Send a notification to a system owner if a set of required tags are missing.
        3. Push findings to an s3 bucket.
        4. Stop instances with no tag key 'Name' or no tags at all after a specified number of days.

### Prerequisites
#### 1.IAM roles with attached policies
          a)Describe ec2 resources
          b)Create and delete tags
          c)SNS publish notification
          d)AWSLambdaBasicExecutionRole
#### 2.Lambda Function containing the following.
        a)tag.py - Main function. tags resources that have no tags with Name tag not-provided
        b)lower.py - Converts any upper case to lower case.
        c)comliance.py - Checks for tag compliance and sends results for non-compliant resources to s3 and notifies the system owner through sns 
        d)input.py - Takes user variables and passes them during execution
        e)spaces.py - Replaces spaces with hypens on the tags.
        f)instances.py - stop instances according to date tag.
#### 3. Create an SNS Topic and subscribe.

#### 4. Create a schedule on cloudwatch.
        
 ## HOW TO DEMO
 #### STEP1: Create a role as described in the Prerequisites.
 sample:        
 ```
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ec2:DescribeTags",
                "ec2:CreateTags",
                "ec2:DeleteTags",
                "ec2:DescribeVpnGateways",
                "ec2:DescribeVpnConnections",
                "ec2:DescribeAddresses",
                "ec2:DescribeVolumes",
                "ec2:DescribeVpcs",
                "ec2:DescribeImages",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeRouteTables",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeNetworkAcls",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeNetworkAcls",
                "ec2:DescribeInstances",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSnapshots",
                "ec2:DescribeCustomerGateways",
                "ec2:DescribeRegions",
                "ec2:DescribeVpnConnections",
                "ec2:DescribeVpnGateways",
                "ec2:StopInstances",
                "SNS:Publish",
                "s3:PutObject",
                "tag:TagResources",
                "tag:UntagResources",
                "tag:GetResources"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```
## Step2: Implementation
1. Download the lamda code.<br/>
        A. To perform checks on a single aws region:<br/>
        Option 1: download [singleRegionTags.zip](https://github.com/ericsoi/AWS-Tag-Compliance/singleRegionTags.zip).<br/>
        Option 2: download single region split function. [instancestags.zip](https://github.com/ericsoi/AWS-Tag-Compliance/instancestags.zip) and [resourcetags.zip](https://github.com/enquizit/DDD-EQ-Tasks/blob/development/Task%2017/resourcetags.zip) and create two functions.<br/>
        B. To perform checks on all aws regions, download [allRegionTags.zip](https://github.com/ericsoi/AWS-Tag-Compliance/allRegionTags.zip).<br/>


2. Change to AWS **Lambda** console under services
3. Type in the name of the function under **Name**
4. Select Python3.6 under **RunTime** 
5. Select or create a role with enough privilages to tag and untag resources.
6. Select **Create Function**
7. Navigate to Function code and choose **Upload a .ZIP file** under **Code entry type** and upload the downloaded zip file. 
8. On **Handler**:<br/>
A. With Option 1, replace the text with **Instances.instancetag**<br/>
B. With Option 2, replace the text with **Tag.resources** and **Instances.instancetag** on the functions created with **resourcetags** and **instancestags** respectively.
9. On the input.py file

        1. Replace required_tags with a set of your required tags.
        2. Replace sns_region with the region where your sns topic in 3 above resides
        3. Replace topic_arn with your topic arn.
        4. Replace output_bucket with your s3 bucket
        5. Replace allowed_untaged_days with your desired mumber of days to
10. Increase the values on **Memory (MB)** and **Timeout** under **Basic settings**
11. Select **Save**, then **Test**
## Step3: Scheduling 
1. Change to AWS **Cloud Watch** console under services.
2. On **Rules**, select **Create rule**.
3. Check **Schedule** under **Event Source**.
4. Check **Cron expression** and specify your schedule. eg **00 00 * * ? 2018** for every midnight.
5. Under **Targets** select the function created in step 2.
6. Select **Configure details**
7. Type in the name of your rule and select **Create rule**

## NB Instances missing the tag key 'Name' will be tagged with a stoppage time tag, making them for stoppage after seven days. <br> If no change in tags is done in the seven-day period, the instances will be stopped once the function runs on the seventh day.

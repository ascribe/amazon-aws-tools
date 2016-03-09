# Amazon-AWS-tools


### AWS Cli basic workflow

If you want to quickly list instances and their Security groups for particular environment, here is the one-liner:
```sh
    aws ec2 describe-instances --filter Name=tag:Environment,Values=ENVIRONMENT_NAME --query 'Reservations[*].Instances[*].{ID:InstanceId,SG:SecurityGroups,Tags:Tags}' --output text --profile profile_name
```
This will produce you nice output of your instances and their assigned Security groups. This can be filtered in multiple ways, but the best way is to select only first element without filters:
```sh
aws ec2 describe-instances --query 'Reservations[0].Instances[0]' --output json
```
That way you will see which elements you have in your response, and based on that you can build filter.

This is VERY important feature which will do logical AND operation for those filters, so only values which are having ALL requirements will be in the output:
```sh
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" "Name=tag:Environment,Values=ENV_NAME" "Name=tag:Project,Values=PROJECT_NAME" --profile PROFILE_NAME --query Reservations[*].Instances[*].State
```
If you specify like this (all filters together), output will be produced if ANY of those filters is found:
```sh
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running,Name=tag:Environment,Values=ENV_NAME,Name=tag:Project,Values=PROJECT_NAME" --profile PROFILE_NAME --query Reservations[*].Instances[*].State
```
If you want to query and output certain values from Tags, for example instance Name, along with other details, you go like this:
```sh
aws ec2 describe-instances --filters 'Name=tag:KEY,Values=VALUE' --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value,InstanceId,InstanceType]' --output table
```
Also, you can configure your AWS CLI to work with profiles, so for example if you have different access keys for different environments, you can access them by using –profile (as shown above).


### Other AWS Cli commands

List instanceIDs
```sh
aws ec2 describe-instances --output text --query "Reservations[].Instances[].InstanceId"
```
List Architectures
```sh
aws ec2 describe-instances --output text --query "Reservations[].Instances[].Architecture"
```
Search instances with tag “Name” and value “TAG”
```sh
aws ec2 describe-instances --filters "Name=tag:Name,Values=TAG"
```
Search instances with tag values containing “VALS”
```sh
aws ec2 describe-instances –filters "Name=tag-value,Values=*VALS*"
```
Search instances with tag values beginning with “VALS”
```sh
aws ec2 describe-instances –filters "Name=tag-value,Values=VALS*"
```
Search instances with tag keys “Name”
```sh
aws ec2 describe-instances –filters "Name=tag-key,Values=Name"
```
Delete an S3 bucket and all its contents with just one command
```sh
aws s3 rb s3://bucket-name --force
```
Recursively copy a directory and its subfolders from your PC to Amazon S3.
```sh
aws s3 cp MyFolder s3://bucket-name -- recursive [--region us-west-2]
```
Display subsets of all available ec2 ubuntu-images
```sh
aws ec2 describe-images | grep ubuntu
```
List users in a table format
```sh
aws iam list-users --output table
```
List the sizes of an S3 bucket and its contents
```
aws s3api list-objects --bucket BUCKETNAME --output json --query "[sum(Contents[].Size), length(Contents[])]"
```
Get the total data of your S3 bucket:
```sh
aws s3 ls s3://ascribebackup --recursive | grep -v -E "(Bucket: |Prefix: |LastWriteTime|^$|--)" | awk 'BEGIN {total=0}{total+=$3}END{print total/1024/1024" MB"}'
```
Get bucketsize and number of files in a list
```sh
aws s3api list-objects --bucket ascribebackup --output json --query "[sum(Contents[].Size), length(Contents[])]"
```
Move S3 bucket to different location
```sh
aws s3 sync s3://oldbucket s3://newbucket --source-region us-west-1 --region us-west-2
```
List all of your  instances that are currently stopped, and the reason for the stop
```sh
aws ec2 describe-instances --filters Name=instance-state-name,Values=stopped --region eu-west-1 --output json | jq -r .Reservations[].Instances[].StateReason.Message
```
Test one of your public CloudFormation templates
```sh
aws cloudformation validate-template --region eu-west-1 --template-url https://s3-eu-west-1.amazonaws.com/ca/ca.cftemplate
```
Request spot-instance
```sh
aws ec2 request-spot-instances  --spot-price "0.5995" --instance-count 1 --type "one-time" --launch-specification {\"ImageId\":\"ami-062c161b\",\"InstanceType\":\"c4.4xlarge\",\"Placement\":{\"AvailabilityZone\":\"eu-central-1\"},\"SecurityGroupIds\":\"sg-de11b4b7\"}
```

### s3cmd-tool

Install s3cmd
```sh
sudo apt-get install s3cmd
```
List all buckets
```sh
s3cmd ls
```
List the contents of the bucket
```sh
s3cmd ls s3://my-bucket-name
```
Upload a file into the bucket (private)
```sh
s3cmd put myfile.txt s3://my-bucket-name/myfile.txt
```
Upload a file into the bucket (public)
```sh
s3cmd put --acl-public --guess-mime-type myfile.txt s3://my-bucket-name/myfile.txt
```
Recursively upload a directory to s3
```sh
s3cmd put --recursive my-local-folder-path/ s3://my-bucket-name/mydir/
```
Download a file
```sh
s3cmd get s3://my-bucket-name/myfile.txt myfile.txt
```
Recursively download files that start with myfile
```sh
s3cmd --recursive get s3://my-bucket-name/myfile
```
Delete a file
```sh
s3cmd del s3://my-bucket-name/myfile.txt
```
Delete a bucket
```sh
s3cmd del --recursive s3://my-bucket-name/
```
Create a bucket
```sh
s3cmd mb s3://my-bucket-name
```
List bucket disk usage (human readable)
```sh
s3cmd du -H s3://my-bucket-name/
```
Sync local (source) to s3 bucket (destination)
```sh
s3cmd sync my-local-folder-path/ s3://my-bucket-name/
```
Sync s3 bucket (source) to local (destination)
```sh
s3cmd sync s3://my-bucket-name/ my-local-folder-path/
```
Do a dry-run (do not perform actual sync, but get information about what would happen)
```sh
s3cmd --dry-run sync s3://my-bucket-name/ my-local-folder-path/
```
Apply a standard shell wildcard include to sync s3 bucket (source) to local (destination)
```sh
s3cmd --include '2014-05-01*' sync s3://my-bucket-name/ my-local-folder-path/
```


### Go ahead with boto (examples)

Get instance informations

    from pprint import pprint
    import boto
    import os
    AWS_ACCESS_KEY_ID = os.environ["AWS_ACCESS_KEY_ID"]
    AWS_SECRET_ACCESS_KEY = os.environ["AWS_SECRET_ACCESS_KEY"]

    conn = boto.ec2.connect_to_region("eu-central-1",
                    aws_access_key_id=AWS_ACCESS_KEY_ID,
                    aws_secret_access_key=AWS_SECRET_ACCESS_KEY)
    ​
    reservations = conn.get_all_instances()
    instances = [i for r in reservations for i in r.instances]
    for instance in instances:
        pprint(instance.__dict__)
        break # remove this to list all instances
              # this break is just for testing to return only one record!!!!


Listing all of your EC2 Instances using boto

    import boto.ec2
    import os
    AWS_ACCESS_KEY_ID = os.environ["AWS_ACCESS_KEY_ID"]
    AWS_SECRET_ACCESS_KEY = os.environ["AWS_SECRET_ACCESS_KEY"]

    def get_ec2_instances(region):
        conn = boto.ec2.connect_to_region("eu-central-1",
                    aws_access_key_id=AWS_ACCESS_KEY_ID,
                    aws_secret_access_key=AWS_SECRET_ACCESS_KEY)
        reservations = conn.get_all_reservations()
        for reservation in reservations:
            print(region+':',reservation.instances)

        for vol in conn.get_all_volumes():
            print(region+':',vol.id)

    def main():
    #    regions = ['us-east-1','us-west-1','us-west-2','eu-west-1','sa-east-1',
    #                'ap-southeast-1','ap-southeast-2','ap-northeast-1']
        regions = ['eu-central-1']
        for region in regions:
            get_ec2_instances(region)

    if  __name__ =='__main__':
        main()


### Moving S3-buckets between Useraccounts...

Before using the SYNC command, you must give the destination AWS account access to the source AWS accounts resources by using Amazon S3 ACLs or bucket policies.
First, get the 12-digit account ID for the destination account. Next, in the source account, attach the following policy to the bucket you want to copy:

 - Bucket policy in the source AWS account

        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "DelegateS3Access",
                    "Effect": "Allow",
                    "Principal": {"AWS": "destinationAccountNumber"},
                    "Action": "s3:*", "Resource": [
                        "arn:aws:s3:::sourcebucket/*",
                        "arn:aws:s3:::sourcebucket"
                    ]
                }
            ]
        }

Next, attach a policy to a user in the destination AWS account to delegate access to the bucket in the source AWS account:

 - User or group policy in the destination AWS account

        {
            "Version": "2012-10-17",
            "Statement": {
                "Effect": "Allow",
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::sourcebucket",
                    "arn:aws:s3:::sourcebucket/*",
                    "arn:aws:s3:::destinationbucket",
                    "arn:aws:s3:::destinationbucket/*",
                ]
            }
        }

When these steps are completed, you can copy objects by using the AWS CLI.
```sh
aws s3 sync s3://sourcebucket s3://destinationbucket
```

For more information on transferring ownership of bucket objects to a different account, please refer the following links:<br>
https://aws.amazon.com/premiumsupport/knowledge-center/account-transfer-s3/<br>
http://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example2.html<br>
http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html

You can use Cross-Region Replication(CRR), wherein the copying of objects across buckets in different AWS regions is automatic, asynchronous.
Please refer the following link for more: <br>
http://docs.aws.amazon.com/AmazonS3/latest/dev/crr.html

You can use a shell script that when run using a cron, a time based job scheduler which will do the sync periodically at fixed times, dates, or intervals.
Please refer the link to how to create a cron job: <br>
http://www.thesitewizard.com/general/set-cron-job.shtml


### Best practices

######Tag EBS Volumes
Indicate name of the instance and purpose (boot, database, etc) so you know what that available state volume was used for!

######Power Down for the Weekend
Turn off your development environment during the weekends, holidays or when is not in use to save some bucks.

######Throw out unattached IP Addresses
Remember that an Elastic IP address when not in use will be charged for every hour it is not attached to an instance.

######Replace Outdated S3 Objects with Glacier 
You don't have to delete out of date S3 objects.
AWS S3 has an automated process that allows you to transfer objects over to Glacier.




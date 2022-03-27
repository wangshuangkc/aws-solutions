# Copy ami-xxxxxxxxxxxacross Partitions

Copy ami-xxxxxxxxxxxfrom/to Global to/from China or US-gov cloud

* Store ami-xxxxxxxxxxxto S3 bucket
* Download object from S3 bucket to local
* Upload object to S3 bucket in another partition
* Restore ami-xxxxxxxxxxxfrom S3 object

Reference:

* https://aws.amazon.com/about-aws/whats-new/2021/04/amazon-ec2-allows-copy-amazon-machine-images-across-aws-govcloud-aws-china-other-regions/
* https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ami-store-restore.html

## From source partition

* Set up AWS credentials of the account with the ami-xxxxxxxxxxxfor calling APIs. The principal should have at least below permissions

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:CreateStoreImageTask",
                "ec2:DescribeStoreImageTasks"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:ImageID": "ami-xxxxxxxxxxx"
                }
            }
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags",
                "ebs:ListSnapshotBlocks"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:AbortMultipartUpload",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::<image_bucket_us-east-1>/*",
                "arn:aws:s3:::<image_bucket_us-east-1>"
            ]
        }
    ]
}
```

* Create task to store ami-xxxxxxxxxxxto S3 bucket

```
aws ec2 create-store-image-task \
    --image-id ami-xxxxxxxxxxx \
    --bucket <image_bucket_us-east-1> \
    --region us-east-1
```

```
aws ec2 describe-store-image-tasks \
    --image-ids ami-xxxxxxxxxxx \
    --region us-east-1
```

* Download ami-xxxxxxxxxxxobject to local

```
aws s3api list-objects-v2 \
    --bucket <image_bucket_us-east-1> \
    --region us-east-1
```

```
aws s3api get-object \
    --bucket <image_bucket_us-east-1> \
    --key ami-xxxxxxxxxxx.bin \
    /tmp/ami-xxxxxxxxxxx.bin \
    --region us-east-1
```

* Alternatively use `aws s3 sync` to download all AMI files in the bucket

## To Target Partition

* Set up AWS credentials of the target account for calling APIs. The principal should have below permissions

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ebs:StartSnapshot",
                "ec2:CreateRestoreImageTask",
                "ec2:CreateTags",
                "ec2:DescribeTags",
                "ec2:GetEbsEncryptionByDefault"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:AbortMultipartUpload",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws-cn:s3:::<image_bucket_cn-north-1>",
                "arn:aws-cn:s3:::<image_bucket_cn-north-1>/*"
            ]
        }
    ]
}
```

* Upload ami-xxxxxxxxxxxbin object to S3

```
aws s3api put-object \
    --bucket <image_bucket_cn-north-1> \
    --key ami-xxxxxxxxxxx.bin \
    --body /tmp/ami-xxxxxxxxxxx.bin \
    --region cn-north-1
```

```
aws s3api list-objects-v2 \
    --bucket <image_bucket_cn-north-1> \
    --region cn-north-1
```

* Restore image

```
aws ec2 create-restore-image-task \
    --object-key ami-xxxxxxxxxxx.bin \
    --bucket <image_bucket_cn-north-1> \
    --name "Copied from us-east-1" \
    --region cn-north-1
```

```
aws ec2 describe-images \
    --image-ids ami-xxxxxxxxxxx \
    --region cn-north-1
```



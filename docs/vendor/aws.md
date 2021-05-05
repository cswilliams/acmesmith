#### IAM policy

##### All access (S3 + Route53 setup)

``` json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::{BUCKET-NAME}", "arn:aws:s3:::{BUCKET-NAME}/*"]
    },
    {
      "Effect": "Allow",
      "Action": ["route53:ListHostedZones", "route53:GetChange"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "route53:ChangeResourceRecordSets",
      "Resource": ["arn:aws:route53:::hostedzone/*"]
    },
    {
      "Effect": "Allow",
      "Action": "route53:ListResourceRecordSets",
      "Resource": ["arn:aws:route53:::hostedzone/*"]
    }
  ]
}
```

- You can limit allowed hosted zone by `Resource` of `route53:ChangeResourceRecordSets` grant
- `route53:ListResourceRecordSets` will be only required when `restore_to_original_records` is set

##### Only fetching certificates

``` json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::{BUCKET-NAME}/certs/*"]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::{BUCKET-NAME}"],
      "Condition": {
        "StringEquals": {
          "s3:delimiter": "/"
        },
        "StringLike": {
          "s3:prefix": "certs/*"
        }
      }
    }
  ]
}
```

#### AWS KMS key policy for customer managed keys

If you're going to use `aws_kms_id` option to use customer managed keys instead of AWS managed default KMS key for Amazon S3, use the following policy as base:

Be sure to replace `{S3-REGION}` and `{YOUR-AWS-ACCOUNT-ID}` before applying it.

``` json
{
  "Version": "2012-10-17",
  "Id": "kms-acmesmith-s3-policy",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "s3.{S3-REGION}.amazonaws.com",
          "kms:CallerAccount": "{YOUR-AWS-ACCOUNT-ID}"
        }
      }
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::{YOUR-AWS-ACCOUNT-ID}:root"
      },
      "Action": [
        "kms:Describe*",
        "kms:Get*",
        "kms:List*",
        "kms:Put*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Policy for ACM post issuing hook

``` json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["acm:ImportCertificate", "acm:AddTagsToCertificate"],
      "Resource": ["*"]
    }
  ]
}
```

Optionally you can limit resource to certificate ARN(s).


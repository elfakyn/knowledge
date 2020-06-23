# Cloudtrail Management events in CloudWatch logs

This is how cloudtrail management events appear in cloudwatch logs. The below examples are NOT Insights events.

These are sampled from real events. The values are consistent, so if you see the same value (like `AROA00000000000`) in multiple places in the same event, it was the same value everywhere.

[**AWS DOCUMENTATION ON WHAT EACH FIELD IS AND WHAT EVENT VERSIONS CONTAIN WHAT FIELDS.**](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html)

## GetPublicAccessBlock, IAM user, failure

```json
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AIDA000000000000000",
        "arn": "arn:aws:iam::555555555555:user/asdf",
        "accountId": "555555555555",
        "accessKeyId": "ASIA1111111111111",
        "userName": "asdf",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "true",
                "creationDate": "2019-09-17T16:04:02Z"
            }
        },
        "invokedBy": "signin.amazonaws.com"
    },
    "eventTime": "2019-09-17T16:04:44Z",
    "eventSource": "s3.amazonaws.com",
    "eventName": "GetBucketPublicAccessBlock",
    "awsRegion": "ap-southeast-1",
    "sourceIPAddress": "1.2.3.4",
    "userAgent": "signin.amazonaws.com",
    "errorCode": "NoSuchPublicAccessBlockConfiguration",
    "errorMessage": "The public access block configuration was not found",
    "requestParameters": {
        "publicAccessBlock": [
            ""
        ],
        "host": [
            "bucket.s3.ap-southeast-1.amazonaws.com"
        ],
        "bucketName": "bucket"
    },
    "responseElements": null,
    "additionalEventData": {
        "SignatureVersion": "SigV4",
        "CipherSuite": "ECDHE-RSA-AES128-SHA",
        "AuthenticationMethod": "AuthHeader",
        "vpcEndpointId": "vpce-abcdefab"
    },
    "requestID": "FFFFFFFFFFFFFFF",
    "eventID": "00000000-1111-2222-3333-444444444444",
    "eventType": "AwsApiCall",
    "recipientAccountId": "555555555555",
    "vpcEndpointId": "vpce-abcdefab"
}
```

## PutBucketPolicy, IAM user, success

```json
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AIDA0000000000000000",
        "arn": "arn:aws:iam::555555555555:user/testymctestface",
        "accountId": "555555555555",
        "accessKeyId": "ASIA111111111111111",
        "userName": "testymctestface",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "true",
                "creationDate": "2019-09-17T15:38:13Z"
            }
        },
        "invokedBy": "signin.amazonaws.com"
    },
    "eventTime": "2019-09-17T16:17:04Z",
    "eventSource": "s3.amazonaws.com",
    "eventName": "PutBucketPolicy",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "1.2.3.4",
    "userAgent": "signin.amazonaws.com",
    "requestParameters": {
        "bucketName": "my-bucket",
        "bucketPolicy": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "Test2",
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::666666666666:root"
                    },
                    "Action": "s3:ListBucket",
                    "Resource": "arn:aws:s3:::my-bucket"
                }
            ]
        },
        "host": [
            "s3.amazonaws.com"
        ],
        "policy": [
            ""
        ]
    },
    "responseElements": null,
    "additionalEventData": {
        "SignatureVersion": "SigV4",
        "CipherSuite": "ECDHE-RSA-AES128-SHA",
        "AuthenticationMethod": "AuthHeader",
        "vpcEndpointId": "vpce-abcdefab"
    },
    "requestID": "FFFFFFFFFFFFFF",
    "eventID": "00000000-1111-2222-3333-444444444444",
    "eventType": "AwsApiCall",
    "recipientAccountId": "555555555555",
    "vpcEndpointId": "vpce-abcdefab"
}
```

## AssumeRole, AWSService, both request and response elements

```json
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AWSService",
        "invokedBy": "trustedadvisor.amazonaws.com"
    },
    "eventTime": "2020-06-16T17:21:36Z",
    "eventSource": "sts.amazonaws.com",
    "eventName": "AssumeRole",
    "awsRegion": "eu-central-1",
    "sourceIPAddress": "trustedadvisor.amazonaws.com",
    "userAgent": "trustedadvisor.amazonaws.com",
    "requestParameters": {
        "roleArn": "arn:aws:iam::555555555555:role/aws-service-role/trustedadvisor.amazonaws.com/AWSServiceRoleForTrustedAdvisor",
        "roleSessionName": "TrustedAdvisor_555555555555_aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
        "durationSeconds": 3600
    },
    "responseElements": {
        "credentials": {
            "accessKeyId": "ASIA11111111111111",
            "expiration": "Jun 16, 2020 6:21:36 PM",
            "sessionToken": "loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooong/base64/encoded/token="
        },
        "assumedRoleUser": {
            "assumedRoleId": "AROA0000000000000000:TrustedAdvisor_555555555555_aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
            "arn": "arn:aws:sts::555555555555:assumed-role/AWSServiceRoleForTrustedAdvisor/TrustedAdvisor_555555555555_aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
        }
    },
    "requestID": "00000000-1111-2222-3333-444444444444",
    "eventID": "01010101-1212-2323-3434-454545454545",
    "resources": [
        {
            "accountId": "555555555555",
            "type": "AWS::IAM::Role",
            "ARN": "arn:aws:iam::555555555555:role/aws-service-role/trustedadvisor.amazonaws.com/AWSServiceRoleForTrustedAdvisor"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "555555555555",
    "sharedEventID": "10101010-2121-3232-4343-545454545454"
}
```

## event version 1.06

```json
{
    "eventVersion": "1.06",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROA000000000000000:QuickSight-RoleSession-9876543210",
        "arn": "arn:aws:sts::555555555555:assumed-role/aws-quicksight-service-role-v0/QuickSight-RoleSession-9876543210",
        "accountId": "555555555555",
        "accessKeyId": "ASIA111111111111",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROA000000000000000",
                "arn": "arn:aws:iam::555555555555:role/service-role/aws-quicksight-service-role-v0",
                "accountId": "555555555555",
                "userName": "aws-quicksight-service-role-v0"
            },
            "webIdFederationData": {},
            "attributes": {
                "creationDate": "2019-11-19T16:00:13Z",
                "mfaAuthenticated": "false"
            }
        }
    },
    "eventTime": "2019-11-19T16:00:23Z",
    "eventSource": "athena.amazonaws.com",
    "eventName": "GetQueryExecution",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "1.2.3.4",
    "userAgent": "sbAthenaJDBCDriver/2.0, aws-internal/3",
    "requestParameters": {
        "queryExecutionId": "12121212-3434-5656-7878-909090909090"
    },
    "responseElements": null,
    "requestID": "00000000-1111-2222-3333-444444444444",
    "eventID": "99999999-8888-7777-6666-555555555555",
    "readOnly": true,
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "555555555555"
}
```

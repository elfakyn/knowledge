# Pushing Cloudwatch alarms to SNS in a different account

You have a CloudWatch alarm in account 111111111111 and you need it to post to an SNS topic in account 222222222222. There's no special configuration needed in account 111111111111. All you need is the following policy on the SNS topic in account 222222222222:

You can add multiple `Principal`s and/or multiple `SourceOwner`s in the list, you don't need a separate statement for each account.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CrossAccount",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::111111111111:root"
        ]
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:REGION:222222222222:my-sns-topic"
    },
    {
      "Sid": "CrossAccountWithCloudWatch",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:REGION:222222222222:my-sns-topic",
      "Condition": {
        "StringEquals": {
          "AWS:SourceOwner": [
            "111111111111"
          ]
        }
      }
    }
  ]
}
```

That's all, pop it and ship it!

## Explanation

The `CrossAccount` statement allows entities in 111111111111 to push to the SNS topic. But for some reason that doesn't work with CloudWatch. Instead, you'll need the `CrossAccountWithCloudWatch` statement.

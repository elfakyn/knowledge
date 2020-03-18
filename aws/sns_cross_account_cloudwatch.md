# Posting Cloudwatch alarms to SNS cross account

You have a CloudWatch alarm in account 111111111111 and you need it to post to an SNS topic in account 222222222222. How to do this?

There's no special configuration needed in account 111111111111. All you need is the following policy on the SNS topic in account 222222222222.

If you only use CloudWatch to post to this SNS, you only need the `CrossAccountWithCloudWatch` statement. If you use something other than CloudWatch (such as a lambda publishing messages), you only need the `CrossAccount` one. If you use both you need both.

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

The `CrossAccount` statement allows entities in 111111111111 to push to the SNS topic like any normal cross-account policy, with an account principal. But for some reason that doesn't work with CloudWatch. Instead, you need to use `SourceOwner` as a condition key. I don't like having `*` in the principal, it's a spaghetti factory, but I couldn't find another way. Submit a PR if you do.

## FAQ

### Can I restrict these further? I don't want to give a whole account access.

YES! Use specific principal ARNs in the first one and add further conditions in the second one.

### What if I need more than one account to publish to this SNS?

You can add multiple `Principal`s and/or multiple `SourceOwner`s in the list, you don't need a whole separate `Statement` for each account.

### Can I add all accounts in my org without having to list them out by hand?

I don't know, but you might want to look into the `AWS:PrincipalOrgID` condition key, maybe it'll work.

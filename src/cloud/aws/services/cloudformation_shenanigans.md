# Weird cloudformation shit

You're trying to do something with CFN that should work, but doesn't. What the hell?

what you're trying to do | what you think you should use | what you should ACTUALLY use | ref
--- | --- | --- | ---
get the ARN of an SQS queue | `{"Ref" : "MyQueue"}` | `{"Fn::GetAtt" : ["MyQueue", "Arn"]}` | [1](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-properties-sqs-queues-return-values)

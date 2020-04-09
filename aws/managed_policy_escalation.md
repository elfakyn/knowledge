# AWS managed policies that give too much access

Some AWS managed policies are obviously dangerous such as `IAMFullAccess`. But did you know `AWSCloudTrailReadOnlyAccess` gives you read access to every object in s3?

Below is an incomplete list of AWS policies and the bonkers access they give.

policy | what it gives | on | why it's dangerous
--- | --- | --- | ---
AWSCloudTrailReadOnlyAccess | s3:GetObject | * | if you store PII in s3 you've just given someone access to all of it

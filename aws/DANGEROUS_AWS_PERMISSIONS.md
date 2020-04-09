# Subtly dangerous AWS managed policies

Some AWS managed policies are obviously dangerous such as `IAMFullAccess`. But did you know `AWSCloudTrailReadOnlyAccess` gives you read access to every object in s3?

policy | action it allows | on | why it's dangerous
--- | --- | --- | ---
AWSCloudTrailReadOnlyAccess | s3:GetObject | * | if you store PII in s3 you've just given someone access to all of it

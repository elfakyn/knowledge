# Misleading permission names

Some permissions don't do what their name suggests.

Permission name | What it actually does | Access level | Docs
--- | --- | ---
`glue:GetMapping` | Creates a mapping | WRITE | [[1]](https://docs.aws.amazon.com/en_us/IAM/latest/UserGuide/list_awsglue.html)

# Phantom actions

There isn't a 1:1 mapping between actions and permissions in AWS. Some actions require more permissions than you'd expect.

To do this action... | you need this permission
--- | ---
`s3:HeadObject` | `s3:GetObject`

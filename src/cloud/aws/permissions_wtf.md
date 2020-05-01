# Misleading permission names

Some permissions don't do what their name suggests.

Permission name | What it actually does | Access level
--- | --- | ---
glue:GetMapping | Creates a mapping | WRITE

# Phantom actions

There isn't a 1:1 mapping between actions and permissions in AWS. Some actions require more permissions than you'd expect.

To do this action... | you need this permission
--- | ---
`s3:HeadObject` | `s3:GetObject`

# Phantom actions, resources, and condition keys

There isn't a 1:1 mapping between actions and permissions in AWS. Ever wondered how to give HeadObject permissions? Turns out that permission doesn't exist.

To do this action... | you need this permission
--- | ---
`s3:HeadObject` | `s3:GetObject`

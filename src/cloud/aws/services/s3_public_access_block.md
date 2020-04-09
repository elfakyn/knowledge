# s3 bucket public access block

There are 4 public access block settings. They have different names in the console and in the CLI.

Setting in console | Setting in CLI | What it does | Side effects
--- | --- | --- | ---
Block public access to buckets and objects granted through new access control lists | BlockPublicAcls | Can't create new public bucket or object ACLs |
Block public access to buckets and objects granted through any access control lists | IgnorePublicAcls | Ignores all public  bucket or object ACLs |
Block public access to buckets and objects granted through new public bucket policies | BlockPublicPolicy | Can't create new public policies |
Block public and cross-account access to buckets and objects through any public bucket policies | RestrictPublicBuckets | Can't create new public **and cross-account** bucket policies, ignores existing public **and cross-account** bucket policies | Weird 500 errors on cross account policies

## I'm getting 500 errors on PutBucketPolicy, help!

It's because you've enabled `RestrictPublicBuckets` and are trying to put a cross-account policy. Disable it and it'll work. Really? A 500 error? Really.

# s3 bucket public access block

There are 4 public access block settings. They have different names in the console and in the CLI.

Setting in console | Setting in CLI | What it does
--- | --- | ---
Block public access to buckets and objects granted through new access control lists | BlockPublicAcls | Can't create new public bucket or object ACLs (it will error out if you try), existing ones work.
Block public access to buckets and objects granted through any access control lists | IgnorePublicAcls | Ignores all public bucket or object ACLs. You can create new ones but it'll ignore them.
Block public access to buckets and objects granted through new public bucket policies | BlockPublicPolicy | Can't create new public bucket policies (it will error out if you try).
Block public and cross-account access to buckets and objects through any public bucket policies | RestrictPublicBuckets | Can't create new public **and cross-account** bucket policies _(this appears to be a bug; [it gives a 500 error if you try](https://github.com/elfakyn/knowledge/blob/master/src/cloud/aws/services/s3_bucket_policy_500_errors.md))_, ignores existing public **and cross-account** bucket policies

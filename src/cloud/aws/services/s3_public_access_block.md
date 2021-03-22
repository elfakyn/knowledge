# s3 bucket public access block

There are 4 public access block settings. They have different names in the console and in the CLI.

Setting in console | Setting in CLI | What it does
--- | --- | ---
Block public access to buckets and objects granted through new access control lists | BlockPublicAcls | Can't create new public bucket or object ACLs (it will error out if you try), existing ones work.
Block public access to buckets and objects granted through any access control lists | IgnorePublicAcls | Ignores all public bucket or object ACLs. You can create new ones but it'll ignore them.
Block public access to buckets and objects granted through new public bucket policies | BlockPublicPolicy | Can't create new public bucket policies (it will error out if you try).
Block public and cross-account access to buckets and objects through any public bucket policies | RestrictPublicBuckets | This is weird. If there is any public access in the bucket policy, it will ignore both public and cross-account access. Details below.

If you don't want public access, check all 4 boxes. If you are encountering issues with cross-account access, verify that there isn't a public statement on the bucket.

I would **strongly discourage** putting private objects and public objects in the same bucket; it's a disaster waiting to happen. Use different buckets, even if it will take some effort to make that possible. AWS makes it extremely easy to mess up access control if all your objects are in the same bucket, and public buckets are a frighteningly common source of data breaches. Don't take the risk!

## What does RestrictPublicBuckets do?

If there is at least one public statement in the bucket policy, both public statements and cross-account statements are ignored.

What this means is that if you have RestrictPublicBuckets enabled in a bucket with only cross-account access, the cross-account access will work. But if you then add public access to the bucket, the cross account access will stop working (and, of course, the public access will not work either). What the fuck?

### Response from AWS Support

**RestrictPublicBuckets: Block public and cross-account access to buckets and objects through any public bucket or access point policies**

Setting this option to TRUE restricts access to an access point or bucket with a public policy to only AWS service principals and authorized users within the bucket owner's account. This setting blocks all cross-account access to the access point or bucket (except by AWS service principals), while still allowing users within the account to manage the access point or bucket.Enabling this setting doesn't affect existing access point or bucket policies, except that Amazon S3 blocks public and cross-account access derived from any public access point or bucket policy, including non-public delegation to specific accounts.

Suppose that a bucket has a policy that grants access to a set of fixed principals(`"Principal": {"AWS": "arn:aws:iam::AccountB:user/AccountBUserName"}`). Under the previously described rules, this policy isn't public. Thus, if you enable the RestrictPublicBuckets setting, the policy remains in effect as written, because RestrictPublicBuckets only applies to buckets that have public policies.

**However, if you add a public statement to the policy(`"Principal":"*"`), RestrictPublicBuckets takes effect on the bucket. It allows only AWS service principals and authorized users of the bucket owner's account to access the bucket.**

As an example, suppose that a bucket owned by "Account-1" has a policy that contains the following:

* A statement that grants access to AWS CloudTrail (which is an AWS service principal)
* A statement that grants access to account "Account-2"
* A statement that grants access to the public, for example by specifying `"Principal": "*"` with no limiting Condition

This policy qualifies as public because of the third statement. With this policy in place and RestrictPublicBuckets enabled, Amazon S3 allows access only by CloudTrail. Even though statement 2 isn't public, Amazon S3 disables access by "Account-2." This is because statement 3 renders the entire policy public, so RestrictPublicBuckets applies. As a result, Amazon S3 disables cross-account access, even though the policy delegates access to a specific account, "Account-2."

But if you remove statement 3 from the policy, then the policy doesn't qualify as public, and RestrictPublicBuckets no longer applies. Thus, "Account-2" regains access to the bucket, even if you leave RestrictPublicBuckets enabled[2].

References:  
[1] Blocking public access to your Amazon S3 storage  - https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html  
[2] The meaning of "public"  - https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html#access-control-block-public-access-policy-status

## Possible bugs

I have encountered situations in which RestirctPublicBuckets prevented me from creating new public or cross-account bucket policies _(this appears to be a bug; [it gives a 500 error if you try](https://github.com/elfakyn/knowledge/blob/master/src/cloud/aws/services/s3_bucket_policy_500_errors.md))_ I haven't been able to reliably replicate it. It may have been fixed since.

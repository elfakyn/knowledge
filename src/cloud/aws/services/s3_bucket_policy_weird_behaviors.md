# What You See is Not What You Get: Weird Behaviors in S3 Bucket Policies

Have you ever seen an S3 bucket policy change itself with no human intervention?

Have you ever encountered a 500 "Please try again later" error when setting an s3 bucket policy?

Ever wondered why you got "Invalid Principal" when writing a bucket policy, even though there's nothing obviously wrong?

Why is that one old bucket showing access to `AIDAXYZTABCDEFGHIJK` when you clearly didn't do that?

And what the hell is a Canonical ID?

AWS's documentation is confusing and contradictory on this matter. Let's take a deeper dive.

## Table of Contents


  * [Table of Contents](#table-of-contents)
  * [A necessary refresher: how access works in AWS, and why cross-account access is different](#a-necessary-refresher-how-access-works-in-aws-and-why-cross-account-access-is-different)
    + [IAM policies](#iam-policies)
    + [Examples](#examples)
    + [How policies interact](#how-policies-interact)
  * [What AWS tells you...](#what-aws-tells-you)
    + [How to grant cross-account access](#how-to-grant-cross-account-access)
    + [Example](#example)
  * [What AWS doesn't tell you: how bucket policies ACTUALLY work](#what-aws-doesnt-tell-you-how-bucket-policies-actually-work)
    + [Saving and retrieving bucket policies](#saving-and-retrieving-bucket-policies)
    + [AWS's internal representation](#awss-internal-representation)
  * [So what's the problem?](#so-whats-the-problem)
    + [If a user is deleted, all bucket policies with that user will appear to have changed](#if-a-user-is-deleted-all-bucket-policies-with-that-user-will-appear-to-have-changed)
    + [You cannot edit a bucket policy that references a deleted principal](#you-cannot-edit-a-bucket-policy-that-references-a-deleted-principal)
    + [If you delete a user/role and create one with the same ARN, all resource-based access will break](#if-you-delete-a-userrole-and-create-one-with-the-same-arn-all-resource-based-access-will-break)
        - [The AWS documentation is plain wrong](#the-aws-documentation-is-plain-wrong)
    + [You can't create a bucket policy before you've created the principal that is granted access](#you-cant-create-a-bucket-policy-before-youve-created-the-principal-that-is-granted-access)
    + [Certain bucket policies will result in a cryptic 500 error](#certain-bucket-policies-will-result-in-a-cryptic-500-error)
  * [Security Implications](#security-implications)
    + [Brute-forcing valid principal names is possible](#brute-forcing-valid-principal-names-is-possible)
    + [User compromise will break cross-account access](#user-compromise-will-break-cross-account-access)
    + [Explicit denies will stop working if the principal is deleted and recreated](#explicit-denies-will-stop-working-if-the-principal-is-deleted-and-recreated)
    + [Canonical IDs offer no extra security](#canonical-ids-offer-no-extra-security)
  * [Conclusion](#conclusion)

## A necessary refresher: how access works in AWS, and why cross-account access is different

Feel free to skip this if you have a really good understanding of AWS access.

Humans or automations do stuff in AWS. For that, they need permissions.

These permissions are encoded in "policies", which are JSON documents. [Policies are really complicated](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html), but in a simplified way they have a list of statements. Each statement has:
* A list of actions
* If the actions are `Allow`ed or `Deny`ed
* Who is allowed (or denied) to perform those actions
* What the targets of the actions can (or cannot) be
* Optionally, any additional conditions that need to apply

See "Examples" below and [a whole bunch of examples from AWS here](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html).

### IAM policies: principals and resources

For the purpose of access control, there are entities which perform actions, known as "principals", and entities that are the targets or actions, known as "resources". Some entities (such as IAM roles) can act as either principals or resources depending on context.

For the most part, you use "IAM policies" to control access (the following is a bit simplified):

* Identity-based policies: an identity-based policy (also called a "principal-based" policy) is "attached" to a principal and grants or denies the principal access to one or more resources.
* Resource-based policies: a resource-based policy is "attached" to a resource and grants or denies one or more principals access to the resource

When you have both an identity policy and a resource policy, the resulting access depends on whether the principal and resource are both in the same AWS account or are in different AWS accounts.

### Examples

Here's an identity-based policy that is attached to an EC2 instance and allows it to assume a certain role. This identity-based policy only grants access to the EC2 instance that it is attached to. If you want other EC2 instances to have this permission, you need to attach it to those as well.

```json
{ 
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::111111111111:role/ROLENAME"
        }
    ]
}
```

Here's a resource-based policy that is attached to a SecretsManager secret. This grants access to the `anaya` user to that secret. **I know it says `"Resource": "*"` but because this is attached to a single SecretsManager secret, the policy only grants access to that secret, not all secrets.** To grant access to more secrets, you need to attach the policy to those as well.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "secretsmanager:*",
            "Principal": {"AWS": "arn:aws:iam::123456789012:user/anaya"},
            "Resource": "*"
        }
    ]
}
```

### How policies interact

Here's a summary of how identity and resource policies interact where the access is within the same AWS account:

Situation | Outcome (Same AWS account)
---|---
No identity policy exists, no resource policy exists | No access
Identity policy only | Access according to identity policy
Resource policy only | Access according to resource policy
Both identity and resource policies | UNION of access of identity and resource policies (except if there's an explicit deny or other edge cases as per the policy evaluation logic)

And for access into a different account (known as "cross-account" access):

Situation | Outcome (Cross-account access)
---|---
No identity policy exists, no resource policy exists | No access
Identity policy only | No access
Resource policy only | No access
Both identity and resource policies | INTERSECTION of access of identity and resource policies

N.B. Specifically for cross-account access, you can specify an entire AWS account (e.g. `arn:aws:iam::123456789012:root`) in a resource policy to grant access to *all* principals in that account. You still need a corresponding identity policy for the access to take effects, as it is still the intersection of access. An AWS account can be referenced by the account ARN, which contains the account number (`arn:aws:iam::123456789012:root`) or by a "Canonical ID" (`1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef`). The Canonical ID is an obfuscated form of an AWS account ARN and otherwise acts the same.

Further reading on policies:
* [Identity-based policies and resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html)
* [Policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
* [Cross-account policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html)

## What AWS tells you...

### How to grant cross-account access

[This is what AWS tells you you should do to grant cross-account access to an s3 bucket](https://aws.amazon.com/premiumsupport/knowledge-center/cross-account-access-s3/):

1. Create a Bucket in Account A and a User in Account B
1. Create an identity policy that allows the User in Account B access to the Bucket in Account A. Attach it to the User in Account B.
1. Create a resource policy ("bucket policy") that allows the User in Account B access to the Bucket in Account A. Attach it to the Bucket in Account A.
1. Done!

### Example

This is what AWS tells you to attach to User B:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
  "s3:PutObject",
  "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::AccountABucketName/*"
 
        }
    ]
}
```

And this is what AWS tells you to attach to bucket A:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::AccountB:user/AccountBUserName"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::AccountABucketName/*"
            ]
        }
    ]
}
```

Further discussion on identity-based policies work is outside the scope of this document. Bucket policies are where things can go very, very wrong, in non-obvious ways.

## What AWS doesn't tell you: how bucket policies ACTUALLY work

**The JSON that you've seen so far for bucket policies is not the actual internal representation of a bucket policy.** In a bucket policy, what you see is not what you actually get, and this is the source of many problems.

There is some amount of guesswork in what follows since I don't have internal AWS access, but to my knowledge this is how things work in effect (it may be implemented differently).

### Saving and retrieving bucket policies

When you click "save" on a bucket policy:

1. AWS performs basic validation on the policy (JSON is valid, the bucket ARN is correct etc.)
1. AWS reads the bucket policy and validates if all the principals in the policy exist. If any of the principals fail to validate, you get the error "Invalid Principal in Policy"
1. AWS converts all the principal ARNs into a special backend representation:
    1. For users and roles, it looks up the user/role arn and converts them into a "Principal ID". 
    1. For AWS accounts, it looks up the AWS account number and converts it into a Canonical ID.
1. AWS stores the converted bucket policy.

When you want to see a bucket policy:
1. AWS reads the bucket policy and converts all the special backend representation IDs back into ARNs. If any conversion fails (e.g. a user no longer exists), it skips it.
1. AWS displays to you the resulting bucket policy.

These behaviors are specific for bucket policies. Other types of resource-based policies may or may not behave completely differently (like SecretsManager resource policies or assume role policies).

### AWS's internal representation

All users have an internal unique ID, called a "Principal ID" (of the form `AIDAxxxxxxxxxxxxxxxxx`), that is distinct from the ARN of the user.

All accounts have a unique ID called a "Canonical ID" that is distinct from the account number (but behaves the exact same).

What you see | What AWS stores internally
--- | ---
`arn:aws:iam::123456789012:user/SomeUser` | `AIDAxxxxxxxxxxxxxxxxx` (a user Principal ID)
`arn:aws:iam::123456789012:role/SomeRole` | `AROAxxxxxxxxxxxxxxxxx` (a role Principal ID)
`arn:aws:iam::123456789012:root` | `1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef` (a Canonical ID)

Further reading:

* [Finding your account canonical ID](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html#findingcanonicalid)
* [Finding the IAM identifiers (Principal IDs)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#identifiers-unique-ids)
* [Principal IDs, and AWS warnings on how cross-account access will break](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html). This is a fairly obscure piece of documentation.

## So what's the problem?

### If a user is deleted, all bucket policies with that user will appear to have changed

Let's say you have this bucket policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/Test"
            },
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::TestyMcTestFace"
        }
    ]
}
```

And now you delete the user `Test`. The bucket policy will now show:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "AIDAxxxxxxxxxxxxxxxxx"
            },
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::elfakyn-testytest"
        }
    ]
}
```

*The bucket policy never actually changed.* It is now displayed differently to you because the internally-stored ID can no longer be converted back into an ARN for display purposes.

### You cannot edit a bucket policy that references a deleted principal

Because there is validation every time you save a bucket policy, if the bucket policy references a deleted principal, it will error with "Invalid Principal" every time you try to edit it. You actually have to fix the invalid principal before you can save the policy.

But as long as you don't edit it, it will remain with an invalid principal without any issues.

### If you delete a user/role and create one with the same ARN, all resource-based access will break

When you delete a user and create one with the same ARN, its Principal ID (`AIDAxxxxxxxxxxxxxxxxx`) changes.

Any cross-account bucket policies use the Principal ID and not the ARN to grant access. When the user is deleted and recreated:

1. Cross-account access will break
1. Cross-account bucket policies will now display `AIDAxxxxxxxxxxxxxxxxx` instead of the user ARN

To fix the problem, you have to manually go to all bucket policies and replace them with the user ARN all over again.

The same happens with roles, except it displays `AROAxxxxxxxxxxxxxxxxx`

Access within the same account will break if it is granted using the bucket policy (and not an identity-based policy). But typically, same-account s3 access is granted using identity-based policies and not bucket policies, so same-account access will not break in most cases.

#### The AWS documentation is plain wrong

[The AWS documentation claims](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#identifiers-unique-ids):

> Suppose that the employee named David leaves your company and you delete the corresponding IAM user. But later another employee named David starts and you create a new IAM user named David. If the bucket policy specifies the David IAM user, the policy allows the new David to access information that was left by the former David.

I have tested this, and it is incorrect. If you view the bucket policy, it will show the old Principal ID and will not retain the user ARN, even within the same account.

### You can't create a bucket policy before you've created the principal that is granted access

This one is pretty self-explanatory. If a principal does not exist when you create the policy, it will return an Invalid Principal error. That means that, if you grant access to a third party via a bucket policy, you'll have to coordinate such that the principal is created first.

### Certain bucket policies will result in a cryptic 500 error

If a bucket policy is invalid in such a way that it breaks the internal validator, it will return a 500 error instead of anything useful. Here's one such example: 

```json
{
    "Version": "2012-10-17",
    "Id": "whatever",
    "Statement": [
        {
            "Sid": "ReadOnly",
            "Effect": "Allow",
            "Principal": {
                "AWS": []
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-bucket/*"
        }
    ]
}
```

[Read more here.](./s3_bucket_policy_500_errors.md)

If you have any automation that handles bucket policies, you need to be able to handle such errors.

## Security Implications

### Brute-forcing valid principal names is possible

You can use this mechanism to brute force valid AWS accounts as well as valid principals within an AWS account (such as user names). To do this, simply attempt to apply a bucket policy. If the principal is valid, it will apply successfully. If the principal doesn't exist, it will fail. AWS knows about this issue and it is intended behavior, so don't go submitting a bug report on this one; I already did, and they WontFix.

Principal names may reveal potentially sensitive information such as:

* Who works for your company
* What clients you do business with
* What technologies you use
* If you do anything sketchy security-wise (such as all-powerful machine users)

### User compromise will break cross-account access

If the credentials for an AWS user are compromised, AWS will automatically quarantine the user (they monitor places like GitHub for leaks) and the AWS Abuse Team will ABSOLUTELY INSIST that you delete the user and create a new one (in the strongest possible terms: "You're in violation of our TOS, you must absolutely delete this user, you have no choice").

You are allowed to create a new user with the same name and ARN, but cross-account access will still break. If this cross-account access involves third parties, you're in for an awkward conversation...

### Explicit denies will stop working if the principal is deleted and recreated

If you rely on explicit denies on a bucket policy to deny access to specific principals (such as users or roles), those explicit denies will stop working if the principals are deleted and recreated, since they refer to the old Principal ID.

### Canonical IDs offer no extra security

AWS advertises Canonical IDs (`1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef`) as obfuscated versions of an AWS account ARN (`arn:aws:iam::123456789012:root`). If you want to give a third party your account ARN (for instance, so they can create a cross-account bucket policy granting it access), but don't want to divulge your account number, you can give them the Canonical ID.

But this doesn't actually work. All you have to do is save the Canonical ID in a bucket policy, and next time you view the policy, [AWS will helpfully convert it back into an account ARN with an account number.](https://docs.amazonaws.cn/en_us/AmazonS3/latest/dev/s3-bucket-user-policy-specifying-principal-intro.html) Nifty!

## Conclusion

AWS works in arcane ways. It is almost certain that you will find every idiosyncracy of the system documented somewhere, but it is just as certain that it *will not be in the first place you're looking*. You may find confusing and contradictory documentation.

If you're about to make a mission-critical change, open an AWS support ticket and get the correct team, in this case, s3, to explicitly tell you what will happen (they might still be wrong!). Because whatever happens, it will be a surprise to everyone.

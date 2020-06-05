# I'm getting 500 errors on PutBucketPolicy, help!

## What the error looks like

Sometimes AWS spits a cryptic error:

![Big red error message in the AWS Console. The title of the error is "Error" and the text of the error is "We encountered an internal error. Please try again."](/assets/images/aws_s3_put_policy_500_internal_error.png)

or like this:

```json
{
  "Error": {
    "Code": "InternalError",
    "Message": "We encountered an internal error. Please try again."
  },
  "ResponseMetadata": {
    "HTTPHeaders": {
      "connection": "close",
      "content-type": "application/xml",
      "date": "Fri, 05 Jun 2020 16:43:12 GMT",
      "server": "AmazonS3",
      "transfer-encoding": "chunked",
      "x-amz-id-2": "00000000000000/1111111111111111/22222222222222222222222222222222=",
      "x-amz-request-id": "AAAAAAAAAAAAAAA"
    },
    "HTTPStatusCode": 500,
    "HostId": "00000000000000/1111111111111111/22222222222222222222222222222222=",
    "MaxAttemptsReached": true,
    "RequestId": "AAAAAAAAAAAAAAA",
    "RetryAttempts": 4
  }
}
```

Don't try again; it won't work. The error is likely due to one of the following reasons:

## You are trying to put a cross-account policy with `RestrictPublicBuckets` enabled

### Background

What the issue is:
* You have a public access block on your bucket with `RestrictPublicBuckets` enabled (also known as "Block public and cross-account access to buckets and objects through any public bucket policies")
* You are trying to put a bucket policy that grants cross-account access to the bucket

Why this happens:
* It's undocumented behavior in AWS. [READ MORE HERE](./s3_public_access_block.md)

### How to fix

Disable `RestrictPublicBuckets` ("Block public and cross-account access to buckets and objects through any public bucket policies") in your public access block.

## You are trying to put a malformed policy

### Background

Not all malformed policies will result in a `MalformedPolicy` error. 

Here's an example of a policy that will give you a 500 error:

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

### How to fix

Fix your policy.

Potential causes:
* Policy statements can't have an empty array as a principal. Add a principal or remove the statement entirely.

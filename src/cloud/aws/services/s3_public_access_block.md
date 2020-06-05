# s3 bucket public access block

There are 4 public access block settings. They have different names in the console and in the CLI.

Setting in console | Setting in CLI | What it does
--- | --- | ---
Block public access to buckets and objects granted through new access control lists | BlockPublicAcls | Can't create new public bucket or object ACLs
Block public access to buckets and objects granted through any access control lists | IgnorePublicAcls | Ignores all public  bucket or object ACLs
Block public access to buckets and objects granted through new public bucket policies | BlockPublicPolicy | Can't create new public policies
Block public and cross-account access to buckets and objects through any public bucket policies | RestrictPublicBuckets | Can't create new public **and cross-account** bucket policies _(this bit isn't documented and appears to be unintended behavior)_, ignores existing public **and cross-account** bucket policies

## I'm getting 500 errors on PutBucketPolicy, help!

It's because you've enabled `RestrictPublicBuckets` and are trying to put a cross-account policy. Disable it and it'll work.

Here's the error you will get if you try to put a cross-account policy with `RestrictPublicBuckets` enabled:

```
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

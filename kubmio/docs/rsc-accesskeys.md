# AccessKey

## Description

A common pattern for programmatic access to service like MinIO is to use 'technical account', or
'service account'. Such account are mainly credential, with a different lifecycle than a 'User' account.

Within MinIO, such account does not exist by themselves, but are bound to a `User`. They are called `AccessKey`
in the new MinIO terminology.

> A note about terminology: During its history, MinIO concept and wording has evolved. Formerly, `User` was named
`AccessKey` and what is now `AccessKey` has been named `svcacct` (for 'Service Account'). There is still some rest
of the old wording on some screens of the web console and on `mc` minio CLI command. This could be confusing.

TODO: Still to implement


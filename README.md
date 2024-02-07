# github-action-cache-s3

This action enables caching dependencies to s3 compatible storage, e.g. minio, AWS S3

The main change done for in this repo is you can use cloudfront URL to fetch you cache instead of directly downloading from your bucket.
This will reduce the outbound cost from AWS by A LOT!

Keep in mind you still need to add the AWS key access to allow this action to push new caches.

It also has github [actions/cache@v2](https://github.com/actions/cache) fallback if s3 save & restore fails

## Usage

```yaml
name: CI_DEV

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build_test:
    runs-on: [ubuntu-latest]

    steps:
      - uses: syahmi001/github-action-cache-s3@v1
        with:
          cloudfront: "Cloudfront URL" # required, need HTTPS i.e. https://abc123.cloudfront.net
          accessKey: "YOUR_ACCESS_KEY" # required
          secretKey: "YOUR_SECRET_KEY" # required
          bucket: actions-cache # required
          endpoint: play.min.io # optional, default s3.amazonaws.com
          insecure: false # optional, use http instead of https. default false
          sessionToken: "YOUR_TOKEN_KEY" # optional
          use-fallback: true # optional, use github actions cache fallback, default true

          # actions/cache compatible properties: https://github.com/actions/cache
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          path: |
            node_modules
            .cache
          restore-keys: |
            ${{ runner.os }}-yarn-
```

You can also set env instead of using `with`:

```yaml
      - uses: syahmi001/github-action-cache-s3@v1
        env:
          AWS_ACCESS_KEY_ID: "YOUR_ACCESS_KEY"
          AWS_SECRET_ACCESS_KEY: "YOUR_SECRET_KEY"
          # AWS_SESSION_TOKEN: "xxx"
          AWS_REGION: "us-east-1"
        with:
          cloudfront: https://abc123.cloudfront.net
          endpoint: play.min.io
          bucket: actions-cache
          use-fallback: false
          key: test-${{ runner.os }}-${{ github.run_id }}
          path: |
            test-cache
            ~/test-cache
```

## Restore keys

`restore-keys` works similar to how github's `@actions/cache@v2` works: It search each item in `restore-keys`
as prefix in object names and use the latest one

## Amazon S3 permissions

When using this with Amazon S3, the following permissions are necessary:

 - `s3:PutObject`
 - `s3:GetObject`
 - `s3:ListBucket`
 - `s3:GetBucketLocation`
 - `s3:ListBucketMultipartUploads`
 - `s3:ListMultipartUploadParts`

# Note on release

This project follows semantic versioning. Backward incompatible changes will
increase major version.

There is also the `v1` compatible tag that's always pinned to the latest
`v1.x.y` release.

It's done using:

```
git tag -a v1 -f -m "v1 compatible release"
git push -f --tags
```

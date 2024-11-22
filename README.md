# Upload S3 ☁️

This action upload directory to AWS S3 by [public read](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html) and output key that generated by [shortid](https://github.com/dylang/shortid)

> Note - The last source_dir name(`foo/bar/will-be-replace`)  will be replaced to the key generated as shortid.
          The reason is that upload a new one every time and I want to access s3 with a new key value. If you want to upload a single file, be sure to **create a folder** and upload it.

## Usage

### `workflow.yml` Example

Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://help.github.com/en/articles/workflow-syntax-for-github-actions)

```yaml
name: Upload to S3

on: [pull_request]

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - uses: shallwefootball/upload-s3-action@master
        with:
          aws_key_id: ${{ secrets.AWS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY}}
          aws_bucket: ${{ secrets.AWS_BUCKET }}
          source_dir: 'dirname'
```

Recommend using with [deployment-action](https://github.com/marketplace/actions/deployment-action) in pull request.

```yaml
name: Deploy for preview

on: [pull_request]

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - uses: chrnorm/deployment-action@releases/v1
        name: Create GitHub deployment
        id: test
        with:
          token: ${{ secrets.GITHUB_TOKEN}}
          description: 'Preview my app'
          environment: preview

      - uses: shallwefootball/upload-s3-action@master
        name: Upload S3
        id: S3
        with:
          aws_key_id: ${{ secrets.AWS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY}}
          aws_bucket: ${{ secrets.AWS_BUCKET }}
          source_dir: 'static'

      - name: Update deployment status (success)
        if: success()
        uses: chrnorm/deployment-status@releases/v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          target_url: https://aws-bucket.s3.ap-northeast-2.amazonaws.com/${{steps.S3.outputs.object_key}}/index.html
          state: 'success'
          deployment_id: ${{ steps.test.outputs.deployment_id }}
```

## Action inputs

The following settings must be passed as environment variables as shown in the example. Sensitive information, especially `aws_key_id` and `aws_secret_access_key`, should be [set as encrypted secrets](https://help.github.com/en/articles/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables) — otherwise, they'll be public to anyone browsing your repository's source code

| name                    | description                                                                                                                                                          |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `aws_key_id`            | (Required) Your AWS Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html)                                       |
| `aws_secret_access_key` | (Required) Your AWS Secret Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html)                                |
| `aws_bucket`            | (Required) The name of the bucket you're upload to.                                                                                                                  |
| `source_dir`            | (Required) The local directory (or file) you wish to upload to S3. The directory will replace to key generated by [shortid](https://github.com/dylang/shortid) in S3 |
| `destination_dir`       | (Optional) The destination directory in S3<br />If this field is excluded a [shortid](https://github.com/dylang/shortid) will be generated                           |
| `endpoint`              | (Optional) The endpoint URI to send requests to. [More info here.](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html)                                                                                                                    |

> To upload to the root directory, set `destination_dir: ''` in `action.yml`

## Action outputs

| name               | description                                                                             |
| ------------------ | --------------------------------------------------------------------------------------- |
| `object_key`       | Uploaded object key generated by [shortid](https://github.com/dylang/shortid) in Bucket |
| `object_locations` | Object Locations                                                                        |

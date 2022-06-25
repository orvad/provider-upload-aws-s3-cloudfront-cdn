# provider-upload-aws-s3-cloudfront-cdn

AWS S3 with Cloudfront upload provider for the wonderful [Strapi Open Source Node.js Headless CMS](https://strapi.io/)

Mostly just a copy of https://github.com/strapi/strapi/tree/master/packages/providers/upload-aws-s3. The text below is also mostly copied from Strapi's official provider plugin.

Please refer to this Strapi official [README](https://github.com/strapi/strapi/blob/master/packages/providers/upload-aws-s3/README.md) for details.

## Installation

```bash
# using yarn
yarn add provider-upload-aws-s3-cloudfront-cdn

# using npm
npm install provider-upload-aws-s3-cloudfront-cdn --save
```

### Provider Configuration

`./config/plugins.js`:

```js
module.exports = ({ env }) => ({
  // ...
  upload: {
    config: {
      provider: "aws-s3-cloudfront",
      providerOptions: {
        accessKeyId: env("AWS_ACCESS_KEY_ID"),
        secretAccessKey: env("AWS_ACCESS_SECRET"),
        region: env("AWS_REGION"),
        params: {
          Bucket: env("AWS_BUCKET"),
        },
        cdn: env("AWS_CLOUDFRONT_URL"),
      },
      actionOptions: {
        upload: {},
        uploadStream: {},
        delete: {},
      },
    },
  },
  // ...
});
```

### Security Middleware Configuration

Due to the default settings in the Strapi Security Middleware you will need to modify the `contentSecurityPolicy` settings to properly see thumbnail previews in the Media Library. You should replace `strapi::security` string with the object bellow instead as explained in the [middleware configuration](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/required/middlewares.html#loading-order) documentation.

`./config/middlewares.js`

```js
module.exports = [
  // ...
  {
    name: "strapi::security",
    config: {
      contentSecurityPolicy: {
        useDefaults: true,
        directives: {
          "connect-src": ["'self'", "https:"],
          "img-src": [
            "'self'",
            "data:",
            "blob:",
            "dl.airtable.com",
            "yourBucketName.s3.yourRegion.amazonaws.com",
          ],
          "media-src": [
            "'self'",
            "data:",
            "blob:",
            "dl.airtable.com",
            "yourBucketName.s3.yourRegion.amazonaws.com",
          ],
          upgradeInsecureRequests: null,
        },
      },
    },
  },
  // ...
];
```

If you use dots in your bucket name, the url of the ressource is in directory style (`s3.yourRegion.amazonaws.com/your.bucket.name/image.jpg`) instead of `yourBucketName.s3.yourRegion.amazonaws.com/image.jpg`. Then only add `s3.yourRegion.amazonaws.com` to img-src and media-src directives.

## Required AWS Policy Actions

These are the minimum amount of permissions needed for this provider to work.

```json
"Action": [
  "s3:PutObject",
  "s3:GetObject",
  "s3:ListBucket",
  "s3:DeleteObject",
  "s3:PutObjectAcl"
],
```

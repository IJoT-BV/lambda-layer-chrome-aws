# Google Chrome for AWS Lambda as a layer
Forked from: [shelfio/chrome-aws-lambda-layer](https://github.com/shelfio/chrome-aws-lambda-layer)

> 58 MB Google Chrome to fit inside AWS Lambda Layer compressed with Brotli

[Sparticuz/chromium](https://github.com/Sparticuz/chromium) published as a Lambda Layer.

Tested with Node.js 16x/18x/20x. Compatible with both x86_64 (x64) and arm64 architectures.
Uses Chromium v140.0.0

## Getting Started

Click on Layers and choose "Add a layer", and "Provide a layer version
ARN" and enter the following ARN.

```
arn:aws:lambda:us-east-1:764866452798:layer:chrome-aws-lambda:50
```

When importing the module within lambda, make sure you import `@sparticuz/chromium` not `chrome-aws-lambda`

```js
const chromium = require('@sparticuz/chromium');
```

**package.json**

- `@sparticuz/chromium` marked as a dependency
- `puppeteer-core` marked as a dependency

**lambda container settings**:

- x86_64 (x64) or arm64 architecture
- >= 1024mb memory
- `@sparticuz/chromium` marked as an externalModule in the bundling settings
- A lambda layer marked like so (Terraform):

**For x86_64 (x64) architecture:**
```ts
# Lambda Layer for Chrome from S3
resource "aws_lambda_layer_version" "chrome_aws_layer" {
  layer_name          = "lambda-layer-chrome-aws"
  compatible_runtimes = ["nodejs20.x"]
  description         = "Chrome AWS Lambda Layer"
  
  s3_bucket = "<Bucket Location>"
  s3_key    = "lambda-layer-chrome-aws/lambda-layer-chromium-v141.0.0.x64.zip"
}

layers = [
    aws_lambda_layer_version.chrome_aws_layer.arn
 ]
```

**For arm64 architecture:**
```ts
# Lambda Layer for Chrome from S3
resource "aws_lambda_layer_version" "chrome_aws_layer" {
  layer_name          = "lambda-layer-chrome-aws"
  compatible_runtimes = ["nodejs20.x"]
  description         = "Chrome AWS Lambda Layer"
  
  s3_bucket = "<Bucket Location>"
  s3_key    = "lambda-layer-chrome-aws/lambda-layer-chromium-v141.0.0.arm64.zip"
}

layers = [
    aws_lambda_layer_version.chrome_aws_layer.arn
 ]
```

> **Note:** Make sure to select the layer ARN that matches your Lambda function's architecture.

**In the deployed lambda code**
You can just use a regular ES6 or CommonJS import statement for `@sparticuz/chrome-aws-lambda`, and just use as
indicated.

## License

MIT © [Shelf](https://shelf.io), (C) 2025 IJoT BV

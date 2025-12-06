# Google Chrome for AWS Lambda as a layer
Forked from: [shelfio/chrome-aws-lambda-layer](https://github.com/shelfio/chrome-aws-lambda-layer)

> 58 MB Google Chrome to fit inside AWS Lambda Layer compressed with Brotli

[Sparticuz/chromium](https://github.com/Sparticuz/chromium) published as a Lambda Layer.

Tested with Node.js 16x/18x/20x. Compatible with both x86_64 (x64) and arm64 architectures.

Uses Chromium v143.0.0

## GitHub Secrets Configuration

If you're forking this repository, you'll need to configure the following GitHub secrets for the automated workflow:

| Secret Name | Description | Required |
|------------|-------------|----------|
| `S3_BUCKET_GITHUB_ARTIFACTS` | S3 bucket name where layer files will be stored | Yes |
| `AWS_ACCESS_KEY_ID` | AWS access key with S3 write permissions | Yes |
| `AWS_SECRET_ACCESS_KEY` | AWS secret access key | Yes |
| `AWS_REGION` | AWS region for S3 bucket (defaults to `eu-central-1`) | No |
| `SONAR_TOKEN` | SonarCloud authentication token for code quality analysis | Yes |

### Setting up AWS Permissions

The AWS credentials need the following IAM permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/lambda-layer-chrome-aws/*"
    }
  ]
}
```

## Getting Started

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
  
  s3_bucket = "YOUR_BUCKET_NAME"
  s3_key    = "lambda-layer-chrome-aws/lambda-layer-chrome-aws-v143.0.0.x64.zip"
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
  
  s3_bucket = "YOUR_BUCKET_NAME"
  s3_key    = "lambda-layer-chrome-aws/lambda-layer-chrome-aws-v143.0.0.arm64.zip"
}

layers = [
    aws_lambda_layer_version.chrome_aws_layer.arn
 ]
```

> **Note:** Make sure to select the layer ARN that matches your Lambda function's architecture.

**In the deployed lambda code**
You can just use a regular ES6 or CommonJS import statement for `@sparticuz/chromium`, and just use as
indicated.

## Usage Examples

### Basic Puppeteer Example

```js
const chromium = require('@sparticuz/chromium');
const puppeteer = require('puppeteer-core');

exports.handler = async (event) => {
  let browser = null;
  
  try {
    browser = await puppeteer.launch({
      args: chromium.args,
      defaultViewport: chromium.defaultViewport,
      executablePath: await chromium.executablePath(),
      headless: chromium.headless,
    });

    const page = await browser.newPage();
    await page.goto('https://example.com');
    
    const title = await page.title();
    const screenshot = await page.screenshot({ encoding: 'base64' });
    
    return {
      statusCode: 200,
      body: JSON.stringify({
        title,
        screenshot
      })
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  } finally {
    if (browser !== null) {
      await browser.close();
    }
  }
};
```

### PDF Generation Example

```js
const chromium = require('@sparticuz/chromium');
const puppeteer = require('puppeteer-core');

exports.handler = async (event) => {
  let browser = null;
  
  try {
    browser = await puppeteer.launch({
      args: chromium.args,
      defaultViewport: chromium.defaultViewport,
      executablePath: await chromium.executablePath(),
      headless: chromium.headless,
    });

    const page = await browser.newPage();
    await page.goto(event.url || 'https://example.com', {
      waitUntil: 'networkidle0'
    });
    
    const pdf = await page.pdf({
      format: 'A4',
      printBackground: true
    });
    
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/pdf',
        'Content-Disposition': 'attachment; filename=output.pdf'
      },
      body: pdf.toString('base64'),
      isBase64Encoded: true
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  } finally {
    if (browser !== null) {
      await browser.close();
    }
  }
};
```

### Web Scraping Example

```js
const chromium = require('@sparticuz/chromium');
const puppeteer = require('puppeteer-core');

exports.handler = async (event) => {
  let browser = null;
  
  try {
    browser = await puppeteer.launch({
      args: chromium.args,
      defaultViewport: chromium.defaultViewport,
      executablePath: await chromium.executablePath(),
      headless: chromium.headless,
    });

    const page = await browser.newPage();
    await page.goto(event.url, { waitUntil: 'networkidle0' });
    
    // Example: Extract all links
    const links = await page.evaluate(() => {
      return Array.from(document.querySelectorAll('a')).map(a => ({
        text: a.textContent.trim(),
        href: a.href
      }));
    });
    
    return {
      statusCode: 200,
      body: JSON.stringify({ links })
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  } finally {
    if (browser !== null) {
      await browser.close();
    }
  }
};
```

## Troubleshooting

### Common Issues

#### Memory Errors
**Problem:** Lambda function runs out of memory
**Solution:** Increase Lambda memory to at least 1024 MB. Chrome requires significant memory to run.

#### Timeout Errors
**Problem:** Lambda function times out
**Solution:**
- Increase Lambda timeout (recommended: 30-60 seconds for simple tasks)
- Use `waitUntil: 'networkidle0'` or `'domcontentloaded'` instead of `'load'` for faster page loads
- Consider implementing page load timeouts: `await page.goto(url, { timeout: 30000 })`

#### "Could not find Chrome" Error
**Problem:** Chrome executable not found
**Solution:**
- Verify the layer is attached to your Lambda function
- Ensure `@sparticuz/chromium` is marked as an external module in your bundler configuration
- Check that the layer architecture (x64/arm64) matches your Lambda function architecture

#### Font Rendering Issues
**Problem:** Missing fonts or incorrect text rendering
**Solution:** Chrome includes basic fonts, but for specific fonts, you may need to:
- Include custom fonts in your deployment package
- Set the `FONTCONFIG_PATH` environment variable
- Use web-safe fonts when possible

#### Slow Cold Starts
**Problem:** First invocation takes too long
**Solution:**
- Use Lambda provisioned concurrency for critical functions
- Implement connection pooling if making multiple requests
- Consider using Lambda SnapStart (for Java runtimes)

### Debugging Tips

1. **Enable verbose logging:**
```js
const browser = await puppeteer.launch({
  args: [...chromium.args, '--enable-logging', '--v=1'],
  executablePath: await chromium.executablePath(),
  headless: chromium.headless,
  dumpio: true // Pipe browser process stdout/stderr to Lambda logs
});
```

2. **Check Chrome version:**
```js
const browser = await puppeteer.launch({...});
const version = await browser.version();
console.log('Chrome version:', version);
```

3. **Test locally with Docker:**
```bash
docker run --rm -v "$PWD":/var/task:ro,delegated \
  public.ecr.aws/lambda/nodejs:20 \
  handler.handler
```

### Performance Optimization

- **Reuse browser instances** when possible (be careful with Lambda concurrency)
- **Disable unnecessary features:**
```js
args: [
  ...chromium.args,
  '--disable-dev-shm-usage',
  '--disable-gpu',
  '--single-process',
  '--no-zygote',
  '--disable-setuid-sandbox'
]
```
- **Use page pooling** for multiple operations in a single invocation
- **Minimize page.goto() calls** - navigate only when necessary

## License

MIT © [Shelf](https://shelf.io), © 2025 IJoT BV

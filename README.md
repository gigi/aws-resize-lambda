# AWS Lambda function to create multiple thumbnails

This code creates multiple thumbnails with [AWS lambda](https://aws.amazon.com/ru/lambda/) feature
using node.js (v0.10.42) and [Imagemagick](https://imagemagick.org/) library.

This example doesn't use any external npm packages (even imagemagicwrapper) as recommended here [http://docs.aws.amazon.com/lambda/latest/dg/with-s3-example-deployment-pkg.html](http://docs.aws.amazon.com/lambda/latest/dg/with-s3-example-deployment-pkg.html)

## Introduction

When you or your app uploads image on S3 bucket, this code downloads image, makes thumbnails with command like this:
```
convert FILENAME \( +clone -quality 80 -interlace Plane -resize 264x177\> -write jpeg:- +delete \)
   \( +clone -quality 80 -interlace Plane -resize 576x387\> -write jpeg:- +delete \)
   \( +clone -quality 80 -interlace Plane -resize 1242x834\> -write jpeg:- +delete \)
   \( +clone -quality 80 -interlace Plane -resize 5000x5000\> -write jpeg:- +delete \)
   null:
```
then grabs binary stdout buffer, splits buffer into images, puts images back to S3, notifies you backend via [Amazon SNS](https://aws.amazon.com/ru/sns/) and optionaly removes the original file

## Installation

1. Create lambda function [https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions)
2. Enter any name you want
3. Select runtime Node.js 0.10 (4.3 needs some refactor)
4. Copy code from resizer.js to inline textarea or upload package (see [https://docs.aws.amazon.com/console/lambda/nodejsdeploy](https://docs.aws.amazon.com/console/lambda/nodejsdeploy))
5. Set handler value ``exports.resizer``
6. Select Role for lambda execution (or create new Basic execution role)
7. If you want to upload or delete files with this lambda add policy to execution role:
```
{
    "Action": [
        "s3:DeleteObject",
        "s3:PutObject",
        "s3:PutObjectAcl"
    ],
    "Effect": "Allow",
    "Resource": "arn:aws:s3:::*"
}
```
8. I recommend to change memory limit to 512 MB and timeout to 15 seconds (depends on input/output image sizes)
9. Add [event handler](http://docs.aws.amazon.com/lambda/latest/dg/with-s3.html) for your bucket for necessary input (e.g. input/images) and/or extensions (*.jpg, *.png)

## Usage

You can modify code for you needs.
Adjust thumbnail sizes, prefixes, SNS ARN topics (getTopicArn function) if you use SNS service


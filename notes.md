# AWS for Front-end engineers

[Course link](https://frontendmasters.com/courses/aws-v2)

Steve Kinney - Frontendmasters

## What is being covered

Getting a single-page app:

- Hosted on AWS
- Distributed globally
- Secured with SSL
- Automatically deployed with CI/CD
- Dynamically responding to requests

## What is not covered

- **Servers**. Course focused on the high-performance distribution of client-side JS application.
- **Serverless**

## AWS Free Tier Overview

- S3: 5GB of storage.
- Cloudfront: 1TB of data transfer.
- Lambda: 1 million free requests per month.
- Cognito: 50,000 monthly active users.
- AppSync: 250,000 GraphQL data modifications per month.
- Amazon Location Service: 3 months of location data.

---

## Account Setup & Billing

Set up an account in AWS using the Basic Use - Free tier.

You can later sign is as **Root users** and **IAM** users. As a general rule, don't use the **Root user** other than creating an Admin user. The **root user** has unrestricted access to everything, so it can be dangerous to use it without knowing what you are doing.

First, set up alarms for your free tier usage:

1. Sign in as **root user**.
2. Go to your Account and click on _Billing preferences_.
3. Click on **Edit** on **Alert preferences** and check the box for _Receive AWS Free Tier Usage Alerts_ and add the email.

Then, create a budget:

1. Go to your Account and click on _Budgets_
2. Create a new budget and select the _Use a template_ option as well as _Zero spend budget_.
3. Give the budget a name and add the recipients for the alerts if the budget goes over the free tier limits.
4. Click on the newly created budget to edit it.
5. Select the budgetting period and the budgeted amount.
6. Configure alerts for your budget if you want.

## Securing root account with IAM

You need to secure your root user from any undesired access and follow some general tips:

1. Go to the IAM service.
2. Add MFA for the root user
3. Create an Alias for the account, so that the sign-in url for IAM users is user-friendly. For example, you can put your company name or project.
4. Add a user named for an administrator. Enable both **Access key** and **Password** credential types for the user.
5. Set permissions to the user. Attack the _AdministratorAccess_ policy to the admin user.
6. If you created the user with **Access key** credential type, you will see two important pieces of data: **Access key ID** (think of it as username) and **secret access key** (password). Store it somewhere securely now! You will never see it again.
7. Add MFA for the admin user as well.

---

## S3

**Simple Storage Service**

At a high level, S3 has buckets. You can put objects (read: files) in your buckets. You can read from your buckets. You can host web pages out of your buckets.

It is infinitely scalable and has high durability (99.99999%).

Features:

- Lifecycle management
- Versioning
- Encryption
- Security

S3 is effectively a _key/value_ store. Putting new objects is immediate. Updating and removing them is eventually consistent, which means there is an insignificant probability that a user might get an older version.

Uploading to S3 is free. You get charged for storage and for requests.

Bucket names must be unique and it needs to be the same as the URL. So you should make sure to name it like your domain.

Tip: If you are setting the domain with Route 53, do it before creating a bucket to make sure the domain is available and give it time to be fully registered.

## Setup a Domain with S3 & Route 53

Route 53 is basically the DNS service. You can register domain names here, host in different servers and re-route traffic from slower to faster servers.

It is recommended to use Route 53 for your domain if you are going to use AWS as its integration with the other services are incredibly simple.

Route 53 is the service where you can buy domain names and check for availability. After you register one, you need to give it some time to fully process it.

Back to S3, this service is global, which means, it is not dependent on regions like other services. It is simple to create a bucket:

1. Go to S3.
2. Click on _Create a bucket_
3. Give the bucket a name and select a region (use us-east-1 as default).
4. Access control lists (ACLs) should be disabled.
5. Depending on the purpose of the bucket, you might block all public access on it or not.
6. You can turn versioning on or off. Having it on generates costs.

## S3 Policies

Users and resources can have policies to define what they are allowed to do. In the example of the S3 bucket, you would probably want to have a policy so anyone can read from the bucket, but almost no one can write or delete objects from the bucket.

Adding a policy to a bucket can be done by providing it a JSON or YAML file with its policies:

```json
{
  "Id": "Policy123456789",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Name/identifier for the policy",
      "Principal": {}, // who can do the thing
      "Effect": "Allow", // Or Deny
      "Action": [], // what should they able to do
      "Resource": [] // on what should they be able to do it
    }
  ]
}
```

It's important to notice that the `Resource` property should have the resources' ARN (Amazon Resource Name), which is Amazon's way of identifying a particular resource.

Once an object has been uploaded to the bucket, you can access it on the browser by copying its URL, if the bucket contents are publicly available.

### Hosting a static website with S3

Go to the bucket's properties and scroll down until you see an option that reads **Static website hosting**.

1. Enable static website hosting.
2. Leave the selected option for hosting type: _Host a static website_.
3. Provide the index document (home or default page of the website).
4. Provide an error document (optional).

After all of these steps, you should be able to see the URL for you static website. For example: http://randomcoursedomain.click.s3-website-us-east-1.amazonaws.com

### AWS CLI

AWS CLI is a tool that helps you manage AWS services from a CLI. You need to download it for your local OS [here](https://aws.amazon.com/es/cli/).

After download, open up your terminal and type: `aws configure`. This should ask you for your **Access Key ID** and **Secret Access Key**.

If you have more than one AWS account, you can create additional profiles in `.aws/credentials`. You'll have your default profile but you can create additional ones as well, like such:

```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMxIbNOTREAL

[sideproject]
aws_access_key_id=AKIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbTOTALLYFAKE
```

## Deploy App with the AWS CLI

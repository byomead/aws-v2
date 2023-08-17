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

AWS CLI allows you to use AWS services in a more programmatic way. You can, for instance, list the contents of an S3 bucket: `aws s3 ls s3://<bucket name>`. Or, more importantly, upload files and directories to it using: `aws s3 cp <file or directory> <bucket-name>`. One application for this is to upload your built project and deploy it using S3.

For example: `aws s3 cp build s3://randomcoursedomain.click --recursive`. The `recursive` flag tells AWS CLI to check for inner directories and upload them.

You can then go to your bucket URL and see your built project on the web! Keep in mind, your objects are stored in the default region (usually us-east-1), so anyone far from North Virginia is going to take progressively longer to load.

## Configure Route 53 and Routing

In order for Route 53 and your domain to point to your S3 bucket static endpoint, you need to follow these simple steps:

1. Go to Route 53 and then select **Hosted zones**.
2. Select the hosted zone for your domain.
3. Click on **View details**. You should see all of the records inside that hosted zone.
4. Click on **Create record**. Once the page loads, look for a little toggle labeled _Alias_ and toggle it on.
5. On the **Choose endpoint** dropdown, select _Alias to S3 website endpoint._
6. Select the region where your bucket lives.
7. Look for your bucket and select it.
8. Click on **Create record**. It will take you to the page with the records of the hosted zone where you should now see the new record.

If you go to the domain, don't get discouraged if it doesn't immediately show your page as DNS takes some seconds to be propagated across several servers.

### Configure SSL

In order for your new page to be secure, you need a SSL certificate (the thing that makes your site become HTTPS and not HTTP). Request a certificate by going to **AWS Certificate Manager** service.

Quick note: you cannot request a certificate for Amazon-owned domain names such as those ending in amazonaws.com.

Once you request a certificate, follow these steps:

1. Provide the full (qualified) domain names. It is recommended that you also add a "long" version of the domain name such as: `www.yourdomain.com`.
2. Select DNS validation.
3. Leave the selected key algorithm (RSA 2048).
4. Request the certificate.
5. Refresh the certificates page and see yours in "Pending validation". Don't be confused, you actually need to step in to validate, it is not an automatic process.
6. Click on your requested certificate.
7. Go to the **Domains** section and click on **Create records in Route 53** button.
8. You will see the certificate records ready to be created, just click on **Create records** button.

While your certificate is pending, the DNS should have already been propagated by now.

Now, time to fix another problem: what happens when a user enters an unknown path in your domain and how to handle it. One simple way is to simply provide an error page in your static website configuration for your S3 bucket OR... send them to the index.html file even on error the same way.

## www Redirect

AWS supports the ability to have a bucket redirect to another bucket and that's what you need to do to make sure going to the www version of your domain also serves the same contents as the bucket for your regular domain. In other words: `www.domain.com` bucket will point to the regular `domain.com` bucket.

### Configure new bucket

Simply create another bucket with the www version of your domain (you don't need to turn off _Block all public access_), and go to the **Properties** to enable _Static website hosting_. But in this case, in **Host a static website**, select _Redirect requests for an object_ and provide the **original** domain name (DNS should already be pointing to your original bucket at this point).

### Configure new record in Route 53

If you want to see `www.domain.com` to point to the same as `domain.com` you also need to create a new record for your domain hosted zone in Route 53. The same way you created a record alias pointing to the original bucket, create another one pointing to this www bucket. As a tip, name this record: `www` so that you know this one points to the `www` bucket.

## Configure Cloudfront Distribution

Next issue to address is the fact that the website is hosted exclusively in whatever region you created your S3 bucket. The further away a user is from this region, the longer it would take them to load. To solve this, use **CloudFront**.

1. Create a new CloudFront distribution
2. Point it to the static website on S3
3. Add the domain names
4. Set up gzipping for the assets
5. Set a default root object

### Creating a distribution

First, set up the origin domain for the distribution. Copy and paste the URL that AWS provided for your bucket in the static hosting section.

Second, you can leave most of the pre-selected options as they are, except for the viewer protocol policy. Select _Redirect HTTP to HTTPS_.

Third, select the allowed HTTP methods. Change this accordingly to what your project does.

Fourth, in the **Web Application Firewall**, select _Do not enable security protections_.

Fifth, select the option to use all edge locations.

Sixth, add the alternate domain names (CNAME). These are your regular domains and your www domain.

Seventh, provide your (hopefully ready) ACM certificate. If you don't have one, you'll have to create one before moving on. If you don't see it even after you created one, make sure that the certificate is in us-east-1. While CloudFront is globally distributed, it "lives" in us-east-1.

Finally, create the distribution.

**Note:** It's also a good idea to add the index file name so that the distribution which is the default root object.

### Pointing Route 53 to the CloudFront distribution

Go to Route 53 and look for your hosted zone where your domain records live. Edit the record that points to the S3 bucket and change the _Route traffic to_ section to **Alias to CloudFront distribution**.

Select your newly created distribution and save.

Do the same steps for the www domain record and let it do its magic.

## CloudFront and S3

CloudFront will cache the assets in S3, which reduces the number of read requests to these object, which in turn, reduces the costs.

CloudFront also distributes your assets around several Edge locations around the world, making it faster and more accessible to people almost anywhere in the world, as they just need to hit the edge location closest to them. It also has some regional edge caches so that in case an Edge location doesn't have the website cached, it goes to the regional edge cache, which is still close, and looks for it there.

Utimately, if it doesn't find it anywhere, it goes to the source.

By default, CloudFront ignores request headers.

## OAI (Origin Access Identity)

**NOTE.** THIS SECTION IS OUTDATED. OAI IS NOW LEGACY AND IT IS RECOMMENDED TO USE OAC (ORIGIN ACCESS CONTROL) INSTEAD.

This is a new feature that lets you secure your S3 bucket completely while at the same time allowing CloudFront to be the only one (service/user/whatever) to extract objects from there. This ensures there will be the fewest requests to the bucket as possible.

CloudFront can mask pretty much anything: S3 buckets, EC2, API gateways, your own data centers, etc. CloudFront acts as the middle man that fetches from the source and performs the caching.

To enable this, go to the origin settings of the CloudFront distribution and change the origin domain to the bucket endpoint instead of the website endpoint as you previously had.

Select _OAI_ under the **Origin access** section and create a new OAI. Next, allow AWS to automatically update the bucket's policies and save.

If it fails, check the bucket's policy and change it yourself. Make sure to use the correct OAI id, not the distribution id.

## Securing the S3 Bucket

Finally, you can turn off the bucket's static website hosting and lock it down (block all public access). CloudFront is now the only one with access to get what's inside the bucket and it will handle all of the requests to the page.

## CloudFront Routing

You can provide a custom error page to your CloudFront distribution if the user ever does something unexpected, like trying to reach a page that doesn't exist.

Go to your distribution, click on the _Error pages_ tab and create a custom error response.

Select the error code (usually 404 when user hits an inexistent page), and you can decide if you want to customize error response or not.

You can also create a custom error response for 403 (Unauthorized) to instead show a 404 error. Since we already established that 404 errors will show the index page instead, this then triggers the client side routing. If the route you specified exists in thw website, it will first try to go to the S3 bucket and get an Access Denied, which is a 403 and then be redirected as a 404 and then be redirected to the index page again, which will then use client side routing.

## Create Custom Cache Policies

CloudFront caches everything so that the users don't wait a lot of time for the page to load. But when you deploy a new version, you want that new version to be cached so users can reach it. There are essentially two ways to do it:

1. Update the dafault behavior or create a new one for your index page and edit the time the page remains cached. By default, CloudFront caches a new version every 24 hours, but you can decrease it to minutes.
2. Create an invalidation and set it to all paths using a wildcard (`/*`). This can be used via a CI pipeline to invalidate all cache when there is a new deploy.

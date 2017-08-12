---
layout: post
title: Receiving .io certificate confirmation emails with Amazon SES
tags: [aws]
---

So you just registered one of those super trendy .io domains that all the cool kids have these days. You're a boss because you paid $40 instead of $13 for the .com you wanted but was already taken. Now its time to set up your static site in S3 on the cheap, but that is not enough for you. You need more. You must conquer the world. So you distribute your site with CloudFront. But still... you need more. You must mark your territory with a seal of approval, an SSL certificate.

You set up a certificate with Amazon Certificate Manager and wait for your confirmation email proving you own the domain. And you wait. And wait a little longer. And then you realize, that email is never going to come. But why? Well that's because the Registrant, Administrative, and Technical contact you set up when you registered your domain are not returned by a WHOIS lookup for .io domains.

Basically, Amazon does not know how to reach you to validate you own the domain (even though you likely registered it through Route53). Fortunately, amazon also sends a verification email to the following addresses:

* administrator@your_domain_name
* hostmaster@your_domain_name
* postmaster@your_domain_name
* webmaster@your_domain_name
* admin@your_domain_name

No problemo! We are going to take advantage of that! This is where Amazon SES comes in handy and I'm going to walk you through it.

### Verify Domain
First you are going to want to verify your ownership of the domain. Go to Amazon SES > Domains > Verify Domain > Provide .io domain > And you should see something like this:

![](/img/AWS/verify-new-domain.png)

You need to add a TXT record and an MX record. Click the "Use Route 53" button to take care of this for you. Ignore anything about outgoing or sending email.

* A TXT record is commonly used in verifying domain ownership.
* An MX record is specifies the mail server responsible to accept mail to your domain.

You will see a pending verification screen like this for a few minutes before your domain becomes verified.

![](/img/AWS/domain-registration-pending-verification.png)

### Setup an S3 bucket
If you've made it this far, you likely know how to setup a bucket so I'm not going to take you through the step by step. But its important to know you do need to add a bucket policy that will allow SES to write to it. Personally, I use an 'emails' folder in my logging bucket for my main site. Your policy would look something like this:


```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "GiveSESPermissionToWriteEmail",
            "Effect": "Allow",
            "Principal": {
                "Service": "ses.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::my-awesome-bucket/emails/*",
            "Condition": {
                "StringEquals": {
                    "aws:Referer": "###myAccountNumber###"
                }
            }
        }
    ]
}
```

### Set up SES for receiving email
Again from the SES console page go to Amazon SES > Email Receiving > Rule Sets
> Create Rule Set > Provide a name > Create a Rule Set.

Once you've created a rule set, you still need to set rules and activate it.
Select your rule from the inactive rule sets and edit it:

![](/img/AWS/ses-ruleset.png)

Go ahead and create a rule for your ruleset. I only added one rule.
In your rule, you'll need to:

* Enable the rule.
* Add your S3 bucket.
* Specify the recipient. Amazon will send email to the 5 addresses, but
you really only need one. Optional.
* S3 prefix. Optional.

### Generate certificate
Next, you'll need to jump on over to Certificate Manager and request a new certificate. Fill out your domain information like so. You can add optional additional subdomains if desired. Submit your request.

![](/img/AWS/certificate-manager-add-domain-names.png)

### Certificate validation email
This is where the magic happens! Emails to our mystical 5 addresses should be sent curtesy of Amazon almost immediately. Jump to your S3 bucket and you should see some crazy files in there. Download one and open it with a text editor.  There will be a link inside that will looks something like:
https://us-east-1.certificates.amazon.com/approvals

Open up that link, click the approve button and are golden!

### Helpful Documentation:
[Validate Domain Ownership](http://docs.aws.amazon.com/acm/latest/userguide/gs-acm-validate.html)

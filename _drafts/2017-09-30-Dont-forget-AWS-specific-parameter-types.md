---
layout: post
title: Making a case for AWS-Specific parameter types
tags: [aws, Cloudformation, programming]
---

Recently I was working on a Cloudformation template that allowed a user to deploy a set of resources including an EC2 instance and conditionally pass an instance key parameter. This is so I can deploy instances with no keypair using the same template. I first checked out [cloudonaut's post](https://cloudonaut.io/optional-parameter-in-cloudformation) on the subject and then created a branch on my own project. Each time I would try to deploy my Cloudformation with an empty key, I would immediately receive `Parameter validation failed: parameter value for parameter name KeyName does not exist.`


What gives? It's an empty value! Why won't you work!? Now I'm checking over the whole template and pounding my head on the table! Then I realize the parameter type of my keyname.

```json
  "KeyName": {
    "Description": "Provides the name of the EC2 key pair",
    "Type": "AWS::EC2::KeyPair::KeyName"
  }
```

Notice the type is not "string". Another project contributor had set it to an ["aws-specific parameter" type](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html). This was honestly the first time I recall seeing a Cloudformation template using these. So what are these AWS-Specific parameter types? The official documentation says:

>  "for AWS-specific parameter types, template users must specify existing AWS values that are in their account."

My template was failing because Cloudformation was expecting me to pass a valid keypair in the account.

So it seems impossible to use an aws-specific parameter in this case. Back to good ole fashioned strings. But they can be extremely helpful in some other cases!

### An invalid string type parameter

[Here is a very simple Cloudformation template](\files\string-parameter-example.json) that creates an S3 bucket and a Route53 record set. The record set depends on the bucket. The HostedZoneId is passed as a string and I intentionally pass an invalid name. In this case, the S3 bucket needs to be created before Cloudformation provides feedback that the HostedZoneId is invalid.

```json
{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "An invalid hostedzone ID as string takes time to fail",
    "Parameters": {
      "InvalidHostedZone": {
          "Description": "TestResource",
          "Type": "String",
          "Default": "Z8OOPS8TYPO8X"
      }
    },
    "Resources": {
      "s3Bucket": {
        "Type": "AWS::S3::Bucket",
        "Properties": {
          "AccessControl": "Private"
        }
      },
      "route53RS": {
        "Type": "AWS::Route53::RecordSet",
        "DependsOn": "s3Bucket",
        "Properties": {
          "Name": "param-example.benforce.io",
          "Type": "A",
          "HostedZoneId" : { "Ref": "InvalidHostedZone" },
          "AliasTarget" : {
            "DNSName" : "benforce.io",
            "HostedZoneId" : { "Ref": "InvalidHostedZone" }
          }
        }
      }
    }
}
```

![](\img\AWS\Cloudformation\invalid-param-name1.png)

### An invalid aws-specific type parameter

Now [here is a second example Cloudformation template](\files\aws-specific-parameter-example.json) that uses an AWS-specifc parameter. The only line that has changed is `"Type": "String",` changed to `"Type": "AWS::Route53::HostedZone::Id",`. Parameter validation happens immediately and Cloudformation sends feedback of an invalid HostedZoneId within 1 second.

```json
{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "An invalid hostedzone ID as an AWS-specific parameter fails immediately.",
    "Parameters": {
      "InvalidHostedZone": {
          "Description": "TestResource",
          "Type": "AWS::Route53::HostedZone::Id",
          "Default": "Z8OOPS8TYPO8X"
      }
    },
    "Resources": {
      "s3Bucket": {
        "Type": "AWS::S3::Bucket",
        "Properties": {
          "AccessControl": "Private"
        }
      },
      "route53RS": {
        "Type": "AWS::Route53::RecordSet",
        "DependsOn": "s3Bucket",
        "Properties": {
          "Name": "param-example.benforce.io",
          "Type": "A",
          "HostedZoneId" : { "Ref": "InvalidHostedZone" },
          "AliasTarget" : {
            "DNSName" : "benforce.io",
            "HostedZoneId" : { "Ref": "InvalidHostedZone" }
          }
        }
      }
    }
}
```

![](\img\AWS\Cloudformation\invalid-param-name2.png)

### A little bonus!

Another reason to use AWS-specific parameters is because it bypasses the need to use the "AllowedValues" property of a parameter and input your resources manually. Here are my hostedzones demonstrated:

![](\img\AWS\Cloudformation\invalid-param-name3.png)

---
layout: page
title: AWS Wish List
---

Here is my prioritized list of enhancements I'd like to see over time. I'll try to document them as they are resolved, especially the lesser 
known breakthroughs.

1. **Multi-role Instance Profiles or fix CloudFormation**
    * This one seems like a no brainer and almost like it was the idea all along. At the moment, what is an Instance Profile? It is a role. So why have it at all? A second option would be to have CloudFormation detach policies that were not created by the stack. As an infrastructure guy, I'd love to have lambda attach AmazonEC2RoleforSSM policy to every instance profile, as well as a few more entitlements. But it becomes a problem in cloudformation when the role behind the instance profile is deleted. I'm sure our SOC would also like to make this easier as well as the pipeline owners and many other teams.
3. **Service Catalog integration cloudformation**
    * From my experience the learning curve for onboarding COTS products in a large enterprise is nearly insurmountable. There are just so many things to learn. Being able to provide standard patterns across dozens of accounts as close to the source as possible would be a major win with Service Catalog. It can be stored in version control, but even to checkout out of version control and deploy  through a pipeline is an amazing task in and of itself for 
4. **Target all Systems Manager managed instances**
    * Currently Systems Manager supports associating document targets to tags or instance IDs. I'd like to just assocation the AWS-GatherInventory document to all managed instances. Why wouldn't you want to do that? I had to write a custom resource to describe my managed instances and use that list to tag my ec2 instances just to create a managed instance tag just to do this.
5. **Resource data syncs in CloudFormation** - This is another area where Cloudformation custom resources came to the rescue.
6. **Powershell for Lambda** -  A guy can dream, can't he?




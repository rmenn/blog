---
title: "Terraformer Imports"
date: 2020-09-12T03:05:05+05:30
draft: true
description: "Importing Cloud Resources Using Terraformer"
---
 
I recently inherited an AWS account at my present employer which has a few VPC, managed Data Stores, a few stray instances & a couple of EKS cluters. 


The Problem : No IaC

The Solution : terraform imports

Now this isnt new, here the thing was there was a lot of things to get rid off ( looking at you launchwizard7 security group ), so i needed to bring a certain part of the AWS cloud under terraform while ignoring the others. That would actually provide a definitive guideline as for me to mark resources to get rid of.

I decided to use `terraform import` to pull things i need into the control of terraform. So here is something that you need to know, terraform imports things from the provider to the statefile, thats it, the rest is upto you. By that i mean, you need to have an empty resource like 

```
resource "aws_s3_bucket" "kindle-images" {
}
```

So that you can import the resource from the provider to terraform, which you would do like so

```
terraform import aws_s3_bucket.kindle-images amazon-pottery-images
```

where the syntax is `resource.desired-name actual-bucket-name`

After which you need to add to the all the resource definition to complete the resource. Thats just way too much work.  

Now since this isnt the first time i needed to do this, i started making notes on what needs to be kept and what needs to move. Now the last time i did this there was this very useful tool [terraforming][1], which i could use to import resources in bulk and even have it merge states. But unfortunetely the prokect seems to have run aground, There are missing resources, fixes which have not been merged and updates to terraform version 0.12 which have not been merged. 

I decided to split up the components, get the basic tier in, such as the VPC, route tables, subnets etc, which i had to use terraforming to get, then use terraform import to have a sane state. Once i got all the basic components within the purvue of terraform things started getting easier. There were issues since the latest version of terraform wasnt supported but most of those were dur to the tags. 

So terraforming pulled the data and tags were in the format

```
   tags {
      XXXX = YYYY
  }
```

which needed to be converted to 

```
   tags = {
      XXXX = YYYY
  }
```

which looks like terraform moves to a map structure, which was a minor change to do especially since one has `sed`

Next i had to get the EKS cluster into terraform, which looks like was initially brought up using `eksctl`, i ended up writing a module so that i could bring up self manages nodes for them. The problem was the clusters themselves, so i put on my ruby hat and wrote what i needed to import eks resources using terraforming. So thats where the story stops, since i believed i had the bare minimum i needed to manage the infra.

I was on a slack channel a couple of months later, where i had stopped moving resources to terraform, where someone mentioned handcrafting terraform statefile since they had to move resources, so i pointed them to [terraforming][1] since i had worked with it before, but was thrown back another tool [terraformer][2], now this peaked my interest and tried out the tool, where i got to knwo some of its usefulness and some of its caveats. 


Let me first tell you what i dont like about it

* New Generated Folder

	It creates a `generated/<provider>/<resource>/` folder and places all the files in there, the provider, the output, the <resource>.tf file & the terraform.tfstate

* Filters are per resource
	
	Filters are anyones game, i had to look at the code to figure out some of this


* Statefiles are separate

	As mentioned above, the statefile is in the generated directory and there is no merge state in the tooling, so if you wants to move each resource into its own state, you can stop here.

* Naming Conventions
	
	It would name things as some conversion of the `Name` tag with a prefix, to me it looked ridiculous. Imagine running `terraform state get tf----www2D--****` something, it wasnt for me.  

What ended up doing was to use the `terraforming plan` command to generate a plan file, edit it with what changes i need, naming convention, references to other resources etc was all made in the plan file post which you can import the plan file, which is an interesting feature

```
GIVE CODE EXAMPLE HERE
```

Post which i moved the file from generated and ran the good old `terraform import` to secure my infra in place.

*Filters*

Now to using the filters, this is something i found tricky to say the least, this field is accepted by the command line interface but how its used and interpreted is entirely upto the resource you are trying to fetch, this lead me down a path prior to reading up and figuring out what i needed to do. 

I am going to use an example of 2 `aws_instance` and 2 `aws_eip` which i needed to import as those instances have a critical role, so anyone would want it in source control and under terraform.

```
GIVE CODE EXAMPLE
```

Here im looking to import 2 resources `aws_instance` & `aws_eip`, these can be specified in a `,` separated value in the `--resource` field. Now the filter, since i need to apply different filters to each of `aws_instance` & `aws_eip`, you can make use of `Type` in filter so that you can direct terraformer to use certain filters for certain resources, what is followed by specifying the `Name` of the filter as `id` and providing the `Value` of it as whats required. 


[1]:ihttps://github.com/dtan4/terraforming
[2]:https://github.com/GoogleCloudPlatform/terraformer

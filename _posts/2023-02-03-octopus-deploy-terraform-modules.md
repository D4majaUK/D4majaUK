---
layout: post
title: Octopus Deploy Terraform Modules
categories:
  - octopus-deploy-post
---

# Problem

While Octopus Deploy (OD) handles a flat inline source Terraform template without issue (unless you make mistakes),
there comes the problem when you want to implement more complex solutions. In particular modules.
In the usual Terraform scenario, you'd develop a folder structure that allows you to separate out the templates.
However, OD doesn't make this easy within the inline edit box. But all is not lost...
As you can see below, OD allows you to use packages. 

![Octopus Deploy source template](/Portfolio/images/ODtemplateOptions.jpg)

When I first saw this, I thought, Oh Great! Now I have to link it to a source code repository (which along the lines of IaC, 
that would kind of make sense and will be the goal later). As this was just getting to grips with using OD with Terraform,
it is not something I also want to have to manage, until at least I get it working as expected. Anyway, let's get onto the solution.
It is actually a straight forward process, we just need to upload a package and then it can be referenced within the process.


# Solution

1. Create the folder structure for Terraform.
2. Build out the Terraform implementation within that structure.
3. Package it up.
4. Upload to the in-built package store.
5. Reference within the template.
6. Deploy the solution.

### 1. Create the folder structure for Terraform

![Terraform folder structure](/Portfolio/images/ODtreeStructure.jpg)

Here we can create the structure that we want. In this example, under the modules foilder I have created a VM subfolder.
That will then house the VM module files that will be pulled into the main part of the template.
The purpose of this is to be able to create multiple VMs, by just passing in the VM name into the module.


### 2. Build out the Terraform implementation within that structure

To recall the the flat structure, [click here to see the post](https://d4majauk.github.io/Portfolio/octopus-deploy-post/2022/12/21/octopus-deploy-terraform-build-agent.html).
From that I have basically copied everything in that is repeatable and specific for the VM. It's out of scope for this post.


### 3. Package it up

OD is great in the way that it allows a few extensions to pass for packages that you can upload.
You can go down the Nuget reoute, althought I found this was more for code examples.
But what is great, is that it also allows .ZIP format. Cowabunga!!! I'll take some of that.
So, once you are happy with the structure and contents, all you have to do is send it to a compressed file (Zip it up as they say).
OD has a small requirement, and that is to make sure the filename has some kind of versioning. **\<Filename\>.Major.Minor.Patch.zip** 


### 4. Upload to the in-built package store

OD then has a simple method of uploading files within it's portal, as you can see below. And it really is as simple as that.

![Terraform folder structure](/Portfolio/images/ODtreeStructure.jpg)

This puts it into the OD built-in package repo.

**NB: In this way, you will need to manage the versions and how you use them**


### 5. Reference within the template

As seen at the beginning of this post, you then include the package name (specific version of your choice) in the relevant box.


### 6. Deploy the solution

That's it! All you have to do then is run it and you have a full end-to-end solution.


# Round Up

So there you have it, a whole breakdown of using Octopus Deploy to deliver your Infrastructure as Code solution.
This is more of an experimental or to accompany Runbooks. A complete infrastructure capability would be better use a source code repository.

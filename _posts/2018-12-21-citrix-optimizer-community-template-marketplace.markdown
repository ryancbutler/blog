---
layout: single
title: Citrix Optimizer Community Template Marketplace
joomla_id: 35
joomla_url: citrix-optimizer-community-template-marketplace
date: 2018-12-21 22:37:02.000000000 +00:00
author: Ryan Butler
excerpt: "Recently, Martin Zugec from Citrix released version 2.0 of the popular Citrix Optimizer tool and one of the cool new features added is the ability to add custom template marketplaces."
categories: 
  - Citrix
permalink: /citrix/35-citrix-optimizer-community-template-marketplace
toc: true
toc_sticky: true
---


Recently, [Martin Zugec](https://twitter.com/MartinZugec) from Citrix released version 2.0 of the popular [Citrix Optimizer tool](https://support.citrix.com/article/CTX224676) and one of the cool new features added is the ability to add custom template marketplaces. Once added a marketplace can allow users to choose then download and upgrade specific templates all from the Optimizer GUI. Out of the box, Optimizer includes a marketplace for all the Citrix maintained templates as you can see below

![base.png](/assets/images/content/marketplace/base.png)

After playing with the new version I thought it would be nice if there was a central location for the community to be able to submit, view and share their own custom templates. I ended up creating a home for a marketplace on Github and this post goes over the introduction of the marketplace and how I hope the community can utilize it.

## How to use the marketplace ##

Citrix Optimizer is designed to use [XML](https://ctxsym.citrix.com/supportabilitytools/citrixoptimizer/citrixmarketplace.xml) for the marketplace data, including the URL location of a template and any metadata to display to the user such as author and version.  This makes it easy enough to simply point Optimizer to a custom XML URL and Optimizer will be able to render the information and allow templates to be downloaded or updated.  Let's go over the steps to add the community marketplace to Optimizer.

![add.png](/assets/images/content/marketplace/add.png)
  
  -  From Citrix Optimizer select **Template Marketplace** from the left
  -  Select **Add New Marketplace**
  -  Add the URL:
        ```liquid
        https://github.com/ryancbutler/Citrix_Optimizer_Community_Template_Marketplace/releases/latest/download/communitymarketplace.xml
        ```
  -  and select **Done**
        ![add.png](/assets/images/content/marketplace/add.png)
  -  Now on the left, you should see **Citrix Community Marketplace**
          
        ![market.png](/assets/images/content/marketplace/market.png)
  -  Templates available to download will appear on the right
  -  As the marketplace grows this will allow users to download and update existing templates right from Optimizer

## Submit Templates ##

This marketplace is worthless without templates.  If you have created custom templates I want them!  This section will go over how to submit your templates to the marketplace so they can be shared. 

![cm2](/assets/images/content/marketplace/Cm2.jpg){:height="75%" width="75%"}

### Template Format Rules ###

Before submitting there are a couple of rules that must be followed or else your submission\pull request will fail.

 -  Only templates created with Optimizer 2.0 will be allowed. (1.x will need to be converted)
 -  Only unique display names and ids will be allowed
 -  Template author name must match the directory name of the template
    ![rule](/assets/images/content/marketplace/rule.png)

### Submit via GIT ###

This is the preferred method. If you haven't used GIT before this is a perfect situation to learn.  Don't get discouraged! 
 -  First, you'll need a [Github account](https://github.com/join)
 -  Go to the Github repo at https://github.com/ryancbutler/Citrix_Optimizer_Community_Template_Marketplace and **fork** to your own repo 
    ![fork.png](/assets/images/content/marketplace/fork.png)
     -  If you aren't familiar with this process I can't stress the benefits of at least learning the basics of GIT can be. There are plenty of [guides](https://help.github.com/articles/fork-a-repo/) out there to help. Keep going!
 -  Within your fork go to the **templates** directory and create a new folder named the same as the author of the templates.
 -  Copy your template(s) to the newly created folder
 -  **(Optional but highly recommended)** Create a [readme](https://help.github.com/articles/basic-writing-and-formatting-syntax/) within your directory explaining your templates along with any other information you would like.  Feel free to include your contact info, twitter or whatever.
 -  An example of the layout can be found in the **templates\Ryan Butler** directory
    ![example](/assets/images/content/marketplace/example.png)
 -  Once you're ready to submit to the marketplace you'll want to submit a [pull request]("https://help.github.com/articles/creating-a-pull-request/").
 -  If all the tests pass I'll be able to review and approve the PR making it part of the marketplace!


**Note:** There is no need to edit the **communitymarketplace.xml** file.  This will automatically be re-generated for each submission.
{: .notice--warning}

### OTHER ###

If you aren't comfortable with the GIT process feel free to reach out to me on [Twitter](https://twitter.com/Ryan_C_Butler) or wherever else and I'll be happy to add them or help you get them submitted. 

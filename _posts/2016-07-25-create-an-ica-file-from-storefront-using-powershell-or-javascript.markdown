---
layout: single
title: Create an ICA File from Storefront using PowerShell or JavaScript
joomla_id: 21
joomla_url: create-an-ica-file-from-storefront-using-powershell-or-javascript
date: 2016-07-25 01:00:35.000000000 +00:00
author: Ryan Butler
permalink: /citrix/21-create-an-ica-file-from-storefront-using-powershell-or-javascript
excerpt: "Stand alone ICA files used to allow organizations a multitude of access options, such as single click web shortcuts, login scripts or simple desktop shortcuts for XenApp access as recent as 6.5. When XenApp moved to the FMA architecture the good ol' days of the stand alone ICA files were gone  Administrators can attempt to come close to the same behavior by utilizing Receiver SSO with shortcut publishing or web shortcuts created from Storefront. While the ease of access is still \"kind of\" there, it's not nearly as easy and convenient as running a simple ICA file."
categories:
  - Citrix
  - Storefront
tags:
  - Storefront
  - ICA
  - PowerShell
  - Javascript
redirect_from:
  - /citrix/21-create-an-ica-file-from-storefront-using-powershell-or-javascript
---
# Good ol' ICA File

**Update 7-26-16:** I created an additional PowerShell script that can utilize explict authentication vs anonymous only.  Available on [Github](https://github.com/ryancbutler/StorefrontICACreator).
{: .notice--info}

* * *
Stand alone ICA files used to allow organizations a multitude of access options, such as single click web shortcuts, login scripts or simple desktop shortcuts for XenApp access as recent as 6.5\. When XenApp moved to the FMA architecture the good ol' days of the stand alone ICA files were gone  Administrators can attempt to come close to the same behavior by utilizing Receiver SSO with shortcut publishing or web shortcuts created from Storefront.  While the ease of access is still "kind of" there, it's not nearly as easy and convenient as running a simple ICA file.  This situation adds complexity for both administrators and users by additional configuration challenges for administrators and additional steps for users to access the published resource.  To complicate things even further for administrators, add in some Windows XP clients with an older version of Internet Explorer 8(see notification below when launching Storefont web shortcut) or stand alone Citrix Receiver Web client without SSO or pnagent functionality and it's a difficult scenario to solve.  This post covers scripts I created for both JavaScript and PowerShell that can generate ICA files to be launched a variety of different ways.

![ica](/assets/images/content/icascript/ica.png)

TL;DR: Click here to go to [GitHub](https://github.com/ryancbutler/StorefrontICACreator) to download.
{: .notice--info}

##  Limited Client Options

I was recently placed in a predicament where I needed XenApp application access that was currently done by an ICA file, but now needed to access a new XenDesktop 7.6 environment. The machines were running Windows XP, IE 8 installed and Citrix Receiver 3.4 web installed.  Eventually the machines will be upgraded with Windows 7 and a recent version of Citrix Receiver where many more options exist but I needed something immediate.  Researching solutions I configured both an anonymous access delivery group in XenDesktop, an unauthenticated store in Storefront and grabbed a "Web Shortcut" from Storefront.

![sfweb](/assets/images/content/icascript/sfweb.png)

And when I attempted to launch the link using IE 8 it prompted me with the notification below.  While the application would finally launch if I selected no to closing the tab, I didn't feel this was a very elegant solution and would confuse users.  I wanted to see if there were any other options while the machines got upgraded.

![sfissue](/assets/images/content/icascript/sfissue.png)

## Can I generate an ICA file?

To see if it was even possible to generate an ICA file while not accessing Storefront or Receiver directly I did some searching and found this [blog](https://sysadminasaservice.wordpress.com/2015/08/30/login-to-storefront-with-curl/) as a great starting point.  I ended up with a PowerShell script that can generate and launch an ICA file right from PowerShell  The script only works with version 3 of PowerShell which didn't help the Windows XP situation, but proved it was possible to create an ICA file via the API!  The script uses an unauthenticated store from Storefront, an anonymous XepApp application and then launches the generated ICA file.

![sfica](/assets/images/content/icascript/sfica.png)

## What about XP?

To work with Windows XP, PowerShell wasn't an option, so I created a JavaScript script that completes a similar process in creating and then downloading the ICA file.  The Javascript function can be called for any unauthenticated store and anonymous application.  IE 8 doesn't include a JSON parser so I also included [JSON2](https://github.com/douglascrockford/JSON-js) which works great!  I ended up placing the files right in the unauthenticated store root IIS file directory to allow user access.

 ![sfjs](/assets/images/content/icascript/sfjs.png)

The function is called if a user 'clicks' a button in the test.html but can be called in any fashion.

```html
<button onclick="starticaurl('[https://storefront.mydomain.local/Citrix/unauthWeb/'](https://storefront.mydomain.local/Citrix/unauthWeb/'), 'Notepad++')">Launch App</button>  
```

## Script Download

Both the PowerShell and JavaScript files can be located on GitHub.  Please feel free to share any ideas or comments!

Requirements:

* For PowerShell must have v3 installed
* [Unauthenticated StoreFront Store created](http://docs.citrix.com/en-us/storefront/3/configure-and-manage-stores/sf-create-store-unauthenticated.html)
* [Anonymous Delivery Group](https://www.citrix.com/blogs/2014/04/21/part-5-anonymous-user-support-for-xenapp-on-fma/) created

[DOWNLOAD SCRIPT FROM GITHUB](https://github.com/ryancbutler/StorefrontICACreator)
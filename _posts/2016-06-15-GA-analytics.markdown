---
layout: post
title:  "Use Google Analytic to monitor website"
date:   2016-06-20 11:08:00
categories: Google
---

I want to track visitors who access my webpage which could help me to analyze my websites. With the help of Google, I found that the *"Google Analytic"* is the perfect match for me !

Here are basic introduction of Google Analytic and steps to install Google Analytic to my jekyll website. To integrate GA to your website, you MUST have a google account to work.

### What is Google Analytic ?
Google Analytics tracking is a free web analytics service offered by Google that tracks and reports website traffic. It’s really easy to add the tracking code to your Jekyll static website.

### Steps for integrating GA in a website

* **1. Create A Reporting View for a website**

Login in to Google Analytics and create a view for a website under **"Admin>View>All Web Site>Create new view"**, which tells GA to monitor the specified website.

* **2. Add Google Analytics tracking to all your website pages**

Paste the Google Analytics tracking code into analytics.html and replace your Tracking ID with {{ site.google_analytics }}. When you have finished it should look like this.

~~~html
<script>
 (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
 (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
 m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
 })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

 ga('create', '{{ site.google_analytics }}', 'auto');
 ga('send', 'pageview');

</script>
~~~

* **3. Add Google Analytics Tracking ID to Jekyll’s "\_config.yml" file**

Now you need to reference your Tracking ID in your "\_config.yml" file. ReplaceUA—XXXXXXXX-X with your own unique Tracking ID.

~~~
Google services

google_analytics: UA—XXXXXXXX-X
~~~

Finally, open \_layouts/head.html and add the following code just before the end </head> tag. Google recommends this placement to track all of the pages on your website correctly.

```html
{\% if jekyll.environment == 'production' \%}
{\% include analytics.html \%}
{\% endif \%}
```

In future, when building your Jekyll project with jekyll build, you will need to prefix the command with JEKYLL_ENV=production like this

> JEKYLL_ENV=production jekyll build

### How to analyze

Once GA is successfully integrated, you can get lots of reporting information from GA to analyze the visitors behaviors, which may look like in the following picture. Under the side bar on the left click "Audience > Overview", select a period you want to track (I chose one-month period from 22 May to 22 Jun), then you will see curves showing the number of visitors visited daily and some statistic data for *Session*/*Users*/*Page Views*(etc) are there for your reference.

Say you want to see where your website visitors are from. Under *"Demographics > City"*,  you will see the distribution of where your visitors come from.

![Alt text](/resources/GA/GA.png)

To further investigate Google Analytic, please follow the below website:

> http://www.google.com/analytics/#?modal_active=none

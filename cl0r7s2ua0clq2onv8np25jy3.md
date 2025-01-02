---
title: "Detecting Microsoft Roadmap Changes with Power Automate"
seoTitle: "Use Power Automate for Targeted Microsoft Roadmap Updates"
seoDescription: "Track Microsoft Roadmap changes easily with Power Automate. Set up recurring flows to monitor updates and receive notifications for key features"
datePublished: Mon Mar 14 2022 21:25:00 GMT+0000 (Coordinated Universal Time)
cuid: cl0r7s2ua0clq2onv8np25jy3
slug: detecting-microsoft-roadmap-changes-power-automate
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1647287387169/vxFsrrUJ5.jpg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1647293090158/DcMG0GxJu.jpg
tags: rss, microsoft, automation, roadmap, power-automate, powerautomate

---

Microsoft communicates important product updates, feature additions and bug fixes via the [Roadmap](https://www.microsoft.com/en-us/microsoft-365/roadmap?filters=). It's quite effective and gives developers, administrators and stakeholders great visibility into what's coming down the release pipeline.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647284811284/3br06nRvh.png)

Major updates to Roadmap items are communicated via email and within each tenant's Message Center. But the Roadmap is vast - not every update warrants direct communication. Items change all the time without notification.

So how do we detect changes to the Roadmap items for which we are acutely interested? If you're eagerly awaiting a new feature, for example, how will you know if the release date gets pushed out?

### Option One - Check the Roadmap every day

Nobody has time for that!

Additionally, how would you know *definitively* that what you're looking at today is different than what you saw yesterday?

### Option Two - Subscribe to an item's RSS feed

Every item on the Roadmap has an RSS link. You could subscribe to the items you care about, and monitor for developments.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647284956732/p8Hwp_PrL.png)

However, not everyone uses RSS readers - let alone on a frequent basis, so it might be difficult to detect changes this way.

### Option Three - Poll the RSS feed at in interval with Power Automate

1) Create a scheduled, recurring flow using [Power Automate](https://powerautomate.microsoft.com/)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647285230276/iv6rMGTj3.png)

I chose to query the Roadmap weekly - every Wednesday.

2) Copy the RSS feed url from the Roadmap item you wish to track, and paste it within Automate's RSS action

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647285367672/c1FyZovP-.png)

3) Store the RSS output in a new object variable

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647285411751/M6v0LMYJE.png)

Use an expression to isolate the first return object:

`first(outputs('Feed')?['body'])`

4) Use the Get Past Time action to bracket your time range. You'll want this to be greater than or equal to your flow recurrence schedule. 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647285650211/SODcXVanH.png)

5) Use a Condition to determine if the Roadmap item has been updated within this timeframe

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647285788796/J4JfqIlET.png)

You'll need to grab the updatedOn value from the RSS object using a simple expression:

`variables('varObject')['updatedOn']` 

6) If the Roadmap item was updated within the last 10 days, you can have Automate send an email to your team, notify people via Teams - the choice is yours! If the Roadmap item hasn't been updated recently, the flow simply terminates and tries again the following week.

I went the simplest route, and had Automate email my team with a link back to the original Roadmap item we were interested in:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647286183896/Z-be9ilh3.png)

### How it Works

The Power Automate method is possible thanks to the values returned inside the RSS output. When a Roadmap item is touched, the updatedOn value is refreshed automatically. Even the smallest changes to Roadmap items will trigger a refreshed updatedOn value. We've been using this flow method for about a year now and have found it quite reliable.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647286617566/ntIIHARRe.png)

### How Could This Method Be Improved?

This method doesn't tell you *what* changed, specifically - only that *something* changed. Since we only use this for Roadmap items we are **keenly** interested in, it's usually pretty obvious what's changed, once we get the notification and take a look. And, it's almost always release dates. 😁

The RSS output doesn't contain all the information found within each Roadmap item, either. You won't see General Availability dates, release phase information, or tenant type specifics. If we had that information, we could do some cool change data capture and really drill down into the exact nature of each update. But for our use case, the method outlined here is more than enough.

### The Flow

Here is the whole thing put together:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647287494320/SBORIHWya.png)
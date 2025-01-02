---
title: "Environment Variables in Power Platform Solutions"
seoTitle: "Power Platform Environment Variables Guide"
seoDescription: "Manage environment variables to streamline Power Platform solutions. Clean and optimize connections and flows for easier deployment"
datePublished: Fri Apr 01 2022 22:16:20 GMT+0000 (Coordinated Universal Time)
cuid: cl1gzjfgl04stkbnv1857770x
slug: environment-variables-in-power-platform-solutions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1648850596587/MD0HmtpP3.jpg
tags: variables, microsoft, development, automation

---

## Overview

The availability of [environment variables](https://docs.microsoft.com/en-us/powerapps/maker/data-platform/environmentvariables) within Power Platform solutions is an exciting piece of tech. But like most developing features and platforms, there are some rough edges and pitfalls to be aware of.

Our recent push to a more mature [ALM](https://en.wikipedia.org/wiki/Application_lifecycle_management) model for custom solutions exposed some of those rough edges, so I sought to help clarify and document some guidelines and tips - for my own sanity and yours!

It's easy to get started with environment variables. It's enticingly easy to create them within a solution or environment. Your apps and flows create them automatically, too - just like connections. If you're like me, you'll be absolutely SWIMMING in variables before long - many of them duplicative, redundant or unused.

<div data-node-type="callout">
<div data-node-type="callout-emoji">⚠</div>
<div data-node-type="callout-text">January 2025 note: Some of the information on this page is outdated!</div>
</div>

## The Untangling Process

### Connections

I started with the environment connections - sort of the lowest common denominator in solution config. Power Platform loves to create these on the fly, leaving you with tons of duplicates and no viable way to rename or organize them. (Side note: I used to have luck renaming connections by editing the JSON source with [Flow Studio](https://flowstudio.app/), but I think Microsoft has locked that particular backdoor)

![accessing power platform connections](https://cdn.hashnode.com/res/hashnode/image/upload/v1648846337280/OtSVwWL7W.png align="left")

You can quickly see if a connection is being used by accessing the details pane, and checking the app and flow panes. Not being used? Delete it!

![connection details](https://cdn.hashnode.com/res/hashnode/image/upload/v1648849194441/HZLHW8pCy.png align="left")

But what if you have duplicate connections and they're being used? You can either live with the mess, or clean it up. Living with the mess is a viable option - redundant connections are not as detrimental to a deployment as extra environment variables.

Since this was a DEV environment, I made the decision to try and pare everything down to just ONE connection for each connector type. I knew this would cause rework, but I'd be tinkering with the apps and flows later, giving me the chance to wire everything back up again.

Once I was done, I was left with a clean and tidy set of connections. I *really* should have taken a "before" picture when this list was five times longer (easily) with duplicates galore.

![cleaned connections](https://cdn.hashnode.com/res/hashnode/image/upload/v1648848820034/zj5jk5wE-.png align="left")

Doing this prework will ensure you have *one* SharePoint connection, *one* SQL connection, etc. for use within connection references and the environment variables we'll be sorting out next.

### Environment Variables

A fair amount of trial and error led me to the following observations. Microsoft may smooth this experience out in the future, but I believe these findings are accurate at the time of this posting:

1. Data source variables created in an **app** can be used elsewhere in a solution (within flows, e.g.)
    
2. Data source variables created in a **flow** cannot be used in an app within the same solution
    
3. Data source variables created in a **solution** cannot be used in apps, but can be used within flows
    

The conclusion here is obvious - let the app(s) within your solution create your site and list variables when working with the SharePoint connector. *Don't* create them within the solution itself or within Automate.

If you've already got a tangled mess of environment variables in your solution, and aren't sure which ones were created from PowerApps, you'll probably want a fresh start. Append something like "DELETEME" to the preexisting variables within your solution.

![flag confusing variables for deletion later](https://cdn.hashnode.com/res/hashnode/image/upload/v1648849922461/McPXkKiCW.png align="left")

Before deleting the old mess, however, we'll want to create clean, permanent variables - our "keepers." To get started, open your app, remove your SharePoint data sources, and take a deep breath as you watch countless error warnings cascade across your screen.

When you re-add your data sources to the app, the Advanced tab will not work. Don't despair. Paste in the site URL and the app will create variables on the next step within the pane - one for the site, and one for each list.

![confusing data source interface powerapps](https://cdn.hashnode.com/res/hashnode/image/upload/v1648851095723/PcLC1HldT.png align="left")

Note: If your app doesn't seem to be creating variables for you automatically, check **Settings -&gt; General -&gt; Automatically create environment variables** when adding data sources

Once those variables are created, back out of the app, find them within your solution, and give them friendly titles. I found it helpful to use a convention like:

```plaintext
type_VarName
```

Delete any site/list variables created via other methods (manually, or via Automate) - if you marked them earlier, this step will be easy. Sometimes deleting one will cause several to disappear at once, because you're kneecapping dependencies. Be careful not to delete the variables you just created.

### Rewiring Flows

Go back into your flows, and remap any Site and List values. You must X out the current values and work methodically for this to work correctly. A tip from a colleague: let the list names resolve after you add the site variable, to ensure the method is working.

![rewiring flow config](https://cdn.hashnode.com/res/hashnode/image/upload/v1648847972866/sOLlCs63Y.png align="left")

Your flow should save without errors. If you have errors, go back, and make doubly sure you clear the existing site and list values before adding your app-created environment variables.

### Other Recommendations

If you have environment variables floating around, outside of your custom solution(s), you can find them in the Default Solution in your development environment:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648848128114/XfE5FZ6dk.png align="left")

You'll probably want to clean this up too, while you're at it. Be careful not to delete anything you want to keep!

### Conclusion

When you're finished, you should have an ultra clean and easy-to-understand, minimalist set of environment variables. This will make solution development, packaging and deployment *much* easier!

I went from 12 environment variables down to 3, by following the steps and guidelines outlined above.

![clean and tidy environment variables](https://cdn.hashnode.com/res/hashnode/image/upload/v1648848216617/zrhAQ59yM.png align="left")

It's worth mentioning, this is a loose approach. Your situation may vary. And these guidelines really only apply to SharePoint Online environment variables. Once Microsoft extends data source environment variables to other connector types, the experience may get smoothed out and the information in this article will be outdated. Only time will tell. 😀
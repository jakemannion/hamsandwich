---
title: "Managing Permissions in SharePoint Online with Power Automate"
seoTitle: "Power Automate: SharePoint Online Permissions Guide"
seoDescription: "Explore how to manage SharePoint Online permissions using Power Automate with HTTP actions and SharePoint REST API for efficient automation"
datePublished: Fri May 27 2022 00:06:07 GMT+0000 (Coordinated Universal Time)
cuid: cl3nooh5e02lze0nv1uamhwiq
slug: managing-permissions-in-sharepoint-online-with-power-automate
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1653594273735/dRTmXz1y1.jpg
tags: security, rest-api, sharepoint, powerplatform, powerautomate

---

# Background

SharePoint is known for its highly customizable permissions. That flexibility can have its perils, of course, but with the right approach (and for the right scenarios) it can be very effective.

Power Automate provides a few basic connector actions for working with SharePoint permissions:

1. *Revoke access* to an item
    
2. Create access *links* for an item
    

Not much for now, but fortunately the **HTTP to SharePoint** action opens up numerous other possibilities.

![http to sharepoint action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653594177575/zjwp9i7LH.png align="left")

# Familiar Methods, New Approach

SharePoint's REST API has a number of endpoints for managing permissions. Most methods can be re-used across different entities like sites, lists, items, etc. If you've done on-premises SharePoint development, you may already be familiar [with some of them](https://www.c-sharpcorner.com/UploadFile/fc34aa/break-inheritance-and-add-role-permissions-using-rest-api-in/).

The calls are simple - no payloads, headers or returns to process for most scenarios. The only tricky part can be finding the exact syntax, and there may be some data peculiarities to manage, depending on your scenario.

# Power Automate HTTP Examples - SharePoint Permissions

## Getting the Principal ID of a Group

Type: GET

![flow http action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653609712448/lqdSAD-mh.png align="left")

Note: Replace *GROUP NAME* with your target group name

```plaintext
_api/web/sitegroups/getbyname('GROUP NAME')/Id
```

## Getting Principal IDs for *All* Groups

![flow http action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653597117939/q0RBXQ6eF.png align="left")

```plaintext
_api/web/sitegroups?$select=Title,ID
```

Results will be an array of all groups available on the site.

Note: You can find group IDs manually by clicking individual groups within Site Settings -&gt; People and Groups and checking the address bar.

![People and Groups](https://cdn.hashnode.com/res/hashnode/image/upload/v1653609560344/wkyKIyUwv.png align="left")

Cutting this corner won't save you much time, however. For most scenarios, it's better to fetch the value(s) at runtime.

## Getting Principal ID for Specific User

![exploring data structure with SP Insider](https://cdn.hashnode.com/res/hashnode/image/upload/v1653602710522/XFQm8joSt.png align="left")

There are a couple ways to do this. First, you might want to use a tool like [SP Insider](https://chrome.google.com/webstore/detail/sp-insider/gjckpigahcbffmeofjfedlffddhfidhj?hl=en) (pictured above) to get familiar with the SiteUsers data structure within your site.

![flow action http](https://cdn.hashnode.com/res/hashnode/image/upload/v1653602837728/Ru6xDxgR9.png align="left")

Here are two sample queries - one for email, one for user name:

```plaintext
_api/Web/SiteUsers?$filter=Email eq 'Sample.Person@samplesite.com'
```

Note: The search terms are case sensitive!

```plaintext
_api/Web/SiteUsers?$filter=Title eq 'Sample Person'
```

![http result array](https://cdn.hashnode.com/res/hashnode/image/upload/v1653602203605/rxIHGnor3.png align="left")

You'll get a single-node array in the return, containing the user's ID value for that site.

## Getting Principal IDs for All Users

![flow http action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653609661323/fCd7Rb-8e.png align="left")

You'll be getting a lot of data back with this one, potentially, so consider using *select* to narrow the return down to just the fields you need.

```plaintext
_api/Web/SiteUsers?$select=ID,Title,Email
```

There are some important limitations to consider here:

1. SiteUsers data is *site-specific*
    
2. A user's Principal ID on *one* site will be different from their Principal ID on *another* site
    
3. The SiteUsers list will only contain records for *users who have visited the site* (or have been added via a few other methods).
    

There is an [ensureUser method](https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-visio/jj245233(v=office.15)) to explore, if you want to pursue this approach further.

## Getting All List/Library GUIDs

I tend to work with GUIDs more than GetByTitle endpoints, so the examples on this page are tailored to that approach. Here's a quick way to get all list GUIDs on a site, in case you need it:

![flow action GUIDs](https://cdn.hashnode.com/res/hashnode/image/upload/v1653608951166/CM_u0Odja.png align="left")

Type: GET

```plaintext
/_api/Web/Lists?&$select=Title, ID
```

## Getting All Role Definitions

Each permission role in SharePoint (Full Control, Edit, etc.) has a specific ID associated with it. You'll need this value for some of the example calls below.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653608272920/z7wfD5kHh.png align="left")

Type: GET

```plaintext
_api/web/roledefinitions
```

Results will be an array containing objects for each role definition.

## Get Current Permissions on Item/Document

Type: GET

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681151777936/86c44ab9-e082-4506-858a-6fe7b10e3f2c.png align="center")

```plaintext
_api/web/lists/getbytitle('LIST NAME')/items(ITEM ID)/roleassignments
```

Will return an array of objects. Each **PrincipalId** will tell you *who* has permission to the item, but not the permission type.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681153060752/575ed915-4703-4d4c-8d29-f29d2962c35f.png align="center")

Use an **Apply to Each** step to loop through the results and perform further actions.

## Break Permission Inheritance on an Item or Document

![flow action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653608716377/aijsSa0n9.png align="left")

Type: POST

```plaintext
_api/Web/Lists(guid'LIST GUID')/Items(12)/breakroleinheritance(copyRoleAssignments=true, clearSubscopes=true)
```

## Restore Permission Inheritance on Item/Document

\*Added August, 2024

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Tip of the cap to <a target="_blank" rel="noopener noreferrer nofollow" href="https://tomriha.com/how-to-restore-sharepoint-item-file-permissions-with-power-automate/" style="pointer-events: none">Tom Riha</a> on this one, which I neglected to include when I first slapped this page together. Saved me a few minutes of head scratching tonight.</div>
</div>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724045162715/7435a2a0-a1bb-47b4-8e09-ca3b00363ea4.png align="center")

```plaintext
_api/Web/lists/getByTitle('YOUR-LIST-OR-LIBRARY-HERE')/items(ITEM-ID-HERE)/ResetRoleInheritance()
```

Type: POST

## Remove Permissions from a List/Library

![flow http action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653594693429/JL9p6OsuQ.png align="left")

Type: DELETE

```plaintext
_api/Web/Lists(guid'LIST GUID')/RoleAssignments/GetByPrincipalId(5)
```

Note: Replace the principal ID value for the user/group you are targeting, and the GUID for your list or library.

## Remove Permissions from an Item/Document

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653609149352/7CJqZEyzM.png align="left")

Type: DELETE

```plaintext
_api/Web/Lists(guid'LIST GUID')/Items(12)/RoleAssignments/GetByPrincipalId(5)
```

Note: Replace the the principal ID and list GUID values to suit your specific scenario

You don't need a specific role definition for this one - it will simply remove the target entity's permissions (whatever those may be) for the item.

## Add Permissions to a List/Library

![flow http action](https://cdn.hashnode.com/res/hashnode/image/upload/v1653594982798/oCusLlUbk.png align="left")

Type: POST

```plaintext
_api/Web/Lists(guid'LIST GUID')/RoleAssignments/addroleassignment(principalid=5,roledefid=1073741827)
```

Note: Replace the the principal ID, role definition and list GUID values to suit your specific scenario

# Conclusion

That covers many of the more common permission management activities in SharePoint. You can extend this approach to cover lots of other scenarios, as well. 👍
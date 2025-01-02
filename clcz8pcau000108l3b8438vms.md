---
title: "Handling Paginated Results from Microsoft Graph with Azure Data Factory"
seoTitle: "Paginated Microsoft Graph Results in Azure Data Factory"
seoDescription: "Learn to handle paginated results from Microsoft Graph using Azure Data Factory with linked services, datasets, and pagination rules"
datePublished: Mon Jan 16 2023 20:09:29 GMT+0000 (Coordinated Universal Time)
cuid: clcz8pcau000108l3b8438vms
slug: handling-paginated-results-from-microsoft-graph-with-azure-data-factory
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1673899265363/230c5f79-1b50-4cc4-ae77-1c1581554227.jpeg
tags: microsoft, azure, apis, graph, azure-data-factory

---

This topic is covered in a handful of blog posts, forum topics, etc. but none of the information I found recently got me 100% of the way there. That can happen with methods involving two evolving services - [Azure Data Factory](https://learn.microsoft.com/en-us/azure/data-factory/) and [Microsoft Graph](https://learn.microsoft.com/en-us/graph/use-the-api) - information becomes out of date.

Here's what worked for me in early 2023!

# Prerequisites

Even though some of these are detailed topics, I am going to gloss over them since they are covered in-depth elsewhere on the web:

1. [Register an Azure App](https://learn.microsoft.com/en-us/graph/auth-register-app-v2) for use as a service principal (later) with your data factory
    
2. Grant the application [appropriate roles](https://learn.microsoft.com/en-us/graph/permissions-reference) for your intended use of Graph
    
3. Store the *client secret* value for the app in [Azure Key Vaul](https://azure.microsoft.com/en-us/products/key-vault/)t
    
4. Enable managed identities for your data factory
    
5. Give the managed identity for your data factory the appropriate role on the Key Vault (to retrieve stored *client secret*)
    
6. Configure a data sink to store the Graph results, and ensure the data factory has permission to write there
    
7. Create a new [Linked Service](https://learn.microsoft.com/en-us/azure/data-factory/concepts-linked-services?tabs=data-factory) within ADF (type: REST)
    
    1. Set the Authentication type as **Service Principal**
        
    2. Use the Azure Key Vault method to retrieve the *client secret* from AKV
        
    3. AAD Resource: [**https://graph.microsoft.com/**](https://graph.microsoft.com/)
        
    4. Parameterize the Base URL value
        

This will get you something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673897531734/8b9d0cb5-8105-4a1f-aa7c-3e792f084013.png align="center")

# Dataset, Pipeline and Activity Setup

1. Create a new Dataset using the new Linked Service you created (above steps)
    
    1. You can parameterize the **baseURL** value if you'd like to use *one* Dataset for *multiple* Graph queries.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673898081449/92b92657-c219-47ba-9588-31baa4dd2049.png align="center")
        
2. Otherwise, provide a fixed Graph API endpoint value within the Dataset, like:
    
    https://graph.microsoft.com/v1.0/users?$select=id,accountEnabled,createdDateTime,department,displayName,employeeId,employeeType,jobTitle,mail,userPrincipalName,userType,manager,employeeHireDate&$expand=manager($select=id)
    
3. Create a new Pipeline and add a Copy Data activity
    
4. Configure the Copy Data activity to use the Dataset you created
    
5. Delete the automatically-supplied Pagination criteria (RFC5988)
    
6. Add a new Pagination Rule, like this:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673898523166/87e4ede4-a47c-4881-8f0e-6bb024ce8796.png align="center")
    

Your source definition within the Copy activity should look something like this:

```json
"source": {
    "type": "RestSource",
    "httpRequestTimeout": "00:01:40",
    "requestInterval": "00.00:00:00.010",
    "requestMethod": "GET",
    "paginationRules": {
        "AbsoluteUrl": "$['@odata.nextLink']"
    }
}
```

Now, when your Copy Data activity executes, it will keep fetching results from Graph until the **@odata.nextlink** is no longer present in the result body, at which point it will start the sink activity.

> Note: this is my first time using the new Neptune editor on Hashnode ... forgive any oddities with the numbered list formatting... I am in a bit of a hurry and will try to fix the markdown later
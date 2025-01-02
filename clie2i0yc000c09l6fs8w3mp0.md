---
title: "Create Planner Tasks with Microsoft Form Attachments"
seoTitle: "Create Planner Tasks from Microsoft Form Attachments - Power Automate"
seoDescription: "Learn to automate creating Planner tasks with Microsoft Form attachments, manage storage locations, and handle attachment workflows efficiently"
datePublished: Fri Jun 02 2023 04:30:54 GMT+0000 (Coordinated Universal Time)
cuid: clie2i0yc000c09l6fs8w3mp0
slug: create-planner-tasks-with-microsoft-form-attachments
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685680099701/48c96273-fbca-47fc-8eda-705cbd1f1f32.jpeg
tags: forms, powerplatform, logic-apps, microsoft365, planner

---

## Form Options

There are several options available when file uploads are enabled for Microsoft Forms:

* Can be optional or required
    
* Can be limited from 1-10 attachments
    
* Can restrict to certain file types
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685676368941/8bf3f9a4-9fe8-4710-b7c4-5c3b668c812a.png align="center")

You can construct workflow logic around these options, but it's usually best to handle a multitude of scenarios, in case the form or connector changes later.

## Storage Locations

Attachments for [both types of Forms](https://support.microsoft.com/en-au/office/create-a-group-form-or-quiz-7228eebb-a6ab-45ec-8123-52026a2f52ff) are stored in different locations:

| Form Ownership | Location |
| --- | --- |
| Personal | Stored within the form creator's OneDrive, under ...**\\Apps\\Microsoft Forms\\** |
| Group | Stored within the group's file storage, within the General folder for the associated Team |

**Tip**: Using the same group for both the Form (source) and Planner board (destination) simplifies permissions and avoids the need to move attachment content.

## Workflow Logic

The automation is triggered by new form submissions. With each run, the details for each submission are collected and a corresponding Planner task is created.

Optionally, add a condition to determine if attachment(s) were included in the form response, so the subsequent logic can account for both scenarios:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685676993872/21a09390-3954-4b34-bc01-4c0c7c9cbb2d.png align="center")

You will need an expression to determine if the file upload "question" within the form is empty or not:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685677484948/48167799-e461-4adf-aa20-3968940e06f3.png align="center")

The expression uses a *length* function on the question within the form that collects attachments. If the outcome is greater than 0, the workflow will "know" attachments were included and can respond accordingly:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685677937439/fd4cb300-9742-4dc9-9d52-5555348f88c2.png align="center")

Within the **Yes** branch of the condition, use a [Select action](https://learn.microsoft.com/en-us/power-automate/data-operations#use-the-select-action) to convert the array of form attachments into a format compatible with Planner. Once complete, it will look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685678044992/cb80ae76-b540-4ece-989b-89df1795f49d.png align="center")

For the **From** field, wrap the dynamic value used above (your attachment "question" within the form) inside a JSON function:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685678128221/40c8ef74-5703-4463-bcc9-e43343f8dc6f.png align="center")

Next, map two properties - **resourceLink** and **alias**. Toggle to advanced/text mode...

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685678330489/6d88ba04-f652-438f-bef7-99a57feb5c9b.png align="center")

...and paste the following snippet:

```plaintext
{
    "resourceLink": @{item()?['link']},
    "alias": @{item()?['name']}
}
```

Once that's complete, add an **Update Task Details** action, using the ID value from the preceding **Create Task** action:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685678804191/63a607f6-34cc-4a45-b767-500c38fc9d77.png align="center")

Switch to advanced mode within the References section of the Select action and add the Output from the previous Select statement:

**Note**: attachments in Planner are known internally as *references*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685679369582/8ac268e6-be67-4382-a357-209da34c973a.png align="center")

At this point, your attachment handling logic should be complete.

Create a test form submission, and you should see a task card with the correct attachments included:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685679754162/cc83ed04-36da-44f8-bd23-a64d8c838dff.png align="center")

Don't forget to go back and complete the **No** path for the *"does the response contain attachments"* condition, and incorporate any additional logic you might need in the workflow.
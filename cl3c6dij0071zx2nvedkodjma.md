---
title: "Parsing Custom Planner Labels with Power Automate"
seoTitle: "Planner Label Parsing with Power Automate"
seoDescription: "Guided steps (with pictures and code) to work with Microsoft Planner task labels using Power Automate"
datePublished: Wed May 18 2022 22:48:15 GMT+0000 (Coordinated Universal Time)
cuid: cl3c6dij0071zx2nvedkodjma
slug: parsing-custom-planner-labels-with-power-automate
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652912933691/hiN8Q0Hyj.jpg
tags: microsoft, sql, power-automate, powerplatform, planner, powerautomate

---

# Context

I'm a fan of [Microsoft Planner](https://tasks.office.com/). It's easy to use, flexible, and the UI/UX is rock solid. The tie-in with O365 Groups and Teams is a strong selling point, as well.

Now, Planner isn't the perfect solution for *every* scenario (you probably wouldn't want to manage sprints with it) but it is very popular within many companies, large and small.

## Definitions

Microsoft naming conventions don't always make things easy, so:

| Term | Meaning |
| --- | --- |
| Planner | the name for the overall application/service |
| Plan | an individual task board for a team or person |
| Task | the items within a Plan |
| Bucket | used to group collections of similar tasks |
| Label | a way to tag/classify individual tasks |

## Integration Opportunities Abound

Planner can be used in many different ways, but at its core it's all about task management, which makes it a juicy target for automation. The [Power Platform connector](https://docs.microsoft.com/en-us/connectors/planner/) is pretty robust, and you can extend that even further by hitting the Planner API through [Microsoft Graph](https://docs.microsoft.com/en-us/graph/overview).

However, any integration efforts quickly expose Planner's underlying data structure, which has some quirks.

# Task Labels

![planner label default name](https://cdn.hashnode.com/res/hashnode/image/upload/v1652907372232/jIfiT92CE.png align="left")

Planner boards come with 25 labels to help you and your team classify the individual tasks on the board. Each label has a *default* name - its associated color - and that name can be customized to provide more meaning and context for the plan users. Cool feature!

![Planner task label customization](https://cdn.hashnode.com/res/hashnode/image/upload/v1652907131573/hL91NaTY5.png align="left")

Customizations, like label names, are contained at the individual *plan* level. For example, one team might use *red* for marking "Urgent" work and a different team may use it to classify something as "Top Secret."

## The Challenge

When retrieving Planner *task* data, any label information comes back as an **appliedCategories** object:

![label data from task](https://cdn.hashnode.com/res/hashnode/image/upload/v1652908590244/IIymDzzzZ.png align="left")

It's not an array.

There are no name/value pairs.

It's just... *booleans* with no tie-back to the label names. What does **category18** mean? Has it been customized? How can I tell?

Fortunately, you can get a bit of a "glossary" if you retrieve the overall *plan* data:

![plan label customization data](https://cdn.hashnode.com/res/hashnode/image/upload/v1652909292622/tOctbXrNT.png align="left")

That's great, but how do I map those values? How do I bounce a name/value object against an object of booleans?

## The Power Automate Way

Different platforms/languages will have their own approach to the challenges above. For [Automate](https://powerautomate.microsoft.com/), it required some considerable thinking!

At the time this came up, I was working on a solution that iterated over *many* plans, so I needed a method that scaled. If you are working with data from just an individual plan, there are definitely some corners you could cut.

### Building a Glossary Array

1.) When I got to the point in my automation where I was working with individual plans, I used the **Get plan details** action.

![plan details and array](https://cdn.hashnode.com/res/hashnode/image/upload/v1652910714701/fZDddaP94.png align="left")

2.) Next, I used a **Compose** statement to craft my array. It took a lot of typing, but copy/paste made it less of an ordeal.

![coalesce expression within array](https://cdn.hashnode.com/res/hashnode/image/upload/v1652910916961/nTTsqO4RQ.png align="left")

3.) Most teams that use labels within Planner also customize the label names - but it's not an absolute certainty. The **Get plan details** action only returns customized values. I wanted to be certain I covered both scenarios, so I used a coalesce expression within each node of the array.

### Creating a Task Label Array

The steps above get us halfway there, but we still need to handle the other part of the mapping - the data from each individual task.

![parse JSON for task labels](https://cdn.hashnode.com/res/hashnode/image/upload/v1652911439704/FzSoxExpW.png align="left")

4.) Inside of a **For each task** loop, I started with a **Parse JSON** step, using the **appliedCategories** value as the content source.

5.) The schema needs to be created manually (unless you magically have a task with all 25 labels applied, to feed into the generator):

{ "type": "object", "properties": { "category1": { "type": "boolean" }, "category2": { "type": "boolean" }, "category3": { "type": "boolean" }, "category4": { "type": "boolean" }, "category5": { "type": "boolean" }, "category6": { "type": "boolean" }, "category7": { "type": "boolean" }, "category8": { "type": "boolean" }, "category9": { "type": "boolean" }, "category10": { "type": "boolean" }, "category11": { "type": "boolean" }, "category12": { "type": "boolean" }, "category13": { "type": "boolean" }, "category14": { "type": "boolean" }, "category15": { "type": "boolean" }, "category16": { "type": "boolean" }, "category17": { "type": "boolean" }, "category18": { "type": "boolean" }, "category19": { "type": "boolean" }, "category20": { "type": "boolean" }, "category21": { "type": "boolean" }, "category22": { "type": "boolean" }, "category23": { "type": "boolean" }, "category24": { "type": "boolean" }, "category25": { "type": "boolean" } } }

The **Parse JSON** step is important because it saves you having to write 25 expressions with null-handling logic in the next step.

![task array creation](https://cdn.hashnode.com/res/hashnode/image/upload/v1652911792904/VfYKm_0uW.png align="left")

6.) Next, I used a **Compose** statement (and a lot of copy/pasting) to create the initial array for the task labels. Note the key/value format matches the "glossary" we created previously.

![filtering nulls out of array](https://cdn.hashnode.com/res/hashnode/image/upload/v1652912429052/rRKoSZl0P.png align="left")

7.) Then we **filter** the task label array to remove any nodes containing nulls. Booleans are tricky when filtering arrays in Automate, so switch to advanced mode and use an expression:

`@equals(item()?['value'], true)`

This yields an array like this:

\[ { "key": "category14", "value": true }, { "key": "category18", "value": true }, { "key": "category19", "value": true }, { "key": "category23", "value": true } \]

### Putting it All Together

8.) Now, you've got matching arrays - both sides of the equation! Where you take it from here may vary depending on your solution needs.

![final iteration and array filtering](https://cdn.hashnode.com/res/hashnode/image/upload/v1652912740639/3c0hwcamA.png align="left")

I needed each applied label as an individual record, so I **looped** through the newly-created task label array, **filtering** against the *glossary* array we created earlier with each iteration.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><strong>Start of Additional Information</strong> - October, 2023</div>
</div>

A couple people reached out to ask for more details on step #8 recently.

Here are the details on the *For Each* action and the *Filter* step just inside. You may need to zoom in or click the picture to see smaller details:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697259257592/84962063-c316-4983-93a2-51369c552ed7.png align="center")

This last part may vary, depending on how you want the label data structured in your flow. I packed each value into an object and rolled those into an array variable using an *Append* action:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697259569761/69284625-ffe9-4db6-a09b-70c2003222d9.png align="center")

Here is the object, with each expression expanded. I include taskID since it's an effective way to relate child entities in Planner (labels, assignments, etc.) back to the parent:

```json
{
  "labelTitle": first(body('filter_plan_labels_with_task_labels'))?['value'],
  "taskID": items('for_each_task')?['ID'],
  "labelID": int(
                substring(
                    first(body('filter_plan_labels_with_task_labels'))?['key'],
                    8
                )
             )
}
```

The final stage for this workflow gathers up the data collected for the various Planner entities and ships it off to a separate flow for processing and storage.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697260427550/8bb0e481-4171-400c-8ba7-f20d42135be9.png align="center")

I hope that helps!

There is a pretty fun story to be told about this particular solution, but I will save that for another article - don't want to get off track here tonight.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><strong>End of Additional Information</strong> - October. 2023</div>
</div>

## Conclusion

This was an interesting puzzle to solve, and the only real drawback was that it involved a fair amount of repetition. But now that it's done, I've got a robust solution that can be scaled considerably.

Some of the upper-echelon Automate junkies might be able to squeeze a *little* more speed out of this method, but overall it is quite performant. Using **Compose** actions instead of variables any time you can get away with it definitely helps. Automate handles arrays and array filtering very quickly, too.

## Flow Source

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><strong>Additional Information Added</strong> - June, 2024</div>
</div>

If you want to see a mermaid visual of the logic flow, please visit [this GitHub Gist](https://gist.github.com/jakemannion/a96d4575c70f520a54ab7689f69654cc).

And here is the (sanitized) source JSON for the flow:

```json
{
  "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "$connections": {
      "defaultValue": {},
      "type": "Object"
    },
    "$authentication": {
      "defaultValue": {},
      "type": "SecureObject"
    },
    "bool_emailNotifications (jake_emailNotifications)": {
      "defaultValue": true,
      "type": "Bool",
      "metadata": {
        "schemaName": "jake_emailNotifications",
        "description": "Set to No if you want to turn off 'process complete' emails"
      }
    },
    "text_sqlServer (jake_sqlServer)": {
      "defaultValue": "***SCRUBBED***",
      "type": "String",
      "metadata": {
        "schemaName": "jake_sqlServer"
      }
    },
    "text_sqlDatabase (jake_sqlTable)": {
      "defaultValue": "***SCRUBBED***",
      "type": "String",
      "metadata": {
        "schemaName": "jake_sqlTable"
      }
    },
    "text_emailRecipients (jake_emailRecipients)": {
      "defaultValue": "***SCRUBBED***",
      "type": "String",
      "metadata": {
        "schemaName": "jake_emailRecipients"
      }
    },
    "text_processFlowEndpoint (jake_test_postURL)": {
      "defaultValue": "***SCRUBBED***",
      "type": "String",
      "metadata": {
        "schemaName": "jake_test_postURL",
        "description": "endpoint for triggering processing sub flow "
      }
    }
  },
  "triggers": {
    "Recurrence": {
      "recurrence": {
        "frequency": "Day",
        "interval": 1,
        "timeZone": "Pacific Standard Time",
        "startTime": "2022-05-01T00:00:00Z",
        "schedule": {
          "hours": [
            "1"
          ]
        }
      },
      "metadata": {
        "operationMetadataId": "48f13d96-ccee-42b8-9de4-d9e92c683b79"
      },
      "type": "Recurrence"
    }
  },
  "actions": {
    "init_assignmentArray": {
      "runAfter": {
        "init_taskArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "ee36ea65-589b-4cbb-88f4-4d427253f226"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "assignmentArray",
            "type": "array"
          }
        ]
      }
    },
    "init_taskObject": {
      "runAfter": {
        "init_errorCount": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "df6b1fbf-f47b-42a4-ac2e-f6af361e2887"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "taskObject",
            "type": "object",
            "value": {
              "taskID": "",
              "planID": "",
              "bucketID": "",
              "taskTitle": "",
              "planTitle": "",
              "bucketTitle": "",
              "taskDescription": "",
              "startDate": "",
              "createdDate": "",
              "dueDate": "",
              "completeDate": "",
              "createdByID": "",
              "orderHint": "",
              "pctComplete": ""
            }
          }
        ]
      }
    },
    "init_errorString": {
      "runAfter": {
        "init_outcomeArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "8b4796a3-4846-4fef-9c94-11860d9767c9"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "errorString",
            "type": "string"
          }
        ]
      }
    },
    "init_taskArray": {
      "runAfter": {
        "init_taskObject": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "242f3f77-c144-4d0e-a9f4-5dd030a23526"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "taskArray",
            "type": "array"
          }
        ]
      }
    },
    "init_checklistArray": {
      "runAfter": {
        "init_labelArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "a8e30162-dc67-40a2-aa2c-5c3d9234b896"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "checklistArray",
            "type": "array"
          }
        ]
      }
    },
    "init_labelArray": {
      "runAfter": {
        "init_labelValuesArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "fb3b0418-82c1-4001-8135-04c7bcde2a7b"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "labelArray",
            "type": "array"
          }
        ]
      }
    },
    "init_attachmentArray": {
      "runAfter": {
        "init_assignmentArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "40513abf-01c1-4699-8681-263ee9e882a4"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "attachmentArray",
            "type": "array"
          }
        ]
      }
    },
    "init_labelValuesArray": {
      "runAfter": {
        "init_attachmentArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "9d17111f-b964-422c-b416-9a3ee0b59349"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "labelValuesArray",
            "type": "array"
          }
        ]
      },
      "description": "used for storing the label info for a given plan - dont think it's used"
    },
    "Plan_and_task_iteration": {
      "actions": {
        "for_each_plan": {
          "foreach": "@outputs('get_active_plans')?['body/value']",
          "actions": {
            "List_buckets": {
              "runAfter": {},
              "metadata": {
                "operationMetadataId": "0e8aa5dd-aac0-4784-b418-693f426629a2"
              },
              "type": "OpenApiConnection",
              "inputs": {
                "host": {
                  "connectionName": "shared_planner",
                  "operationId": "ListBuckets_V3",
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_planner"
                },
                "parameters": {
                  "groupId": "@items('for_each_plan')?['GroupID']",
                  "id": "@items('for_each_plan')?['PlanID']"
                },
                "authentication": "@parameters('$authentication')"
              }
            },
            "List_tasks": {
              "runAfter": {
                "plan_label_values_array": [
                  "Succeeded"
                ]
              },
              "metadata": {
                "operationMetadataId": "d7847aa4-8952-41c1-8f4e-533c880b946b"
              },
              "type": "OpenApiConnection",
              "inputs": {
                "host": {
                  "connectionName": "shared_planner",
                  "operationId": "ListTasks_V3",
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_planner"
                },
                "parameters": {
                  "groupId": "@items('for_each_plan')?['GroupID']",
                  "id": "@items('for_each_plan')?['PlanID']"
                },
                "authentication": "@parameters('$authentication')"
              }
            },
            "for_each_task": {
              "foreach": "@outputs('List_tasks')?['body/value']",
              "actions": {
                "filter_bucket_array": {
                  "runAfter": {},
                  "metadata": {
                    "operationMetadataId": "bab69193-9ca3-4715-8d61-254f7fa86136"
                  },
                  "type": "Query",
                  "inputs": {
                    "from": "@outputs('List_buckets')?['body/value']",
                    "where": "@equals(item()?['id'], items('for_each_task')?['bucketId'])"
                  }
                },
                "is_more_task_detail_needed": {
                  "actions": {
                    "Get_task_details": {
                      "runAfter": {},
                      "metadata": {
                        "operationMetadataId": "40bd4ba4-1fe8-4566-9c90-783a1a028b88"
                      },
                      "type": "OpenApiConnection",
                      "inputs": {
                        "host": {
                          "connectionName": "shared_planner",
                          "operationId": "GetTaskDetails_V2",
                          "apiId": "/providers/Microsoft.PowerApps/apis/shared_planner"
                        },
                        "parameters": {
                          "id": "@items('for_each_task')?['id']"
                        },
                        "authentication": "@parameters('$authentication')"
                      }
                    },
                    "for_each_checklist_item": {
                      "foreach": "@outputs('Get_task_details')?['body/checklist']",
                      "actions": {
                        "append_to_checklist_array": {
                          "runAfter": {},
                          "metadata": {
                            "operationMetadataId": "77340521-feb2-4cf3-8a6d-00f31d7277d1"
                          },
                          "type": "AppendToArrayVariable",
                          "inputs": {
                            "name": "checklistArray",
                            "value": {
                              "itemTitle": "@items('for_each_checklist_item')?['value/title']",
                              "itemComplete": "@items('for_each_checklist_item')?['value/isChecked']",
                              "taskID": "@items('for_each_task')?['id']",
                              "orderHint": "@items('for_each_checklist_item')?['value/orderHint']",
                              "itemID": "@int( items('for_each_checklist_item')?['id'] )",
                              "modifiedByID": "@items('for_each_checklist_item')?['value/lastModifiedBy/user/id']",
                              "modifiedDate": "@items('for_each_checklist_item')?['value/lastModifiedDateTime']"
                            }
                          }
                        }
                      },
                      "runAfter": {
                        "append_taskObject_to_taskArray_detailed": [
                          "Succeeded"
                        ]
                      },
                      "metadata": {
                        "operationMetadataId": "491f333b-8f0c-4b00-90c9-f19c10948a6c"
                      },
                      "type": "Foreach",
                      "runtimeConfiguration": {
                        "concurrency": {
                          "repetitions": 50
                        }
                      }
                    },
                    "for_each_attachment": {
                      "foreach": "@outputs('Get_task_details')?['body/references']",
                      "actions": {
                        "append_to_attachmentArray": {
                          "runAfter": {},
                          "metadata": {
                            "operationMetadataId": "23902441-6a89-4dd2-b1eb-6b2068053cb7"
                          },
                          "type": "AppendToArrayVariable",
                          "inputs": {
                            "name": "attachmentArray",
                            "value": {
                              "createdByID": "@items('for_each_attachment')?['value/lastModifiedBy/user/id']",
                              "attachmentTitle": "@items('for_each_attachment')?['value/alias']",
                              "attachmentLink": "@items('for_each_attachment')?['resourceLink']",
                              "taskID": "@items('for_each_task')?['id']"
                            }
                          }
                        }
                      },
                      "runAfter": {
                        "for_each_checklist_item": [
                          "Succeeded"
                        ]
                      },
                      "metadata": {
                        "operationMetadataId": "1db02863-5401-4ccb-a543-835d83505745"
                      },
                      "type": "Foreach",
                      "runtimeConfiguration": {
                        "concurrency": {
                          "repetitions": 50
                        }
                      }
                    },
                    "append_taskObject_to_taskArray_detailed": {
                      "runAfter": {
                        "Get_task_details": [
                          "Succeeded"
                        ]
                      },
                      "metadata": {
                        "operationMetadataId": "8a962b72-e609-496a-b789-05e80f48f3ee"
                      },
                      "type": "AppendToArrayVariable",
                      "inputs": {
                        "name": "taskArray",
                        "value": {
                          "taskID": "@items('for_each_task')?['ID']",
                          "planID": "@items('for_each_task')?['planId']",
                          "bucketID": "@items('for_each_task')?['bucketId']",
                          "taskTitle": "@items('for_each_task')?['title']",
                          "planTitle": "@items('for_each_plan')?['PlanName']",
                          "bucketTitle": "@first(body('filter_bucket_array'))?['name']",
                          "taskDescription": "@outputs('Get_task_details')?['body/description']",
                          "startDate": "@items('for_each_task')?['startDateTime']",
                          "createdDate": "@items('for_each_task')?['createdDateTime']",
                          "dueDate": "@items('for_each_task')?['dueDateTime']",
                          "completeDate": "@items('for_each_task')?['completedDateTime']",
                          "createdByID": "@items('for_each_task')?['createdBy/user/id']",
                          "orderHint": "@items('for_each_task')?['orderHint']",
                          "pctComplete": "@items('for_each_task')?['percentComplete']"
                        }
                      }
                    }
                  },
                  "runAfter": {
                    "filter_bucket_array": [
                      "Succeeded"
                    ]
                  },
                  "else": {
                    "actions": {
                      "append_taskObject_to_taskArray": {
                        "runAfter": {},
                        "metadata": {
                          "operationMetadataId": "8a962b72-e609-496a-b789-05e80f48f3ee"
                        },
                        "type": "AppendToArrayVariable",
                        "inputs": {
                          "name": "taskArray",
                          "value": {
                            "taskID": "@items('for_each_task')?['ID']",
                            "planID": "@items('for_each_task')?['planId']",
                            "bucketID": "@items('for_each_task')?['bucketId']",
                            "taskTitle": "@items('for_each_task')?['title']",
                            "planTitle": "@items('for_each_plan')?['PlanName']",
                            "bucketTitle": "@first(\n    body('filter_bucket_array')\n    )?['name']",
                            "taskDescription": "",
                            "startDate": "@items('for_each_task')?['startDateTime']",
                            "createdDate": "@items('for_each_task')?['createdDateTime']",
                            "dueDate": "@items('for_each_task')?['dueDateTime']",
                            "completeDate": "@items('for_each_task')?['completedDateTime']",
                            "createdByID": "@items('for_each_task')?['createdBy/user/id']",
                            "orderHint": "@items('for_each_task')?['orderHint']",
                            "pctComplete": "@items('for_each_task')?['percentComplete']"
                          }
                        }
                      }
                    }
                  },
                  "expression": {
                    "or": [
                      {
                        "equals": [
                          "@items('for_each_task')?['hasDescription']",
                          true
                        ]
                      },
                      {
                        "greater": [
                          "@items('for_each_task')?['referenceCount']",
                          0
                        ]
                      },
                      {
                        "greater": [
                          "@items('for_each_task')?['checklistItemCount']",
                          0
                        ]
                      }
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "b8b0c2c6-d713-480e-9669-b824959b2c54"
                  },
                  "type": "If",
                  "description": "if a task has no description, checklist, or attachments, we don't need to run 'Get task details'"
                },
                "for_each_assignment": {
                  "foreach": "@items('for_each_task')?['_assignments']",
                  "actions": {
                    "append_to_assignmentArray": {
                      "runAfter": {},
                      "metadata": {
                        "operationMetadataId": "1d93b63a-b9e4-4c3b-8667-84e4845b2ed3"
                      },
                      "type": "AppendToArrayVariable",
                      "inputs": {
                        "name": "assignmentArray",
                        "value": {
                          "taskID": "@items('for_each_task')?['id']",
                          "assignedToID": "@items('for_each_assignment')?['userId']"
                        }
                      }
                    }
                  },
                  "runAfter": {
                    "is_more_task_detail_needed": [
                      "Succeeded"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "0aacd0d8-1c14-46c3-9ac5-80431deb8100"
                  },
                  "type": "Foreach",
                  "runtimeConfiguration": {
                    "concurrency": {
                      "repetitions": 50
                    }
                  }
                },
                "task_labels_object": {
                  "runAfter": {
                    "for_each_assignment": [
                      "Succeeded"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "1bd3d278-4616-405e-a76d-f9f1bb457947"
                  },
                  "type": "Compose",
                  "inputs": "@items('for_each_task')?['appliedCategories']"
                },
                "Parse_JSON_for_task_labels": {
                  "runAfter": {
                    "task_labels_object": [
                      "Succeeded"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "c7ce52b2-6115-4ad8-81f1-107cc7efed71"
                  },
                  "type": "ParseJson",
                  "inputs": {
                    "content": "@outputs('task_labels_object')",
                    "schema": {
                      "type": "object",
                      "properties": {
                        "category1": {
                          "type": "boolean"
                        },
                        "category2": {
                          "type": "boolean"
                        },
                        "category3": {
                          "type": "boolean"
                        },
                        "category4": {
                          "type": "boolean"
                        },
                        "category5": {
                          "type": "boolean"
                        },
                        "category6": {
                          "type": "boolean"
                        },
                        "category7": {
                          "type": "boolean"
                        },
                        "category8": {
                          "type": "boolean"
                        },
                        "category9": {
                          "type": "boolean"
                        },
                        "category10": {
                          "type": "boolean"
                        },
                        "category11": {
                          "type": "boolean"
                        },
                        "category12": {
                          "type": "boolean"
                        },
                        "category13": {
                          "type": "boolean"
                        },
                        "category14": {
                          "type": "boolean"
                        },
                        "category15": {
                          "type": "boolean"
                        },
                        "category16": {
                          "type": "boolean"
                        },
                        "category17": {
                          "type": "boolean"
                        },
                        "category18": {
                          "type": "boolean"
                        },
                        "category19": {
                          "type": "boolean"
                        },
                        "category20": {
                          "type": "boolean"
                        },
                        "category21": {
                          "type": "boolean"
                        },
                        "category22": {
                          "type": "boolean"
                        },
                        "category23": {
                          "type": "boolean"
                        },
                        "category24": {
                          "type": "boolean"
                        },
                        "category25": {
                          "type": "boolean"
                        }
                      }
                    }
                  }
                },
                "task_labels_array": {
                  "runAfter": {
                    "Parse_JSON_for_task_labels": [
                      "Succeeded"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "65d680cc-2ba7-4198-a3ef-007d5841dcf0"
                  },
                  "type": "Compose",
                  "inputs": [
                    {
                      "key": "category1",
                      "value": "@body('Parse_JSON_for_task_labels')?['category1']"
                    },
                    {
                      "key": "category2",
                      "value": "@body('Parse_JSON_for_task_labels')?['category2']"
                    },
                    {
                      "key": "category3",
                      "value": "@body('Parse_JSON_for_task_labels')?['category3']"
                    },
                    {
                      "key": "category4",
                      "value": "@body('Parse_JSON_for_task_labels')?['category4']"
                    },
                    {
                      "key": "category5",
                      "value": "@body('Parse_JSON_for_task_labels')?['category5']"
                    },
                    {
                      "key": "category6",
                      "value": "@body('Parse_JSON_for_task_labels')?['category6']"
                    },
                    {
                      "key": "category7",
                      "value": "@body('Parse_JSON_for_task_labels')?['category7']"
                    },
                    {
                      "key": "category8",
                      "value": "@body('Parse_JSON_for_task_labels')?['category8']"
                    },
                    {
                      "key": "category9",
                      "value": "@body('Parse_JSON_for_task_labels')?['category9']"
                    },
                    {
                      "key": "category10",
                      "value": "@body('Parse_JSON_for_task_labels')?['category10']"
                    },
                    {
                      "key": "category11",
                      "value": "@body('Parse_JSON_for_task_labels')?['category11']"
                    },
                    {
                      "key": "category12",
                      "value": "@body('Parse_JSON_for_task_labels')?['category12']"
                    },
                    {
                      "key": "category13",
                      "value": "@body('Parse_JSON_for_task_labels')?['category13']"
                    },
                    {
                      "key": "category14",
                      "value": "@body('Parse_JSON_for_task_labels')?['category14']"
                    },
                    {
                      "key": "category15",
                      "value": "@body('Parse_JSON_for_task_labels')?['category15']"
                    },
                    {
                      "key": "category16",
                      "value": "@body('Parse_JSON_for_task_labels')?['category16']"
                    },
                    {
                      "key": "category17",
                      "value": "@body('Parse_JSON_for_task_labels')?['category17']"
                    },
                    {
                      "key": "category18",
                      "value": "@body('Parse_JSON_for_task_labels')?['category18']"
                    },
                    {
                      "key": "category19",
                      "value": "@body('Parse_JSON_for_task_labels')?['category19']"
                    },
                    {
                      "key": "category20",
                      "value": "@body('Parse_JSON_for_task_labels')?['category20']"
                    },
                    {
                      "key": "category21",
                      "value": "@body('Parse_JSON_for_task_labels')?['category21']"
                    },
                    {
                      "key": "category22",
                      "value": "@body('Parse_JSON_for_task_labels')?['category22']"
                    },
                    {
                      "key": "category23",
                      "value": "@body('Parse_JSON_for_task_labels')?['category23']"
                    },
                    {
                      "key": "category24",
                      "value": "@body('Parse_JSON_for_task_labels')?['category24']"
                    },
                    {
                      "key": "category25",
                      "value": "@body('Parse_JSON_for_task_labels')?['category25']"
                    }
                  ]
                },
                "for_each_task_label": {
                  "foreach": "@body('filter_task_labels_array')",
                  "actions": {
                    "filter_plan_labels_with_task_labels": {
                      "runAfter": {},
                      "metadata": {
                        "operationMetadataId": "a9bad329-fa19-41ff-8533-527fdba98367"
                      },
                      "type": "Query",
                      "inputs": {
                        "from": "@outputs('plan_label_values_array')",
                        "where": "@equals(item()?['key'], items('for_each_task_label')?['key'])"
                      }
                    },
                    "append_to_labelArray": {
                      "runAfter": {
                        "filter_plan_labels_with_task_labels": [
                          "Succeeded"
                        ]
                      },
                      "metadata": {
                        "operationMetadataId": "4b7a974e-2343-4abc-b9fe-ca9ce06465fc"
                      },
                      "type": "AppendToArrayVariable",
                      "inputs": {
                        "name": "labelArray",
                        "value": {
                          "labelTitle": "@first(body('filter_plan_labels_with_task_labels'))?['value']",
                          "taskID": "@items('for_each_task')?['ID']",
                          "labelID": "@int(\r\n    substring(\r\n        first(body('filter_plan_labels_with_task_labels'))?['key'],\r\n        8\r\n    )\r\n)"
                        }
                      }
                    }
                  },
                  "runAfter": {
                    "filter_task_labels_array": [
                      "Succeeded"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "503bf544-b0e9-433c-82f5-12ffaedbc419"
                  },
                  "type": "Foreach",
                  "runtimeConfiguration": {
                    "concurrency": {
                      "repetitions": 50
                    }
                  }
                },
                "filter_task_labels_array": {
                  "runAfter": {
                    "task_labels_array": [
                      "Succeeded"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "ab454f83-d0f0-4956-9311-40840ab8ca9e"
                  },
                  "type": "Query",
                  "inputs": {
                    "from": "@outputs('task_labels_array')",
                    "where": "@equals(item()?['value'], true)"
                  }
                },
                "if_label_25_support_is_true": {
                  "actions": {
                    "if_task_not_complete": {
                      "actions": {
                        "link_to_this_task": {
                          "runAfter": {},
                          "metadata": {
                            "operationMetadataId": "559bc95a-b489-45af-8d41-5b6e96875fa7"
                          },
                          "type": "Compose",
                          "inputs": "<a href=\"https://tasks.office.com/***SCRUBBED***/Home/Task/@{outputs('Get_task_details')?['body/id']}\">@{items('for_each_task')?['title']}</a>"
                        },
                        "Post_message_in_a_chat_or_channel": {
                          "runAfter": {
                            "link_to_this_task": [
                              "Succeeded"
                            ]
                          },
                          "metadata": {
                            "operationMetadataId": "e6abb82c-7b65-42e2-99b6-19cc68093c5c"
                          },
                          "type": "OpenApiConnection",
                          "inputs": {
                            "host": {
                              "connectionName": "shared_teams_1",
                              "operationId": "PostMessageToConversation",
                              "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams"
                            },
                            "parameters": {
                              "poster": "Flow bot",
                              "location": "Channel",
                              "body/recipient/groupId": "***SCRUBBED***",
                              "body/recipient/channelId": "***SCRUBBED***",
                              "body/messageBody": "<p>There has been a request for help from a task on the @{items('for_each_plan')?['PlanName']} (@{items('for_each_plan')?['Team']}) board in Planner.<br>\n<br>\n<span style=\"font-size: 16px\"></span><span style=\"font-size: 16px\">@{outputs('link_to_this_task')}</span><span style=\"font-size: 16px\"></span></p>"
                            },
                            "authentication": "@parameters('$authentication')"
                          }
                        },
                        "log_help_request": {
                          "runAfter": {
                            "Post_message_in_a_chat_or_channel": [
                              "Succeeded"
                            ]
                          },
                          "metadata": {
                            "operationMetadataId": "57d990aa-2533-4875-a43a-032328619dca"
                          },
                          "type": "OpenApiConnection",
                          "inputs": {
                            "host": {
                              "connectionName": "shared_sql",
                              "operationId": "PostItem_V2",
                              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
                            },
                            "parameters": {
                              "server": "***SCRUBBED***",
                              "database": "***SCRUBBED***",
                              "table": "***SCRUBBED***",
                              "item/workflowType": "taskRetrieve",
                              "item/outcome": "Support Request",
                              "item/details": "Task: @{items('for_each_task')?['title']}\nPlan: @{items('for_each_plan')?['PlanName']}\nTeam: @{items('for_each_plan')?['Team']}",
                              "item/startTime": "@body('Start_time_PST')"
                            },
                            "authentication": "@parameters('$authentication')"
                          }
                        }
                      },
                      "runAfter": {},
                      "expression": {
                        "not": {
                          "equals": [
                            "@items('for_each_task')?['percentComplete']",
                            100
                          ]
                        }
                      },
                      "metadata": {
                        "operationMetadataId": "97078d0f-d82e-4382-9501-5f5da3d914d6"
                      },
                      "type": "If"
                    }
                  },
                  "runAfter": {
                    "for_each_task_label": [
                      "Succeeded"
                    ]
                  },
                  "expression": {
                    "equals": [
                      "@body('Parse_JSON_for_task_labels')?['category25']",
                      "@true"
                    ]
                  },
                  "metadata": {
                    "operationMetadataId": "d9694eb5-44b2-4dfc-87aa-445050326d1c"
                  },
                  "type": "If"
                }
              },
              "runAfter": {
                "List_tasks": [
                  "Succeeded"
                ]
              },
              "metadata": {
                "operationMetadataId": "c62e1965-b11e-4ba9-bd86-d226d0c6aa8b"
              },
              "type": "Foreach",
              "runtimeConfiguration": {
                "concurrency": {
                  "repetitions": 50
                }
              }
            },
            "plan_label_values_array": {
              "runAfter": {
                "Get_plan_details": [
                  "Succeeded"
                ]
              },
              "metadata": {
                "operationMetadataId": "6d9990c3-257a-44a3-b7dc-4a290f5d9bc5"
              },
              "type": "Compose",
              "inputs": [
                {
                  "key": "category1",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category1'],\r\n    'Pink'\r\n)"
                },
                {
                  "key": "category2",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category2'],\r\n    'Red'\r\n)"
                },
                {
                  "key": "category3",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category3'],\r\n    'Yellow'\r\n)"
                },
                {
                  "key": "category4",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category4'],\r\n    'Green'\r\n)"
                },
                {
                  "key": "category5",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category5'],\r\n    'Blue'\r\n)"
                },
                {
                  "key": "category6",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category6'],\r\n    'Purple'\r\n)"
                },
                {
                  "key": "category7",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category7'],\r\n    'Bronze'\r\n)"
                },
                {
                  "key": "category8",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category8'],\r\n    'Lime'\r\n)"
                },
                {
                  "key": "category9",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category9'],\r\n    'Aqua'\r\n)"
                },
                {
                  "key": "category10",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category10'],\r\n    'Gray'\r\n)"
                },
                {
                  "key": "category11",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category11'],\r\n    'Silver'\r\n)"
                },
                {
                  "key": "category12",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category12'],\r\n    'Brown'\r\n)"
                },
                {
                  "key": "category13",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category13'],\r\n    'Cranberry'\r\n)"
                },
                {
                  "key": "category14",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category14'],\r\n    'Orange'\r\n)"
                },
                {
                  "key": "category15",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category15'],\r\n    'Peach'\r\n)"
                },
                {
                  "key": "category16",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category16'],\r\n    'Marigold'\r\n)"
                },
                {
                  "key": "category17",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category17'],\r\n    'Light green'\r\n)"
                },
                {
                  "key": "category18",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category18'],\r\n    'Dark green'\r\n)"
                },
                {
                  "key": "category19",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category19'],\r\n    'Teal'\r\n)"
                },
                {
                  "key": "category20",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category20'],\r\n    'Light blue'\r\n)"
                },
                {
                  "key": "category21",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category21'],\r\n    'Dark blue'\r\n)"
                },
                {
                  "key": "category22",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category22'],\r\n    'Lavender'\r\n)"
                },
                {
                  "key": "category23",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category23'],\r\n    'Plum'\r\n)"
                },
                {
                  "key": "category24",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category24'],\r\n    'Light gray'\r\n)"
                },
                {
                  "key": "category25",
                  "value": "@coalesce(\r\n    outputs('Get_plan_details')?['body/categoryDescriptions/category25'],\r\n    'Dark gray'\r\n)"
                }
              ]
            },
            "Get_plan_details": {
              "runAfter": {
                "List_buckets": [
                  "Succeeded"
                ]
              },
              "metadata": {
                "operationMetadataId": "0dc5fdf6-2fa2-48ae-a4e5-020dec34cb88"
              },
              "type": "OpenApiConnection",
              "inputs": {
                "host": {
                  "connectionName": "shared_planner",
                  "operationId": "GetPlanDetails",
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_planner"
                },
                "parameters": {
                  "id": "@items('for_each_plan')?['PlanID']"
                },
                "authentication": "@parameters('$authentication')"
              }
            },
            "Append_to_errorString_2": {
              "runAfter": {
                "List_buckets": [
                  "Failed"
                ]
              },
              "metadata": {
                "operationMetadataId": "62a6b5ae-3e2e-490c-9f7e-932225197b4e"
              },
              "type": "AppendToStringVariable",
              "inputs": {
                "name": "errorString",
                "value": "Failed to get Planner data (buckets)\n@{items('for_each_plan')?['PlanName']} | @{items('for_each_plan')?['PlanID']} | @{items('for_each_plan')?['GroupID']} | @{items('for_each_plan')?['GroupName']} | @{items('for_each_plan')?['InsertedOn']} | @{items('for_each_plan')?['Team']} | @{items('for_each_plan')?['Leader']} | @{items('for_each_plan')?['LastRunDate']}<br>"
              },
              "description": "if service account does not have plan access, an error will generate"
            },
            "increment_errorCount_2": {
              "runAfter": {
                "Append_to_errorString_2": [
                  "Succeeded"
                ]
              },
              "metadata": {
                "operationMetadataId": "dab9642e-300b-4202-9be7-323e2f874e11"
              },
              "type": "IncrementVariable",
              "inputs": {
                "name": "errorCount",
                "value": 1
              }
            },
            "update_lastRunDate_on_source_row": {
              "runAfter": {
                "for_each_task": [
                  "Succeeded"
                ]
              },
              "metadata": {
                "operationMetadataId": "9288db35-967c-4ad8-8010-55fe7fef0234"
              },
              "type": "OpenApiConnection",
              "inputs": {
                "host": {
                  "connectionName": "shared_sql",
                  "operationId": "PatchItem_V2",
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
                },
                "parameters": {
                  "server": "@parameters('text_sqlServer (jake_sqlServer)')",
                  "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
                  "table": "***SCRUBBED***",
                  "id": "@items('for_each_plan')?['ID']",
                  "item/LastRunDate": "@body('Start_time_PST')"
                },
                "authentication": "@parameters('$authentication')"
              }
            }
          },
          "runAfter": {
            "get_active_plans": [
              "Succeeded"
            ]
          },
          "metadata": {
            "operationMetadataId": "4795519f-7313-40f8-9956-4b55cd3d8164"
          },
          "type": "Foreach",
          "runtimeConfiguration": {
            "concurrency": {
              "repetitions": 50
            }
          }
        },
        "get_active_plans": {
          "runAfter": {},
          "metadata": {
            "operationMetadataId": "41fba7a5-dec9-4cf3-b9c9-2ab3af9320f1"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_sql",
              "operationId": "GetItems_V2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
            },
            "parameters": {
              "server": "@parameters('text_sqlServer (jake_sqlServer)')",
              "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
              "table": "***SCRUBBED***",
              "$filter": "Active eq 1"
            },
            "authentication": "@parameters('$authentication')"
          },
          "description": "get active plans from SQL table (populated and managed by customer via powerApp)"
        },
        "increment_errorCount": {
          "runAfter": {
            "get_active_plans": [
              "Failed",
              "TimedOut"
            ]
          },
          "metadata": {
            "operationMetadataId": "5fe7081a-63d9-4a0c-86d9-23e0e70d964a"
          },
          "type": "IncrementVariable",
          "inputs": {
            "name": "errorCount",
            "value": 1
          }
        },
        "append_errorString": {
          "runAfter": {
            "increment_errorCount": [
              "Succeeded"
            ]
          },
          "metadata": {
            "operationMetadataId": "4331ac95-7f3e-447a-8a1f-c5510d0c2ca8"
          },
          "type": "AppendToStringVariable",
          "inputs": {
            "name": "errorString",
            "value": "Failed to get active plans<br>"
          }
        }
      },
      "runAfter": {
        "init_recordCounts": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "97ae9def-14eb-4ffe-95f7-2f5e5ec0b360"
      },
      "type": "Scope"
    },
    "check_for_email_notification_preference": {
      "actions": {
        "send_completion_email_lite": {
          "runAfter": {},
          "metadata": {
            "operationMetadataId": "b340dff7-7d1b-4081-ab75-dbe9dc7180d1"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_office365_1",
              "operationId": "SendEmailV2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365"
            },
            "parameters": {
              "emailMessage/To": "@parameters('text_emailRecipients (jake_emailRecipients)')",
              "emailMessage/Subject": "Task Aggegation Complete - @{div(sub(ticks(body('End_Time_PST')), ticks(body('Start_time_PST'))), 600000000)} minutes - @{variables('errorCount')} errors",
              "emailMessage/Body": "<p>@{variables('recordCounts')} </p>",
              "emailMessage/Importance": "Normal"
            },
            "authentication": "@parameters('$authentication')"
          }
        }
      },
      "runAfter": {
        "http_response": [
          "TimedOut"
        ]
      },
      "expression": {
        "equals": [
          "@parameters('bool_emailNotifications (jake_emailNotifications)')",
          true
        ]
      },
      "metadata": {
        "operationMetadataId": "05ae3102-3d82-424e-ad05-b2dfc05f743f"
      },
      "type": "If"
    },
    "init_errorCount": {
      "runAfter": {
        "init_errorString": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "31631ada-9b18-4f95-9e4e-660597cb8d1d"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "errorCount",
            "type": "integer",
            "value": 0
          }
        ]
      }
    },
    "Start_time_PST": {
      "runAfter": {},
      "metadata": {
        "operationMetadataId": "ccf5cf73-8dfa-4d3b-9b80-df852045c97e"
      },
      "type": "Expression",
      "kind": "ConvertTimeZone",
      "inputs": {
        "baseTime": "@{utcNow()}",
        "formatString": "o",
        "sourceTimeZone": "UTC",
        "destinationTimeZone": "Pacific Standard Time"
      }
    },
    "get_existing_SQL_records": {
      "actions": {
        "get_existing_tasks": {
          "runAfter": {},
          "metadata": {
            "operationMetadataId": "076b756a-9716-4a44-87ec-2582a1f0b7e8"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_sql",
              "operationId": "GetItems_V2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
            },
            "parameters": {
              "server": "@parameters('text_sqlServer (jake_sqlServer)')",
              "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
              "table": "***SCRUBBED***",
              "$filter": "isActive eq 1"
            },
            "authentication": "@parameters('$authentication')"
          },
          "runtimeConfiguration": {
            "paginationPolicy": {
              "minimumItemCount": 25000
            }
          }
        },
        "get_existing_assignments": {
          "runAfter": {
            "get_existing_tasks": [
              "Succeeded"
            ]
          },
          "metadata": {
            "operationMetadataId": "076b756a-9716-4a44-87ec-2582a1f0b7e8"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_sql",
              "operationId": "GetItems_V2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
            },
            "parameters": {
              "server": "@parameters('text_sqlServer (jake_sqlServer)')",
              "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
              "table": "***SCRUBBED***",
              "$filter": "isActive eq 1"
            },
            "authentication": "@parameters('$authentication')"
          },
          "runtimeConfiguration": {
            "paginationPolicy": {
              "minimumItemCount": 25000
            }
          }
        },
        "get_existing_attachments": {
          "runAfter": {
            "get_existing_assignments": [
              "Succeeded"
            ]
          },
          "metadata": {
            "operationMetadataId": "076b756a-9716-4a44-87ec-2582a1f0b7e8"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_sql",
              "operationId": "GetItems_V2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
            },
            "parameters": {
              "server": "@parameters('text_sqlServer (jake_sqlServer)')",
              "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
              "table": "***SCRUBBED***",
              "$filter": "isActive eq 1"
            },
            "authentication": "@parameters('$authentication')"
          },
          "runtimeConfiguration": {
            "paginationPolicy": {
              "minimumItemCount": 25000
            }
          }
        },
        "get_existing_checklist_items": {
          "runAfter": {
            "get_existing_attachments": [
              "Succeeded"
            ]
          },
          "metadata": {
            "operationMetadataId": "076b756a-9716-4a44-87ec-2582a1f0b7e8"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_sql",
              "operationId": "GetItems_V2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
            },
            "parameters": {
              "server": "@parameters('text_sqlServer (jake_sqlServer)')",
              "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
              "table": "***SCRUBBED***",
              "$filter": "isActive eq 1"
            },
            "authentication": "@parameters('$authentication')"
          },
          "runtimeConfiguration": {
            "paginationPolicy": {
              "minimumItemCount": 25000
            }
          }
        },
        "get_existing_labels": {
          "runAfter": {
            "get_existing_checklist_items": [
              "Succeeded"
            ]
          },
          "metadata": {
            "operationMetadataId": "076b756a-9716-4a44-87ec-2582a1f0b7e8"
          },
          "type": "OpenApiConnection",
          "inputs": {
            "host": {
              "connectionName": "shared_sql",
              "operationId": "GetItems_V2",
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
            },
            "parameters": {
              "server": "@parameters('text_sqlServer (jake_sqlServer)')",
              "database": "@parameters('text_sqlDatabase (jake_sqlTable)')",
              "table": "***SCRUBBED***",
              "$filter": "isActive eq 1"
            },
            "authentication": "@parameters('$authentication')"
          },
          "runtimeConfiguration": {
            "paginationPolicy": {
              "minimumItemCount": 25000
            }
          }
        }
      },
      "runAfter": {
        "init_checklistArray": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "0a15e51f-ea62-4662-9ec6-64805e8c3d3e"
      },
      "type": "Scope"
    },
    "init_recordCounts": {
      "runAfter": {
        "get_existing_SQL_records": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "18ee4e3e-b4c7-4071-97b4-0bed010e698b"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "recordCounts",
            "type": "string",
            "value": "<h3>EXISTING SQL RECORDS</h3>\n<ul>\n<li>Tasks: @{length( outputs('get_existing_tasks')?['body/value'] )}</li>\n<li>Assignments: @{length( outputs('get_existing_assignments')?['body/value'] )}</li>\n<li>Attachments: @{length( outputs('get_existing_attachments')?['body/value'] )}</li>\n<li>Checklist Items: @{length( outputs('get_existing_checklist_items')?['body/value'] )}</li>\n<li>Labels: @{length( outputs('get_existing_labels')?['body/value'] )}</li>\n</ul>"
          }
        ]
      }
    },
    "init_outcomeArray": {
      "runAfter": {
        "init_inactiveChecklist_array": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "ac0964c2-d3e3-4baa-a5a7-d611751a2567"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "outcomeArray",
            "type": "array"
          }
        ]
      }
    },
    "init_inactiveTasks_array": {
      "runAfter": {
        "if_weekday": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "81057d90-262d-409e-87fc-5d3ddaf9d56f"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "inactiveTasks",
            "type": "array"
          }
        ]
      }
    },
    "init_inactiveLabels_array": {
      "runAfter": {
        "init_inactiveAssignments_array": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "2b6ede09-4521-4637-b8cd-559e7d298517"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "inactiveLabels",
            "type": "array"
          }
        ]
      }
    },
    "init_inactiveAttachments_array": {
      "runAfter": {
        "init_inactiveTasks_array": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "822732d7-f448-4df5-a865-e10ca2898905"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "inactiveAttachments",
            "type": "array"
          }
        ]
      }
    },
    "init_inactiveAssignments_array": {
      "runAfter": {
        "init_inactiveAttachments_array": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "0e5642fd-169e-4d2f-acc7-36fc12415e49"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "inactiveAssignments",
            "type": "array"
          }
        ]
      }
    },
    "init_inactiveChecklist_array": {
      "runAfter": {
        "init_inactiveLabels_array": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "5bb90e84-300c-4a53-a9c4-a2440bc7452c"
      },
      "type": "InitializeVariable",
      "inputs": {
        "variables": [
          {
            "name": "inactiveChecklist",
            "type": "array"
          }
        ]
      }
    },
    "HTTP": {
      "runAfter": {
        "array_payloads": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "062aff9d-f992-4b7d-b23a-3bd6cdfc8ee1"
      },
      "type": "Http",
      "inputs": {
        "method": "POST",
        "uri": "@parameters('text_processFlowEndpoint (jake_test_postURL)')",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": "@outputs('array_payloads')"
      },
      "operationOptions": "DisableAsyncPattern"
    },
    "http_response": {
      "runAfter": {
        "HTTP": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "fbe15233-84cb-4618-91cb-c350236a79ec"
      },
      "type": "Compose",
      "inputs": "@{outputs('HTTP')['statusCode']}\n@{body('HTTP')}"
    },
    "End_Time_PST": {
      "runAfter": {
        "Plan_and_task_iteration": [
          "Succeeded",
          "Failed"
        ]
      },
      "metadata": {
        "operationMetadataId": "844d3531-4728-412e-855f-690ba1bd04f5"
      },
      "type": "Expression",
      "kind": "ConvertTimeZone",
      "inputs": {
        "baseTime": "@{utcNow()}",
        "formatString": "o",
        "sourceTimeZone": "UTC",
        "destinationTimeZone": "Pacific Standard Time"
      }
    },
    "compose_totals": {
      "runAfter": {
        "End_Time_PST": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "33b358a6-a5d6-4f25-aef4-4714c2e8e736"
      },
      "type": "Compose",
      "inputs": "<h2>Gather Task Data</h2>\n<h3>@{length(\noutputs('get_active_plans')?['body/value']\n)} Plans</h3>\n<ul><li>Tasks: @{length(\n    variables('taskArray')\n)}</li></ul>\n<ul><li>Assignments: @{length(\n    variables('assignmentArray')\n)}</li></ul>\n<ul><li>Attachments: @{length(\n    variables('attachmentArray')\n)}</li></ul>\n<ul><li>Checklist items: @{length(\n    variables('checklistArray')\n)}</li></ul>\n<ul><li>Labels: @{length(variables('labelArray'))}</li></ul>\n\n<h3>ERROR COUNT</h3>\n<p>@{variables('errorCount')}</p>\n\n<h3>DEBUG LOG</h3>\n<p>@{variables('errorString')}</p>"
    },
    "array_payloads": {
      "runAfter": {
        "compose_totals": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "b459749d-99fd-45a0-90b5-91fa046a813a"
      },
      "type": "Compose",
      "inputs": {
        "parentFlowDuration": "@div(sub(ticks(body('End_Time_PST')), ticks(body('Start_time_PST'))), 600000000)",
        "parentFlowMessage": "@{outputs('compose_totals')}",
        "payloads": [
          {
            "contentType": "task",
            "values": {
              "arraySQL": "@outputs('get_existing_tasks')?['body/value']",
              "arrayCurrent": "@variables('taskArray')"
            }
          },
          {
            "contentType": "checklist",
            "values": {
              "arraySQL": "@outputs('get_existing_checklist_items')?['body/value']",
              "arrayCurrent": "@variables('checklistArray')"
            }
          },
          {
            "contentType": "label",
            "values": {
              "arraySQL": "@outputs('get_existing_labels')?['body/value']",
              "arrayCurrent": "@variables('labelArray')"
            }
          },
          {
            "contentType": "attachment",
            "values": {
              "arraySQL": "@outputs('get_existing_attachments')?['body/value']",
              "arrayCurrent": "@variables('attachmentArray')"
            }
          },
          {
            "contentType": "assignment",
            "values": {
              "arraySQL": "@outputs('get_existing_assignments')?['body/value']",
              "arrayCurrent": "@variables('assignmentArray')"
            }
          }
        ]
      }
    },
    "log_outcome": {
      "runAfter": {
        "http_response": [
          "Succeeded"
        ]
      },
      "metadata": {
        "operationMetadataId": "57d990aa-2533-4875-a43a-032328619dca"
      },
      "type": "OpenApiConnection",
      "inputs": {
        "host": {
          "connectionName": "shared_sql",
          "operationId": "PostItem_V2",
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_sql"
        },
        "parameters": {
          "server": "***SCRUBBED***",
          "database": "***SCRUBBED***",
          "table": "***SCRUBBED***",
          "item/workflowType": "taskRetrieve",
          "item/outcome": "@if(\r\n    greater(\r\n        variables('errorCount'),\r\n        0\r\n    ),\r\n    'Complete with Error',\r\n    'Success'\r\n)",
          "item/itemCount": "@add(\r\n    add(\r\n        length(variables('assignmentArray')),\r\n        length(variables('checklistArray'))\r\n    ),\r\n    add(\r\n        add(\r\n            length(variables('labelArray')),\r\n            length(outputs('get_active_plans')?['body/value'])\r\n        ),\r\n        add(\r\n            length(variables('taskArray')),\r\n            length(variables('attachmentArray'))\r\n        )\r\n    )\r\n)",
          "item/errorCount": "@variables('errorCount')",
          "item/details": "Start: @{body('Start_time_PST')} | Runtime: @{div(\r\n    sub(\r\n        ticks(body('End_Time_PST')),\r\n        ticks(body('Start_time_PST'))\r\n    ),\r\n    600000000\r\n)\r\n\r\n} minutes\n@{length(outputs('get_active_plans')?['body/value'])} pln | @{length(variables('assignmentArray'))} asn | @{length(variables('attachmentArray'))} atc | @{length(variables('checklistArray'))} chk | @{length(variables('labelArray'))} lbl | @{length(variables('taskArray'))}tsk@{variables('errorString')}",
          "item/startTime": "@body('Start_time_PST')"
        },
        "authentication": "@parameters('$authentication')"
      }
    },
    "if_weekday": {
      "actions": {},
      "runAfter": {
        "Start_time_PST": [
          "Succeeded"
        ]
      },
      "else": {
        "actions": {
          "Terminate": {
            "runAfter": {},
            "metadata": {
              "operationMetadataId": "2309e5bb-9878-4721-8a1b-273d5e86b574"
            },
            "type": "Terminate",
            "inputs": {
              "runStatus": "Cancelled"
            }
          }
        }
      },
      "expression": {
        "and": [
          {
            "less": [
              "@dayOfWeek(body('Start_time_PST'))",
              6
            ]
          },
          {
            "greater": [
              "@dayOfWeek(body('Start_time_PST'))",
              0
            ]
          }
        ]
      },
      "metadata": {
        "operationMetadataId": "9fafc0e2-9c45-40f1-b824-ee5cf3561784"
      },
      "type": "If"
    }
  },
  "description": "Fetches all active plans and enumerates tasks, assignments, checklists, assignees, and labels. Aggregates those data points and gets them into Azure SQL."
}
```
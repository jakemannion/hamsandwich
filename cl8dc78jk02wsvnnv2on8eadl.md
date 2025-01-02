---
title: "Extract Excel Worksheet Data Without Tables - Power Automate"
seoTitle: "Power Automate: Extract Excel Data Without Tables"
seoDescription: "Learn to extract unstructured Excel data using Power Automate. Follow the guide to transform and process data effectively"
datePublished: Thu Sep 22 2022 17:37:38 GMT+0000 (Coordinated Universal Time)
cuid: cl8dc78jk02wsvnnv2on8eadl
slug: extract-excel-worksheet-data-without-tables-power-automate
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1663722716067/sC8JvaU5a.jpg
tags: data, table, excel, powerplatform, automate

---

# Overview

One of the more common uses for [Power Automate](https://powerautomate.microsoft.com/en-us/) is working with \*\*Excel files and data\*\*. Overall compatibility is pretty good, with one significant exception.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1663786634965/jSAKMcyb4.png align="left")

| Data Location | Power Automate Compatibility |
| --- | --- |
| Excel table | Good functionality, with some [thresholds and limitations](https://learn.microsoft.com/en-us/connectors/excelonlinebusiness/#known-issues-and-limitations-with-actions) |
| Excel table (with key column) | Same as above, plus additional OData capabilities |
| Excel CSV | Supported, but not through the [Excel Online connector](https://learn.microsoft.com/en-us/connectors/excelonlinebusiness/) |
| Excel worksheet | Data operations not supported |

Historically, that last item creates a lot of frustration. 😐 Excel is used extensively in the business world and this limitation puts its *most commonly-used format* outside the reach of Microsoft's own automation platform.

### Unstructured Data - Example

The data in this worksheet *looks* like a table to you and me, but without a formal definition, Automate can't work with the information here.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663715614372/VtT7UFgQM.png align="center")

It needs *explicit* structure before column relationships and the corresponding data inside the Excel document can be utilized.

# The Scenario

You might not use Automate to work with Excel very often[<sup>1</sup>](#heading-thoughts-on-excel-as-a-data-source), but like most juggernauts, it's sometime unavoidable. Here is a situation that presented itself recently:

1. A partner organization emails a collection of Excel worksheets every morning
    
2. A flow stores them in a SharePoint Online document library
    
3. The worksheets have consistent structures but the data is not contained within tables
    
4. The data needs to reach Azure SQL each day without manual intervention
    

# The Solution

## Preparing For Multiple Files/Formats

Our scenario involves multiple worksheets, each with a unique data structure. I used an array variable to store configuration details at the beginning of the flow:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663813947635/gO2Ypc4F0.png align="left")

> Note: If you're working with *one* Excel file, feel free to skip to the next section

Each object contains information about a worksheet:

Name | Example | Usage --- | --- filename | incomingWorksheet2.xlsx | Name of the Excel worksheet colWidth | BA | Identifier for the last expected column rowLength | 2500 | Number of table rows to create (see notes on "padding" in next section) colHeadings | CustomerName;Item;Price | *OPTIONAL* - column heading overrides tableName | Sales | SQL table where data will be stored. Also used to name each Excel table created. keyColumn | saleID | Column used to filter results - use one that has a value in every row

Next, we will loop over the array, and the real party can begin:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663717233734/MNBWFKP8J.png align="left")

> Note: Pre-baked arrays are okay when quickly proving out a new method. For an actual solution, environment variables are a better choice.

## Adding Tables into Excel Worksheets with Existing Data

The Excel connector within Power Automate has a [Create Table](https://learn.microsoft.com/en-us/connectors/excelonlinebusiness/#create-table) action. You can use this action to insert a table into an Excel worksheet that already contains data. I was very surprised to see this work!

We're not out of the woods yet, though. The amount of worksheet data fluctuates each day in our scenario, which is a problem because Automate can't "see" how much data exists at each runtime.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663742393608/bLg-9ZFKr.png align="center")

We can mitigate some of the uncertainty by "padding" the [Create Table](https://learn.microsoft.com/en-us/connectors/excelonlinebusiness/#create-table) action. For example, if you expect ~2,000 rows of data in a worksheet, have the flow create 2,500-3,000 rows when inserting the table.

Without surplus rows, you risk isolating data. Automate can't create tables twice in the same worksheet, so options for self-correction are slim if you undershoot the mark. Aim high, but don't go *too* far overboard!

> Note: This "padding" technique only works if you can make solid predictions about the expected data

Here is the **Create Table** action, using dynamic values from the configuration array:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663718102271/TCdrcr03n.png align="left")

> Note: Use fixed values for the parameters if you're operating on a single worksheet

If your setup is correct, the existing Excel data will remain in place while the new table is inserted. The action will continue until it hits the row limit, adding blank rows to the worksheet once it runs past the data.

## Getting Data Back from Excel

Use **List Table Rows** to pull the data into Automate. Provide an OData filter query (under advanced options) to retrieve non-blank rows only:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663814263174/v4t4gZcmT.png align="left")

```plaintext
items('for_each_Excel_document')?['keyColumn']
```

Unless you are dealing with very small data sets, don't forget to enable pagination for the **List Table Rows** action. This will allow the flow to pull *all* the data back from Excel in successive chunks, then proceed once complete.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663835875635/GKrVcwSWk.png align="left")

Set the threshold equal to the highest row count you'll be using in the **Create Table** action(s).

## Do Something With the Results

If you've made it this far, the possibilities are wide open! Extracting unstructured data from Excel worksheets using Power Automate is no small feat.

### Possibilities From Here

* Loop over each row in the result set and store the data somewhere (SQL, SharePoint, Dataverse, etc.) line-by-line
    
* Use a switch statement to handle the results from unique worksheets in different ways
    
* Pass the result set into a SQL table (1,000 rows at a time) using the **Execute a SQL query** action
    

# Thoughts on Excel as a Data Source

Excel is often a popular data source for users getting started with the Power Platform. It's a familiar, flexible and portable format.

I don't typically recommend it - there are more performant and less volatile options available. Sometimes you don't have a choice, so it's good to have a few tricks up your sleeve.

# Disclosure and Final Thought

There are [better, far more elegant ways to solve this problem](https://sharepains.com/2018/10/17/microsoft-flow-read-large-excel-files-within-seconds-without-creating-tables-using-microsoft-graph/) than the approach described on this page. What's outlined above isn't a foolproof method. It's not designed for heavy usage across a broad spectrum of enterprise-grade scenarios.

But in a pinch, and for the right use cases, this method should get the job done. I hope this comes in handy for individuals who don't necessarily have the means, expertise or time to handle app registration, HTTP calls, Graph, and so forth. As you continue your journey along the Power Platform progression arc, I hope you learn those methods, too.
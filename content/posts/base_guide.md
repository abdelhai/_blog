---
title: "Introducting Guide: A GUI for your Deta Base"
date: 2020-09-30
draft: false
---

## What's a Deta Base Guide?

Base Guide (beta) is a graphical user interface for Deta Base. Base Guide was inspired by the struggles of many developers in interacting with the data powering their apps, *outside the confines* *of the app  itself.*

A video tour is available [at the end of the post](#tour).

## **Motivation**

Since shipping [the earliest alpha of Deta & Deta Base](https://youtu.be/jHYFrS0LVY8?t=110), devs have resonated with the simplicity of spinning up a database using Deta. In it's purest form, there is no marginal config step required, it's just code.

This approach is great when you want to design state from the comforts of your text editor. While Base has provided a persistent state solution for some inspirational apps so far, we realized it was falling short on another dimension we didn't initially expect.  

The biggest shortcoming we've heard from users about Base is the lack of a UI. We often heard '*I want to be able to easily see and interact with the data my app generates*'.  We noticed some users would hack around this shortcoming, adding auxiliary logic to their application or a side script to quickly inspect or modify the records their main app generates. 

But going through both a text editor and running a script to inspect and modify records is less than ideal. Two members of our community— turned summer fellows—both took it a step further, each hacking their own Base records viewers together ([first](https://explorer.deta.dev) w/ [source](https://github.com/fillerInk/deta-base-explorer); [second](https://base-ui.jajoo.fun/) w/ [source](https://github.com/jajoosam/base-ui)). 

Suffice to say, we are extremely excited to address this need and launch Deta's official Base GUI solution into beta, building off the work of the second solution. With Base Guide, you can rapidly inspect, add, delete, and modify the records contained in your Base.

### Inspecting Records

To view a Base's records, all you need to do is open an individual Base from the Deta Project's Sidebar. By default the Base's records will load into Base Guide.


![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ygo4vp291q1zz9g9bu6u.png)

If you want to inspect a slice of the data within a Base, you can use [Base's queries](https://docs.deta.sh/docs/base/sdk#queries) to do so, i.e.:

```js
[{"cuisine": "Italian", "have_visited": false}]
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/qv2mz4uniqjlv0fcu0ky.png)


### Adding Records

To add records to a base, click the '**+ Add**' button and a new row will appear at the top of your Base's UI.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/n184adeun585uvpr9k27.png)

The new row will be treated as a candidate edit, with a yellow background, which can be permanently saved by clicking the '**Save**' button.

### Modifying Records

For primitive types (strings, booleans, and numbers), you can modify any record directly from within a cell.

For advanced editing, i.e. to change the type of a given value, or edit arrays or objects, you can expand the cell and do so.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pom71tzotlie0s865n7i.png)

Once you are satisfied with your edits, simply click the '**Save Edits**' button, as before, to modify these records in your Base. If you want to discard all your edits locally, click '**Undo Changes**'.


### Deleting Records

For every record displayed in Base UI, there is a checkbox on the left side. By clicking the checkbox, the record will be highlighted in red as a delete candidate. To go forward with the deletion, simply click the '**Delete**' item.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/sootikirv2s5cmz0xj3o.png)

For candidate adds, edits, and deletes, the '**Undo**' button will revert the Base Guide to the state that was last pulled.


### Start with Base Guide Today

To get started with Base Guide, simply [log-in / sign-up](https://web.deta.sh/) with Deta, grab a project key, and create a [Deta Base](https://docs.deta.sh/docs/base/sdk) in your Python or Node.js app. Deta Guide will be accessible for any Base you create.

### Tour

{{< youtube y11dOkP8ZTI >}}


### Thanks

We'd love to thank the Deta community for the incredible feedback that led to this initiative as well as Samarth Jajoo for the initial implementation which we built on top of!
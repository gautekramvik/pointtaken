---
title: "Updating Single and Multi Value Taxonomy Fields Using Pnp Js Core"
date: 2017-05-16T20:52:50+01:00
draft: false
author: "Øyvind Aas-Mehren"
categories: ["dev"]
tags: ["promo"]
banner: "img/code.png"
---

# Updating single and multi value taxonomy fields using PnP-JS-Core
<br>
#### Point Taken is the leading SharePoint, Office 365 and Nintex consultancy in Norway.
<br>
All the devs in Point Taken are heavy users of the awesome repos provided on https://github.com/SharePoint. But even though our SharePoint dev lives have become easier we still encounter challenges and general SharePoint weirdness. This is one of those cases.

A little background: one of our clients wanted an application that would copy predefined tasks into project sites based on the project type. Some of these columns were taxonomy fields, both single and multi value. Updating term values in taxonomy fields using PnP-JS-Core (and REST in general) proved to be a bit of a struggle, and here are our discoveries.

Single value term fields can be updated using the code provided in this issue on GitHub
<br>
<br>
#### A single value taxonomy field is updated like this:


        const listWeWantToUpdate = pnp.sp.web.lists.getByTitle('ListWeWantToUpdateTitle');

        listWeWantToUpdate.getListItemEntityTypeFullName()
            .then((entityTypeFullName) => {

                const updateObject = {
                    Title: 'Item title', // Only included as an example
                    SomeSingleValueTaxonomyField: {
                        __metadata: { type: 'SP.Taxonomy.TaxonomyFieldValue' },
                        Label: 'LabelOfTerm', // field label - you can also use the Id returned in rest calls here
                        TermGuid: '7dc9d5f8-16c2-48f2-88f7-90db39c7afb7', // field guid
                        WssId: -1 // fake
                    }
                };

                listWeWantToUpdate.items.add(updateObject, entityTypeFullName)
                    .then((updateResult) => {
                        console.dir(updateResult);
                    })
                    .catch((updateError) => {
                        console.dir(updateError);
                    });
            });
 

But what about fields with multiple values?

Based on the replies in the issue mentioned JSOM is the way to go, but where’s the fun in that? After trying the obvious variations of the code above it was time to turn to Google.

A bit of digging brought up blog posts by Jason Lee and Beau Cameron who had already cracked this nut in REST – the answer: use the InternalName of the hidden note field. 

So first we need to get the corresponding note field and then we need to write a term string to it. Now we only had to figure out how to make this work in PnP-JS-Core.
<br>
<br>
#### A multi value taxonomyfield is updated like this:

        const listWeWantToUpdateMulti = pnp.sp.web.lists.getByTitle('ListWeWantToUpdateTitle');

        // Example of a term string.
        // You can also use the Id returned in rest calls instead of the full label
        const termString = '-1;#SomeTerm|02ee415b-99c4-448b-8727-7daa2a4a281;#-1;# SomeOtherTerm |0e2f40d9-09ac-406e-b102-630e8dadade6;';

        // If the name of your taxonomy field is SomeMultiValueTaxonomyField, the name of your note field will be SomeMultiValueTaxonomyField_0
        const multiTermNoteFieldName = 'SomeMultiValueTaxonomyField_0';

        listWeWantToUpdateMulti.getListItemEntityTypeFullName()
            .then((entityTypeFullName) => {
                listWeWantToUpdateMulti.fields.getByTitle(multiTermNoteFieldName).get()
                    .then((taxNoteField) => {
                        const multiTermNoteField = taxNoteField.InternalName;
                        const updateObject = {
                            Title: 'Item title', // Only included as an example
                        };
                        updateObject[multiTermNoteField] = termString;

                        listWeWantToUpdateMulti.items.add(updateObject, entityTypeFullName)
                            .then((updateResult) => {
                                console.dir(updateResult);
                            })
                            .catch((updateError) => {
                                console.dir(updateError);
                            });
                    });
            });

And there you go. Hope this information was useful to you

<i>Update May 31st 2017:</br>
A friendly Belgian reader pointed out that the code for multi value fields was missing an important step. The term string should be inserted into the corresponding note field and not the taxonomy field itself. The code has been updated to reflect this important detail.</i>

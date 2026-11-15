---
title: "Layering Clothing"
icon: "🧅"
created: 2024-12-20
updated: 2026-11-15
---

# Layering Clothing

Make and test clothing against the Human model. The older illustrations and recordings below show the retired Citizen model; they illustrate layering principles, but use the Human assets and paths described in the text.

## **Layering of Shirts, Jackets, Trousers, Shoes and Gloves**


In S&box, we want a library of clothing that all fit together, resulting in hundreds, thousands of outfit combinations.

![Layering clothing system](./images/layering.gif)

If you're making a clothing piece, you'll need to consider what fits under or over it.  


---


## Simple Breakdown

![](./images/simple-breakdown.png)

* A 2 layer system for the torso, shirts **(layer 1)** and jackets / vests / etc **(layer 2)**. 

  \
* Trousers are always tucked underneath the shirts.

  \
* Shoes are always tucked underneath trousers.

  \
* Gloves can be above or under shirts **(layer 1)** but never over / clipping into jackets **(layer 2)**.

  \
* If you're making an asset like a jumpsuit, that fills both the torso **(layer 1)** and the trousers area, this is fine. Though you will still need to make sure it fits underneath jackets **(layer 2)**.


* Full outfits are an exception to the rule, since they won't be fitting under or over any other clothing.


* If you are making a coat / vest / armour asset that doesn't show any of layer 1, for example a zipless Hoodie, you won't need to worry about it following the layering rule.


---


## Checking your Asset


A great way of making sure your clothing is fitting with the existing clothing, In your S&box project, place your clothing in a scene and add other existing clothing to check for clipping issues.


Right click in your '**Hierarchy**', add an empty asset. Clicking on this new asset, we can 'Add Component' in the Inspector. Type and click in Model Renderer. ![](./images/checking-your-asset-1.png)


Click on the 'Model' input to open the asset picker. Enable **Show Base Content** to find the supplied Human model.

![](./images/checking-your-asset-3.png)


Choose `models/citizen_human/human.vmdl`. Now we have the Human in our scene to start dressing and testing our clothing.

![](./images/checking-your-asset-2.png)

![](./images/checking-your-asset.png)


Now we can start adding clothing to test. The supplied clothing is under `citizen\assets\models\citizen_clothes`. Use the VMDLs assigned to each clothing definition's `Model` or `Female Model` field so you are comparing the correct fits.

![](./images/now-we-can-start-adding-clothing-to-test-head-over-to-citi.png) 


In this example I will be testing the Polo Shirt, for any clipping issues. Which should sit under all jackets / vests / armour, etc.

![](./images/in-this-example-i-will-be-testing-the-polo-shirt-for-any-cl.png)


Add another component and add a layer 2 clothing asset. In this example I'm adding the Bomber jacket.

![](./images/add-another-component-and-add-a-layer-2-clothing-asset-in-t.png)


I can switch out the model input from the Bomber Jacket and try on other jackets.

![](./images/i-can-switch-out-the-model-input-from-the-bomber-jacket-and.png)



:::info
Try out a few jackets till you find any clipping with your shirt, or until you feel confident there is no obvious clipping.

:::


---


## Including Visible Skin 👕🔪


![](./images/cutting-the-citizen-1.png)


For layer 1 clothing that replaces a bodygroup, include the skin that should remain visible as part of your clothing model. Extract the needed geometry from `citizen_human_male_REF.fbx` or `citizen_human_female_REF.fbx`, matching the body your clothing is fitted to.


![](./images/cutting-the-citizen.png)


This is useful for assets that change the silhouette of the Human body: hide the covered bodygroup and replace the exposed skin with the cut-out geometry in your clothing model. Check both body fits so the skin meets the clothing without gaps.


---


## Gloves Layering 🥊


![](./images/gloves-layering.png)


Gloves should sit **over** or **under** the Layer 1 clothing. They should never be any larger or clip with the Layer 2 Jackets, for the sake of preventing clipping. 


---


## Shoes Layering 👟


![](./images/shoes-layering-1.png)


By default, most shoes should fit under all trousers. Which in most cases should be fine, keeping in mind to keep the trousers relatively wide, leaving for enough space for the shoes. 


![](./images/shoes-layering.png)

Though in some cases such as the `football_shoes` which have shinpads, reaching further up the leg, clipping is inevitable.


Considering the use-cases of each clothing item, users will more likely use the `football_shorts` which fit.



:::info
Clipping is inevitable, just keep a close eye on your silhouette of your assets and clipping shouldn't be too egregious. 

:::


---


## Full-Body Outfits

![](./images/full-body-outfits.png)


:::info
Full-body outfits can hide the Human bodygroups and replace the visible character while still using `.clothing` items and the same clothing logic.

:::

Full-Body Outfits are unique compared to the standard clothing and their layering system; they are bespoke and cover a majority or all of the body and would not fit with most clothing. 


**Context is Key**

For outfits like the `Pirate_Outfit`, it is a unique asset with a defined use-case, we treat it as it's own full outfit piece rather than breaking it up into modular layered pieces. 

Another example is the Sci-fi Armour, it covers the whole whole body and has a specific use-case, so would not need to mix or mash with other clothing assets, plus there's no good use-case for a sci-fi armoured character to wear a raincoat. 


For Full-Body Outfits that completely fill the character, the clothing takes up both Layer 1 and Layer 2, since it filling both spaces.



:::info
You definitely want to distinguish: is your clothing a modular / layer friendly asset, or if it's a unique full body outfit that isn't going to fitting under or over other clothing pieces.

:::

---
title: "Morphing to Humans"
icon: "🧍"
created: 2025-01-06
updated: 2026-10-23
---

# Morphing to Humans

Human is now the default character. This guide is for converting existing clothing made for the retired Citizen model. For new clothing, start directly with the Human reference meshes in [First Time Setup](/assets/clothing/first-time-setup.md); skip the conversion steps below. The [female fit guidance](#female-human-version) also applies to new clothing.

## **Morphing your Clothing to Humans**

Dependant on how complex / simple your model is, morphing to the human mesh can be very straightforward. 


:::info
The example shown below is a quickly done morph of a clothing piece we've found as one of the hardest clothing pieces to get right on the human. The lack of shoulders, multiple details and pieces that distort very easily. 

:::


---


## **Important Files & Setup**

To fit existing clothing to the Human body, you'll need its reference FBX file.

Go to - `sbox\addons\citizen\Assets\models\citizen_human`

And grab `citizen_human_male_REF.fbx`.

![](./images/important-files-setup.png)

This reference supplies the Human mesh, rig, and skinning.


---


## Moving Major Shapes

Firstly, we want to bring the neck loosely up to the same position of the human neck. As well as moving the wrists into position. No need to rotate or resize anything yet.

![](./images/moving-major-shapes-1.png)


![](./images/moving-major-shapes.png)


:::info
We use soft select for smooth results when moving around the mesh, which is a feature in most 3D software. 

:::


Turning **Connect Only** on when using Soft select, will allow you to adjust the arms without effecting the nearest topology, keeping the chest area from being morphed.


This works great when getting the arms quickly morphed without effecting the chest.


!['Connect Only' turned ON](./images/connect-only-turned-on.gif)

!['Connect Only' turned OFF](./images/connect-only-turned-off.gif)


:::info
Any details like buttons, separate meshes, keep removed while morphing the main body of the clothing, as to keep from distorting the buttons. Later you can place them back on.

:::


---


## **Moving Details** 


![](./images/moving-details.png)

Moving the neck closer to the body, adjusting the pockets, this stage I'm focusing on the secondary shapes and details to fit them correctly onto the human body.



:::info
Any morphing done will cause stretching and possible distortion of certain details, at this stage, we spent the time to fix and adjust these distorted area, till we are happy with the results.

:::


---


## **Adjustments and Fixes**

![](./images/adjustments-and-fixes-1.png)

![](./images/adjustments-and-fixes.png)When morphing clothing, you'll find areas that are so stretched that you'll need to **add more edge loops** to even it out.


In this example shirt, you can see the **shoulders**, since they were stretched so far to fit the collar around the neck, the shoulder barely have any edge loops.


The easy solution is to reinforce the area with more edge loops. 


Since the **UVs are staying the same**, you should be able to add edge loops easily without distorting the original texture.



:::info
You're heavily morphing a model. Distorted, stretched textures and wobbly topology **should** be fixed, though sometimes it's inevitable that some distortion is near impossible to completely fix. That's fine.

:::


![](./images/here-you-can-see-the-stretched-human-model-has-some-distorti.png)


Here you can see the clothing fitted to the Human model has some distortion in the textures compared to the original mesh. This can be further adjusted and fixed. Though, we're not expecting miracles, this amount of morphing is going to produce distorted results.


---


## What to Keep in Mind? 


* Give more room around the shoulder area so there's more to play with when stretching it out to the human shoulder size.


:::info
Older clothing pieces naturally weren't made to be morphed to humans, so had little to no shoulders room, creating stretched textures.

:::

![](./images/what-to-keep-in-mind.png)


---


## Final Checks & Layering

![](./images/final-checks-layering.png)

Test that your clothing fits over and under the other Human clothing layers. Read the [Layering Clothing](/assets/clothing/layering-clothing.md) page for reference.


## **LODs**


Your converted clothing needs LODs. You may be able to reuse the original clothing's LODs by transferring vertex positions from the fitted mesh using the same UVs.


Here is an example of 'Transferring Attributes' in Maya. Other software have different ways of doing this. 

![Human clothing LOD setup](./images/transfer.mp4)


:::info
Tools in 3D software can transfer attributes using shared UVs. Use the fitted LOD0 as the source and the existing LOD1, LOD2, and LOD3 meshes as the targets, then check the results.

If the transfer does not produce a good fit, rebuild the Human clothing LODs. You do not need to maintain the old Citizen version.

:::


If you're making major changes to the topology to fit it better around the human, creating a unique LOD 1, 2 and 3 may be inevitable either way. 



---


## Female Human Version

 

Use `citizen_human_female_REF.fbx` to create a female fit. It can be found in `sbox\addons\citizen\Assets\models\citizen_human`.


A **female version** of clothing is **not required** for every clothing piece. Female versions are expected for clothing like short sleeve shirts, shorts, clothing that shows a notable amount of the base body underneath. 


In cases like a jacket or trousers that completely cover the legs, we can get away with just using the male standard version. 


![](./images/female-human-version.gif)

In this example, the Leotard , most of the body is visible, so we have to want to make a female version, which is simply bringing the waist in and contouring around the chest. 


We want to make sure our female version of clothing isn't any bigger than the male version of the clothing, as this may cause clipping with the upper layers (jackets / vests on Layer 2).

So when editing to make the female version, only edit inwards, bringing in the ribs and under the chest. It's very tempting to just bring the chest out slightly for more silhouette, but it will cause clipping since it's making the clothing bigger.


![](./images/so-when-editing-to-make-the-female-version-only-edit-inward.png)


:::info
You can see here, all that's been done is bringing in the ribs and upper waist to follow the female base mesh. As well contouring around the chest slightly.

:::

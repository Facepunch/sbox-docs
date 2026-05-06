---
title: "Motion Editing"
icon: "🖌️"
created: 2025-06-30
updated: 2025-07-09
---

# Motion Editing

![](./images/motion-editing.png)This editor mode gives you finer control over track data. Instead of keyframes, you sculpt the raw track data itself at any resolution you want.

To motion edit, you select a time range of one or more tracks, then tweak the properties that those tracks represent to manipulate track data. You can also cut / copy / paste ranges of time, or save the selection as a separate .movie file.

# Time Range Selection

A time range selection describes how much a modification affects each moment in time. It's broken up into three parts:

* **Fade In** - modifications gradually ramp up within this range
* **Peak Range** - times in this range are fully affected by modifications
* **Fade Out** - modifications ramp down within this range


![](./images/time-range-selection.png)To make and update a selection:

* Click and drag in the timeline to select a time range
* Hold *Shift* and scroll to increase / decrease the fade in / out time
* Drag any of the parts of a time range to move or resize them
* Use *Ctrl+A* to select the whole movie's duration
* Press *Esc* to clear the selection


![Creating time range selections](./images/creating-time-range-selections.mp4)


You can change the easing type of the fade in / out sections with the corresponding button in the toolbar, or pressing number keys when mousing over those sections. The shape of the top / bottom of the time selection UI shows what the current easing function looks like.


![](./images/you-can-change-the-easing-type-of-the-fade-in-out-sections.png)

# Modifications

After selecting a time range, you can manipulate the properties of your movie's tracks. Only the selected time range will be affected, with the changes ramping up and down inside the fade in / out sections of your selection.


![Applying track modifications](./images/applying-track-modifications.mp4)

The selection will turn yellow to show a modification is in progress. At this point you can freely tweak the time selection, and when you're happy hit *Enter* or click the green tick to commit the change.

# Context Menu

The context menu for time selections has a ton of extra editing actions.


![](./images/context-menu.png)

### Time Selection Actions

* **Insert** (*Tab*) - add empty time inside the selected range, moving existing track data to the right
* **Remove** (*Backspace*) - delete the selected range, moving everything afterwards to the left
* **Clear** (*Delete*) - delete the selected range, leaving empty time
* **Save As Sequence..** - save the selected range to a .movie, and reference it in the current project

### Clipboard Actions

* **Cut** (*Ctrl+X*) - copy the selected range to the clipboard, then clear the range
* **Copy** (*Ctrl+C*) - copy the selected range to the clipboard
* **Paste** (*Ctrl+V*) - paste previously copied track data as a modification


#  ![](./images/clipboard-actions.png)Additive Editing

When pasting a time selection, you can additively layer track data over the existing animation. This is enabled by default, can can be disabled by clicking the *Additive* button in the toolbar while modifications are active.


![Additive editing with layers](./images/additive-editing-with-layers.mp4)

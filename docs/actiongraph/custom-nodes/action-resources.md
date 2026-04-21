---
title: "Action Resources"
icon: "📦"
created: 2024-01-23
updated: 2024-01-23
---

# Action Resources

You can create your own nodes by making a `.action` game resource.

First select a bunch of nodes that do whatever you want your new node to do, then right-click and select *Create Custom Node…*


![Creating a new custom node from an existing graph.](./images/creating-a-new-custom-node-from-an-existing-graph.png)

The button won't appear if it's not possible to split the selected nodes from the current graph, for example if you've selected the root node.

You'll then get a dialog to save your new node to a file.


![Custom node file creation dialog.](./images/custom-node-file-creation-dialog.png)

After clicking *Save*, you'll see the selected nodes have been replaced by a new single node with the name you chose.


![The graph from before, but with a new Jump node replacing the selection.](./images/the-graph-from-before-but-with-a-new-jump-node-replacing-the.png)

You can double-click on a custom node to open it in the editor. You'll see a root node was created for you, and if your selection had any out-going links there will be an output node too.


![Inside the custom Jump node.](./images/inside-the-custom-jump-node.png)

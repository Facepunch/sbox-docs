---
title: "Creating a Subgraph"
icon: "📦"
created: 2025-08-20
updated: 2025-09-01
---

# Creating a Subgraph

Subgraphs are a collection of Shader Graph nodes that can be easily reused as a single node across other Shaders and even other Subgraphs. This is an easy way of develop custom Shader Graph Functions of your own.

# Creating a Subgraph

You can create an empty Shader Graph Function by creating a new Asset and selecting Shader -> Shader Graph Function

![Creating an empty Subgraph](./images/creating-an-empty-subgraph.png)

You can also create a Shader Graph Function from pre-existing nodes in a graph by selecting them all, right clicking, and selecting "Create Custom Node"

![Right clicking on pre-existing nodes to create a Subgraph from them](./images/right-clicking-on-pre-existing-nodes-to-create-a-subgraph-fr.png)

# Subgraph Properties

When no nodes are selected, you can edit the properties of the Subgraph itself, including it's Name, Description, Icon, and whether or not it should appear in the Node Library (If not, the node will not appear in the right click menu or the Palette)


![The Subgraph Properties](./images/the-subgraph-properties.png) ![The Custom Node appears when added to the Node Library](./images/the-custom-node-appears-when-added-to-the-node-library.png)





# Creating Outputs

Every Subgraph has a "Result" node, which is similar to the "Material" output node from regular Shader Graphs. However in a Subgraph you can define as many outputs as you want via the Properties panel, and can even choose which outputs to display in the Preview tab.


![Creating an Output called "Output", and previewing it as the Albedo of the Preview tab 1380x770](./images/creating-an-output-called-output-and-previewing-it-as-the-al.mp4)

![The Output as seen on the Custom Node](./images/the-output-as-seen-on-the-custom-node.png)

# Creating Inputs

To create an input that is exposed on the resulting Subgraph Node, you can create a "Subgraph Input" node,

!["Tint" is a Color input, and "Intensity" is a Float input](./images/tint-is-a-color-input-and-intensity-is-a-float-input.png)

![The properties for the "Tint" Subgraph Input](./images/the-properties-for-the-tint-subgraph-input.png)

![The Inputs as seen on the Custom Node (with their default values)](./images/the-inputs-as-seen-on-the-custom-node-with-their-default-val.png)

Setting "Is Required" to true will not pre-populate a default value, and will instead require you to connect another node for the Graph to compile.

Order is used to ensure the correct order of the inputs on the left side of the resulting node.

# Using Textures

Any Texture nodes that are used in a Subgraph MUST be named in order to compile, as they cannot (currently) be baked with the shader. This is because any Material made using this shader needs to define any Textures used, and whoever is using your shader might not have the referenced Texture.

When using a Subgraph Node, the default Texture can be changed via the Properties tab, alongside the unrequired Inputs.

![The default Texture property exposed in the Properties tab when the Custom Node is selected](./images/the-default-texture-property-exposed-in-the-properties-tab-w.png)

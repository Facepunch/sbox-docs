---
title: "Feature Upgrade"
icon: "🛠️"
created: 2026-07-06
updated: 2026-09-03
---

# Feature Upgrade

`FeatureUpgrade` is a special utility in `FEATURES` block that mark an already-declared feature as an internal upgrade toggle. New materials will be automatically created with a new value assigned, while keeping old materials unaffected. This may be useful if you are changing some shader code that can break old, existing materials and you want to avoid that.

Features that are marked as an upgrader will be hidden from material editor UI. Existing materials will not have this feature referenced at all, meaning that the feature value there will be always **0**. New materials however will always have this feature included, and will be set to a value that was assigned from `FeatureUpgrade`.

```cpp
Feature( F_YOUR_FEATURE_HERE, 0..1 );
FeatureUpgrade( F_YOUR_FEATURE_HERE, Value );
```

## Usage Example

Add a new feature and give it a name. It is not necessary to specify the UI group name for this feature, since it will be marked as internal and won't be visible from material editor anyway.

```cpp
FEATURES
{
	Feature( F_UPGRADER_NAME, 0..1 );
}
```

To mark this feature as an actual upgrade toggle, add `FeatureUpgrade` anywhere after the main feature was declared:
```cpp
{
    Feature( F_UPGRADER_NAME, 0..1 );
    FeatureUpgrade( F_UPGRADER_NAME, 1 );
}
```

Now, this feature will be always set to **1** in your new materials. Old materials will not have this feature referenced in their files at all, this will be applied only to all new materials that will be created from now on. If you create a new static combo and connect it to this feature, it basically allows you to implement shader code which will be executed only in new materials, while leaving all old assets unaffected. 

Let's imagine that we changed the way textures are packed together into a Texture2D. If we apply this code change as is, without any code branching, it will break all existing materials. Instead, let's create a new static combo, which will be connected to our upgrade feature, and use it to make two different code paths. 

```cpp
PS
{
    // <...>
    StaticCombo( S_UPGRADER_NAME, F_UPGRADER_NAME, Sys( ALL ) );

    #if !S_UPGRADER_NAME
    {
        // This code path is for old, existing implementation of texture inputs and Texture2D packing.
        // Old materials will continue relying on this code path.

		CreateInputTexture2D( Albedo, Srgb, 8, "None", "_color", ",0/,0/0", Default4( 1.00, 1.00, 1.00, 1.00 ) );
		CreateInputTexture2D( Normal, Srgb, 8, "None", "_color", ",0/,0/0", Default4( 1.00, 1.00, 1.00, 1.00 ) );
        CreateInputTexture2D( Opacity, Srgb, 8, "None", "_color", ",0/,0/0", Default4( 1.00, 1.00, 1.00, 1.00 ) );

        Texture2D g_tAlbedo < Channel( RGBA, Box( Albedo ), Srgb ); OutputFormat( DXT5 ); SrgbRead( true ); >;
        Texture2D g_tNormal < Channel( RGBA, Box( Normal ), Srgb ); OutputFormat( DXT5 ); SrgbRead( true ); >;
        Texture2D g_tOpacity < Channel( RGBA, Box( Opacity ), Srgb ); OutputFormat( DXT5 ); SrgbRead( true ); >;
    }
    #else
    {
        // This is a new code path, where we fixed a bunch of input settings, and changed how textures are packed. 
        // ALL new materials will be using only this code path from now on. It will never be used in old materials.

        CreateInputTexture2D( Albedo, Srgb, 8, "", "_color", "Material, 10/10", Default3( 1, 1, 1 ) );
        CreateInputTexture2D( Normal, Linear, 8, "NormalizeNormals", "_normal", "Material, 10/20", Default3( 0.5, 0.5, 1 ) );
        CreateInputTexture2D( Opacity, Linear, 8, "", "_opacity", "Material, 10/30", Default( 1 ) );

        Texture2D g_tAlbedo < Channel( RGB, Box( Albedo ), Srgb ); OutputFormat( BC7 ); SrgbRead( true ); >;
        Texture2D g_tNormal < Channel( RGB, Box( Normal ), Linear ); Channel( A, Box( Opacity ), Linear ); OutputFormat( DXT5 ); SrgbRead( false ); >;
    }
    #endif
    // <the rest of the shader code...>
}
```

You can check out the [Fur Shader source code](https://github.com/Facepunch/sbox/blob/master/game/core/shaders/fur.shader) to see how feature upgrade is used to split the code path between old and new materials, and safely implement changes that would break existing materials without this upgrader.
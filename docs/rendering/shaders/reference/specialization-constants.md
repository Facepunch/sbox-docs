---
title: "Specialization Constants"
icon: "🔗"
created: 2026-09-22
updated: 2026-09-22
---

# What is it?

Shaders can use Vulkan specialization constants for features that change code without changing the shader inputs or resource layout. Add `ShaderConstant()` to a StaticCombo declaration, and the compiler can share compiled code between feature combinations, with the selected values supplied when the GPU pipeline is created.

Values of these constants are expected to be used in normal conditions and expressions instead of preprocessor branches. You still need to use regular static combos (without specialization constants) if they change resources or vertex/pixel struct.

# Usage

```cpp
FEATURES
{
	Feature( F_DETAIL_TEXTURE, 0..4, "Detail Texture" );
}

PS
{
    // ShaderConstant() is an additional argument that turns this static combo
    // into a specialization constant. Remove it and it will become a regular static combo again.
	StaticCombo( S_DETAIL_TEXTURE, F_DETAIL_TEXTURE, Sys( ALL ), ShaderConstant() );

	// Using static combo as a variable, notice if() instead of #if.
    // Still treated by GPU as a static combo, but it isn't required to compile it
	if ( S_DETAIL_TEXTURE == 3 || S_DETAIL_TEXTURE == 4 )
	{
		// detail normal mapping...
	}
}
```

It essentially turns this static combo into a variable instead of another `#define`, which would add up to list of shader variants that engine must compile. The benefit is that it is works as a variable while GPU still treats it as a static combo, doing necessary optimizations, but without having to compile it.

# ActiveIf() Shader Variable Attribute

You can also use this attribute along with specialized constants or features to mark specific variables as active/inactive so they become accessible and useable only when certain constants are enabled. 
Inactive variables will be hidden from material editor UI.

```cpp
StaticCombo( S_USE_BENT_NORMALS, F_USE_BENT_NORMALS, Sys( ALL ), ShaderConstant() );

Texture2D g_tBentNormal <
    Channel( RG, HemiOctIsoRoughness_RG_B( TextureBentNormal ), Linear );
    OutputFormat( ATI2N );
    SrgbRead( false );
    ActiveIf( S_USE_BENT_NORMALS != 0 );
>;

float g_flStrength < Default( 1.0 ); ActiveIf( S_USE_BENT_NORMALS != 0 ); >;
```

# Limitations

- This can be used only with `if`, `switch` and expressions. `#if` or `#elif`, texture processing annotations, conditional includes, render states or material attributes **will not** work with this.
- Works only with static combos, dynamic combos are not supported
- You can have up to **64** specialization constants in your shader
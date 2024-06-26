---
linktitle: SRG Semantics
title: Shader Resource Group Semantics (ShaderResourceGroupSemantic).
description: Learn about Amazon Shading Language (AZSL) shader resource group semantics (SRG) for Atom Renderer.
weight: 100
---

The Shader Resource Group Semantic (SRG Semantic), for short, defines the order in which descriptor bindings and descriptor sets are emitted for each SRG. In HLSL, descriptor sets are also known as _register spaces_. In AZSL, define SRGs by using the `ShaderResourceGroupSemantic` keyword.  
```cpp
ShaderResourceGroupSemantic <name>
{
    FrequencyId = <0-based index>;
    [ShaderVariantFallback = <number of bits>;] // Required only if using shader variant options.
}
```
An SRG Semantic relates to the frequency of change of SRG data. The render pipeline has a predefined set of SRG semantics in [`<Engine Root>/Gems/Atom/Feature/Common/Assets/ShaderLib/Atom/Features/SrgSemantics.azsli`](https://github.com/o3de/o3de/blob/main/Gems/Atom/Feature/Common/Assets/ShaderLib/Atom/Features/SrgSemantics.azsli) 
  
The table below outlines the various predefined SRG semantics and their change frequencies.
| SRG semantic | Change frequency |
| - | - |
| `SRG_PerDraw` | Data changes per draw packet. Highest frequency. |
| `SRG_PerObject` | Data changes per object. |
| `SRG_PerMaterial` | Data changes per material. |
| `SRG_PerSubPass` | Data changes per sub-pass. |
| `SRG_PerPass` | Data changes per pass. |
| `SRG_PerPass_WithFallback` | Same as SRG_PerPass, but used for cases where the Pass SRG defines the Fallback key. |
| `SRG_PerView` | Data changes per viewport. Each viewport has its own camera. |
| `SRG_PerScene` | Data changes per scene. Lowest frequency. |
| `SRG_RayTracingGlobal` | Reserved for ray tracing shaders. |
| `SRG_RayTracingLocal_[1..7]` | Reserved for ray tracing shaders. |
  
## `FrequencyId`
The value of `FrequencyId` defines the order of the compiled SRGs in register space and the order in which they are emitted.
Consider the following SRG definitions:  
```cpp
    ShaderResourceGroupSemantic SRG_PerDraw
    {
        FrequencyId = 0;
    }
    ShaderResourceGroupSemantic SRG_PerObject
    {
        FrequencyId = 1;
    }
    
    ShaderResourceGroup ObjectSrg : SRG_PerObject
    {
        int                      m_objectCounter;
        Texture2D<float4>        m_texture;
        RWTexture2D<float4>      m_rwtexture;
        Sampler                  m_sampler;
    }
    
    // Even though "DrawSrg" is declared after "ObjectSrg",
    // it uses a ShaderResourceGroupSemantic with higher frequency(FrequencyId == 0)
    // than "ObjectSrg" with lower frequency (FrequencyId == 1).
    // This means the resources of "DrawSrg" will be emitted before the resources
    // of "ObjectSrg".
    ShaderResourceGroup DrawSrg : SRG_PerDraw
    {
        int                      m_drawCounter;
        Texture2D<float4>        m_texture;
        RWTexture2D<float4>      m_rwtexture;
        Sampler                  m_sampler;
    }
```
When the AZSL code shown above is compiled with default arguments, it yields the following HLSL code:  
```cpp
    /* Generated code from
    ShaderResourceGroup ObjectSrg
    */
    Texture2D<float4> ObjectSrg_m_texture : register(t1);
    
    RWTexture2D<float4> ObjectSrg_m_rwtexture : register(u1);
    
    SamplerState ObjectSrg_m_sampler : register(s1);
    
    struct ObjectSrg_SRGConstantsStruct
    {
        int ObjectSrg_m_objectCounter;
    };
    
    ConstantBuffer<::ObjectSrg_SRGConstantsStruct> ObjectSrg_SRGConstantBuffer : register(b1);
    
    /* Generated code from
    ShaderResourceGroup DrawSrg
    */
    Texture2D<float4> DrawSrg_m_texture : register(t0);
    
    RWTexture2D<float4> DrawSrg_m_rwtexture : register(u0);
    
    SamplerState DrawSrg_m_sampler : register(s0);
    
    struct DrawSrg_SRGConstantsStruct
    {
        int DrawSrg_m_drawCounter;
    };
    
    ConstantBuffer<::DrawSrg_SRGConstantsStruct> DrawSrg_SRGConstantBuffer : register(b0);
```
In the compiled output above, you can see that both SRGs share the same register space (space0, implicit), aka the same descriptor set. And of particular importance, please note how the register binding for `DrawSrg` starts at 0, while the resources for `ObjectSrg` start at 1. This ordering was determined by the `FrequencyId` member of the SRG Semantics.  
  
Let's see what happens when *--use-spaces* is passed as the command line argument for **AZSLc**:  
--use-spaces: Each SRG will be assigned its own register space, ordered by the FrequencyId of the SRG Semantics.  
```cpp
    /* Generated code from
    ShaderResourceGroup ObjectSrg
    */
    Texture2D<float4> ObjectSrg_m_texture : register(t0, space1);
    
    RWTexture2D<float4> ObjectSrg_m_rwtexture : register(u0, space1);
    
    SamplerState ObjectSrg_m_sampler : register(s0, space1);
    
    struct ObjectSrg_SRGConstantsStruct
    {
        int ObjectSrg_m_objectCounter;
    };
    
    ConstantBuffer<::ObjectSrg_SRGConstantsStruct> ObjectSrg_SRGConstantBuffer : register(b0, space1);
    
    /* Generated code from
    ShaderResourceGroup DrawSrg
    */
    Texture2D<float4> DrawSrg_m_texture : register(t0, space0);
    
    RWTexture2D<float4> DrawSrg_m_rwtexture : register(u0, space0);
    
    SamplerState DrawSrg_m_sampler : register(s0, space0);
    
    struct DrawSrg_SRGConstantsStruct
    {
        int DrawSrg_m_drawCounter;
    };
    
    ConstantBuffer<::DrawSrg_SRGConstantsStruct> DrawSrg_SRGConstantBuffer : register(b0, space0);
    
```
In the compiled output above, you can see that both SRGs start the register bindings at index 0, but all resources of `DrawSrg` are assigned to `space0`, while the resources for `ObjectSrg` start at `space1`. This ordering was determined by the `FrequencyId` member of the SRG Semantics.  
  
  
Some platforms, like Vulkan & Metal, require all registers to be bound in sequence across all resource types for a given register space (descriptor set). To accommodate for this case, **AZSLc** accepts the command line argument *--unique-idx*.  
This is the output of the same AZSL file when compiled with *--unique-idx*, and using a single register space.  
```cpp
    /* Generated code from
    ShaderResourceGroup ObjectSrg
    */
    Texture2D<float4> ObjectSrg_m_texture : register(t4);
    
    RWTexture2D<float4> ObjectSrg_m_rwtexture : register(u5);
    
    SamplerState ObjectSrg_m_sampler : register(s6);
    
    struct ObjectSrg_SRGConstantsStruct
    {
        int ObjectSrg_m_objectCounter;
    };
    
    ConstantBuffer<::ObjectSrg_SRGConstantsStruct> ObjectSrg_SRGConstantBuffer : register(b7);
    
    /* Generated code from
    ShaderResourceGroup DrawSrg
    */
    Texture2D<float4> DrawSrg_m_texture : register(t0);
    
    RWTexture2D<float4> DrawSrg_m_rwtexture : register(u1);
    
    SamplerState DrawSrg_m_sampler : register(s2);
    
    struct DrawSrg_SRGConstantsStruct
    {
        int DrawSrg_m_drawCounter;
    };
    
    ConstantBuffer<::DrawSrg_SRGConstantsStruct> DrawSrg_SRGConstantBuffer : register(b3);
    
```
Note how all registers associated with `DrawSrg` start at index 0, while the registers bound to `ObjectSrg` start at index 4. Again, the reason the `DrawSrg` register indices start before `ObjectSrg` is because the `FrequencyId` of the SRG Semantic `SRG_PerDraw` is 0, while the `FrequencyId` of the SRG Semantic `SRG_PerObject` is 1.  
  
  
Finally, if we compile to use a register space per SRG, with *--use-spaces*, and with *--unique-idx* this will be the output HLSL code:  
```cpp
    /* Generated code from
    ShaderResourceGroup ObjectSrg
    */
    Texture2D<float4> ObjectSrg_m_texture : register(t0, space1);
    
    RWTexture2D<float4> ObjectSrg_m_rwtexture : register(u1, space1);
    
    SamplerState ObjectSrg_m_sampler : register(s2, space1);
    
    struct ObjectSrg_SRGConstantsStruct
    {
         int ObjectSrg_m_objectCounter;
    };
    
    ConstantBuffer<::ObjectSrg_SRGConstantsStruct> ObjectSrg_SRGConstantBuffer : register(b3, space1);
    
    /* Generated code from
    ShaderResourceGroup DrawSrg
    */
    Texture2D<float4> DrawSrg_m_texture : register(t0, space0);
    
    RWTexture2D<float4> DrawSrg_m_rwtexture : register(u1, space0);
    
    SamplerState DrawSrg_m_sampler : register(s2, space0);
    
    struct DrawSrg_SRGConstantsStruct
    {
         int DrawSrg_m_drawCounter;
    };
    
    ConstantBuffer<::DrawSrg_SRGConstantsStruct> DrawSrg_SRGConstantBuffer : register(b3, space0);
    
```
Note, for each register space, in this case, each SRG, all registers start at 0, BUT resources that belong to `DrawSrg` are bound to register space 0, while resources that belong to `ObjectSrg` are bound to space 1. This ordering of register spaces was determined by the `FrequencyId` member of the SRG Semantics.  
  
## ShaderVariantFallback: Defining Who Owns The Bit Array Of Options
`ShaderVariantFallback` is an optional keyword that marks an SRG as the owner of the array of bits where all Shader Variants will be encoded.  
Because there can only be one array of bits to encode the shader variant options, only one SRG can bind to an SRG Semantic that uses the `ShaderVariantFallback` keyword.  
If a shader makes no use of shader variant options, then the `ShaderVariantFallback` is not required to be used.  
  
For a deeper dive into shader variant options encoding see: [Shader variant options & The Fallback Key](shader-variants-fallback-key).
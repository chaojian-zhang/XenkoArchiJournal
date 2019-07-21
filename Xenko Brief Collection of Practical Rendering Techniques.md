In this document we provide an overview and server as a to-practice list for some of the useful rendering techniques, providing both index and hopefully elaboration.

# Xenko Usage Specific

1. [Render to render target](https://doc.xenko.com/latest/en/manual/graphics/graphics-compositor/render-textures.html)
    * Related: [Camera Properties](https://doc.xenko.com/latest/en/manual/graphics/cameras/index.html)
2. [Texture Properties](https://doc.xenko.com/latest/en/manual/graphics/textures/index.html#texture-properties)
    * Remark: Provides practical considerations concerning memory management and performance measurements
3. (Texture Compression and Effect)[https://doc.xenko.com/latest/en/manual/graphics/textures/compression.html]
    * Remark: Depending on texture detail density and contrast and sharp edges, use compression appropriately; Thankfully texture options in Xenko is not complicated and options are limited
4. (Texture Streaming\(Pending\))[https://doc.xenko.com/latest/en/manual/graphics/textures/streaming.html]
5. (Skyboxes)[https://doc.xenko.com/latest/en/manual/graphics/textures/skyboxes-and-backgrounds.html]
    * Remark: Also see (Skylight)[https://doc.xenko.com/latest/en/manual/graphics/lights-and-shadows/skybox-lights.html]
    * Remark: You can **capture a cubemap** from a position in your scene inside **Game Studio**
    * Remark: Xenko includes **an entity with a background component** in the project by default (this is what gives the **gray color**). Only **one background** can be active in a scene at a time.
    * Various **tools** exist to convert a panoramas to cubemaps and vice versa, including: (Panorama Converter)[http://gonchar.me/blog/goncharposts/2150], (Panorama to Cubemap)[https://jaxry.github.io/panorama-to-cubemap/], (Convert Cubemap to Equirectangular)[https://doc.xenko.com/latest/en/manual/graphics/textures/skyboxes-and-backgrounds.html]
6. Use multiple camera slots, Scene Renderer, Render to Texture
    * [Camera Slots](https://doc.xenko.com/latest/jp/manual/graphics/cameras/camera-slots.html)
    * [Scene Renderes](https://doc.xenko.com/latest/en/manual/graphics/graphics-compositor/scene-renderers.html)
    * [Render groups and masks](https://doc.xenko.com/latest/jp/manual/graphics/graphics-compositor/render-groups-and-masks.html)
    * [Render Textures](https://doc.xenko.com/latest/jp/manual/graphics/graphics-compositor/render-textures.html)
    * [Textures and render textures](https://doc.xenko.com/latest/jp/manual/graphics/low-level-api/textures-and-render-textures.html)
        * Also see **Textures and Render Textures** snippets in **Xenko Snippets.md**
    * [Custom scene renderers](https://doc.xenko.com/latest/jp/manual/graphics/graphics-compositor/custom-scene-renderers.html)
    * [Debug renderer](https://doc.xenko.com/latest/jp/manual/graphics/graphics-compositor/debug-renderers.html)

# Topics

## Shadow Map Resolution

Shadows (maps) are controlled by: **MeshComponent**'s Case Shadow option, **LightComponent**'s Shadow property. Only **lights** (specifically **directional lights**, **point lights**, and **spot lights**) can cast shadows.

Xenko creates a **shadow map** for **each light** that casts shadows. Light **Shadow options** can configure **shadow map size**, but there is an pretty small upper bound around 2K. However, the **actual display quality** of such shadow maps is not just affected by **shadow map resolution** of the given light, but also on the (size and quantity of) **shadow receiving objects**. Shadow maps are shared by all instances of **shadow receiving objects** (*all mesh objects receive shadows*), and the shadow resolution on each of object's surface is greatly limited by its share of shadow map - and this share is deteremined by ??? (*texture size of the object? By the way does object require UV unwrapping to receive shaodws?*). For efficient packing, use **Prefabs** and **Prefab models** for optimized draw calls, but this won't help with **shadow map resolution**.

A decision must be made to control **effective shadow region**, in **UE4** this is done by **LightmassImportanceVolume**, which limits the lighting and shadowing for **dynamic lighting** - notice in UE4 the rest of the scene is usually baked lighting which depending entirely on **shadow map resolution of individual assets**. In open world settings it's thus important to utilize both **baked shadows** and **localized dynamic lights**. UE4 handles it better.

Reference: [Shadow Maps in Xenko](https://doc.xenko.com/latest/en/manual/graphics/lights-and-shadows/shadows.html). From this documentation it seems that **shadow atlas** is shared among **all lights** and thus greatly limited in size. This shouldn't be a problem depending on context because usually the *actual viewport size is no larger than the shadow size* thus per render-wise speaking that should suffice.

## Graphics Compositor - Add a secondary camera



# Generic Applications

1. \[Pending Review\](What is the best way to unwrap a sphere?
)[https://blender.stackexchange.com/questions/10741/what-is-the-best-way-to-unwrap-a-sphere]
    * **Conclusion:** **Icospheres** can be used for stretch-less UV mapping. The seemly **weird stretching** in UV map as per **Spherical Mapping** in Blender actually extends naturally to the other side of the **rectangular image texture**.
    * **Related:** Examples of (Sphere UV Unwrapping Practice for Solar System)[https://polycount.com/discussion/69961/unwrapping-a-sphere]
    * **Remark:** Depends on **Texture** itself's mapping, and the **actual shape** we are trying to represent
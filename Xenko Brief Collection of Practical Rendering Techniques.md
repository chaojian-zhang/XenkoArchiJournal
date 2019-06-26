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

# Generic Applications

1. \[Pending Review\](What is the best way to unwrap a sphere?
)[https://blender.stackexchange.com/questions/10741/what-is-the-best-way-to-unwrap-a-sphere]
    * **Conclusion:** **Icospheres** can be used for stretch-less UV mapping. The seemly **weird stretching** in UV map as per **Spherical Mapping** in Blender actually extends naturally to the other side of the **rectangular image texture**.
    * **Related:** Examples of (Sphere UV Unwrapping Practice for Solar System)[https://polycount.com/discussion/69961/unwrapping-a-sphere]
    * **Remark:** Depends on **Texture** itself's mapping, and the **actual shape** we are trying to represent
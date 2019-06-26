For some practical reasons there are **hard-limits** imposed on the engine, and it's essential and practical to be aware of such so we can better unleash our creative skills.

# Rendering Capabilities

1. Maximum Shadow Map resolution: 1024^2
2. Maximum Image Texture resolution: 

## References

1. (Ideal Game Texture Size)[https://forums.ogre3d.org/viewtopic.php?t=53524]
    * Smaller by **a factor of 4** at least (**hardware convention**). **High res textures** in a game usually are in the **512x512** range, and generally smaller textures are used (128 and 256 being relatively common). 
    * "Mipmapping will force much lower resolution versions to be used for rendered pixels anyway".
    * "One cannot tell what texture size is appropriate without knowing what kind of model you're talking about, from which distance it will be seen and what kind of systems you're targeting."
    * "2048x2048 in anything but DXT/BCx compression is 16MB in memory, and even with DXT1 compression, is still roughly 2.3MB." **Budget your memory** for more important stuff.
2. \[Pending Review\](Texture size in 2017 modern PC Games)[https://forums.unrealengine.com/development-discussion/rendering/109587-texture-size-in-2017-modern-pc-games]
    * "I think in addition to **bigger resolution textures** we are slowly moving back to **smaller texture sizes**, and use additional **noise/masks** to counter tiling/repetitiveness. Additionally **detail albedo/roughness/normal/whatever** will also help giving lower res textures more oomph instead of having to add all that detail in the ever-increasing texture maps." Also "more and more **procedural stuff**".
    * "General rule of thumb has been **1024x1024** per **1 square meter**." (Also depending on **view distance**)
    * (A practical example)[https://www.artstation.com/artwork/6kEmV]
    * **Remark**: this is a very good post covering **quite some details**
2. (How many models/textures in a typical performant game \(After optimization\))[#]
3. (Best practices of texture size)[https://gamedev.stackexchange.com/questions/64106/best-practices-of-texture-size]
    * "I wouldn't use **LOD bias settings** to reduce textures. If you do that, you're still paying for the cost of the high-res texture in memory and loading time. Instead, reduce the texture in a **preprocess**, and only load the smaller version of it. It's still good to author the original textures at a high resolution so you have the extra detail in case you need it later."

# Scene Management

1. Maximum World Scale (with reasonable accuracy): ???

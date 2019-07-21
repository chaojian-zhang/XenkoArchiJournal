# Modules (Per 3.1)

Per Xenko 3.1, engine components are seperated into individual modueles for reference. It's possible to create a project (not necessarily for game) that references only the packages we need. 

It's said that Xenko is **Developer Oriented** game engine, rather than **Designer Oriented**, thus makes it cleaner to program with.

Examples of **“core” packages** for the game engine:

1. Xenko.Engine: allows you to use core engine runtime (including its dependencies)
2. Xenko.Core.Assets.CompilerApp: compile assets at build time
3. Xenko.Core.Mathematics or Xenko.Graphics: yes, if you want to make a custom project only using Xenko mathematics or graphics API without the full Xenko engine, you can!
4. Xenko.Core.Assets, Xenko.Presentation or Xenko.Quantum: all those piece of tech being used to build Xenko tooling are also available for reuse in other projects. Nothing prevents you from generating assets on the fly too!

Various parts of the engine are distributed as **optional packages**:

1. Xenko.Physics
2. Xenko.Particles
3. Xenko.UI
4. Xenko.SpriteStudio
5. Xenko.Video

Most modules are expected to conform to **.Net Standard** soon.

# Gameplay Architecture

Notice this document intend to only record and document **high level** architecture and provide guidance into **implementing gameplay systems**. For **detailed API calls** etc. see official doc. Information of below document is taken from **official manual** and **api doc**, along with some personal experience with the systems and backgrounds. Those shall be mentioned whenever necessary.

For effective interaction with the game engine, **editor usage shortcuts** and **specific API references** are handy and shall be documented in seperate documents.

## Scene and World

Each **level** is called a **scene** and is represented by a **scene file**. Each **scene** contains many **entities**. Each object in the scene, no matter its appearance and function, is represented as an entity.

One cool thing about Xenko is **scenes** can contain **subscenes** in their definitions, each scene can have its own **scene folder**, and **subscenes** can be loaded at runtime. **Resources** from the parent/root scenes are **shared** by subscenes e.g. scripts.

Entities behaviors are defined by their entity **components**.

For maximum flexibility when **programming** against the game engine, **direct manipulation of entities** gives the power for customized behaviors. For instance, **Mesh** entities can be loaded from **assets** in real time.

Entities in the scene can also have **parental hierachical structure**, mostly affecting their **local transformations**.

The **default entry scene** must be specified for the actual game otherwise player has no where to start, this is managed by **Global Game Settings** asset file. Search below for information on this file. Worth noting, as a side effect of this, the **game start menu** is usually implemeneted as **a seperate scene** and configured as the startup scene.

**Enumration of Entity Presets (Tagged, not Categorized, Ordered Alphabetically)**

Entities are **runtime** **object-oriented representations** of things. They live in **memory**. Notice the relation between entities and assets are not directly 1-1: some of the entities mentioned below are **"pre-defined (empty) entities"** in such a sense that they contain **components** that uss the **specific corresponding assets** - if we create an **empty entity** and give the same components we can achieve the same effect.

That's why instead of calling this section *"Enumeration of Types of Entities"*, I called it *"Enumeration of Entity Presets"*.

1. Empty
2. Mesh
2. Camera
3. Light

**Enumeration of Types of Components (Tagged, not Categorized, Ordered Alphabetically)**

Since entities are ultimately composed of **components**, which directly references assets, it is necessary to enumerate all usable components.

Notice each `Xenko.Engine.Entitycomponent` instance is tied to its **entity** (that is, one component can't be added to two entities at the same time).

1. Audio Emitter
2. Audio Listener
2. Background (Skybox)
2. Camera
3. \[Physics\] Character: Enables input and script controlled **collider** behvaiors; Entities with **character components** can only be moved with `SetVelocity`, `Jump`, and `Teleport`.
4. \[Physics\] Kinematic Rigidbody: Use **rigidbody component** with `IsKinematic`; Kinematic rigidbody can be moved in scripts (like **Static Colliders**, unlike **Characters** or regular **Rigid Bodies**), but also affects other **rigid bodies** and **is**  affected by **physics simulation** (e.g. gravity) when `IsKinematic` is turned off.
2. Light
3. Mesh
4. Navigation Bounding Box
5. Particle System
6. \[Physics\] Rigid Bodies: Rigidbodies move based on **physical forces** applied to them (and is only affected by physics), such as **gravity** and **collisions**. Regid bodies represent objects that are pushed, pulled, and knocked around, and also have **effects on other rigidbodies** they collide with (also affect **Characters** and **Kinematic objects**; no direct physical effect on **Static Collider**); Can be toggled **kinematic**, in which case they can be **scripted** through **transformation**.
4. Script*: notice each script will become a seperate "component" type
5. \[Physics\] Static Collider: **Static colliders** aren't moved by forces such as **gravity** and **collisions**, but other physics objects can bump into them. Notice static colliders **can be moved** in script. See script snippets.
5. (2D) Sprite
6. (2D) Sprite Studio
4. Transform*: As is for all tangible physical objects represented in the world, all entities have a **Transform** component representing their whereabouts.
5. (2D) UI
6. Video

**Enumeration of Types of Assets (Tagged, not Categorized, Ordered Alphabetically)**

Assets are specific **binary files** ready for consumption in game. They live **on disk**. Assets are organized using **folders**.

Assets contain **properties** defining their **loading behavior** (i.e. from disk to memory ready to be used as components for entities).

Throughout the engine, assests can be used in 3 ways: 1) reference them in *entity components* (effective at **runtime**); 2) reference them in *other assets* (e.g. **materials** references **textures**, **models** references **materials**) (effective at **loadtime/buildtime**); 3) load them from *code* (effective at **runtime**).

1. (Skeletal) Animations
2. Audios
2. Game Settings
2. Graphics Compositors
2. Lights
2. Materials
3. Models
4. Procedure Models
4. Particles
5. 
6. Prefabs
7. Raw Binary
7. Scenes
8. Script Source Codes
8. Skyboxes
8. Textures
9. UI Pages
10. UI Libraries
11. Video

A recommended way to **organize assets** if **by type** (i.e. rather than by scene, because scene already contains all the **pointers to the assets** and serve as an **organization unit** in itself).

One cool thing about Xenko is the **build of assets** (into final game executable binary) is selective: meaning only 1) asets that are **marked for inclusion** (**root assets**), or 2) assets that are referenced by a **root asset** are built.

**Supported Asset Resource File Formats**

Assets are either created from with-in the **game editor**, or imported from external **content creation tools**. Belows lists supported **file formats** for **importing**. The list is categorized by *general asset type*.

1. **Models, animations, skeletons:** `.dae`, `.3ds`, `.obj`, `.blend`, `.x`, `.md2`, `.md3`, `.dxf`, `.fbx`
2. **Sprites, textures, skyboxes:** `.dds`, `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.tga`, `.psd`, `.tif`, `.tiff`
3. **Audio:** `.wav`, `.mp3`, `.ogg`, `.aac`, `.aiff`, `.flac`, `.m4a`, `.wma`, `.mpc`
4. **Video:** `.mp4` (better performance, no conversion is needed), among others
5. **Fonts:** `.ttf`(**Sprite fonts** take a **TrueType** font as an input and then create all the **images** (sprites) of **characters** (glyphs) from it; See (Sprite Fonts)[https://doc.xenko.com/latest/en/manual/graphics/sprite-fonts.html])

It is important for developers and content creators to become familiar with those file formats and especially their **origins** and **usage history** by reading through corresponding entries on **Wikipedia**.

As a remark, notice **materials**, **scenes** cannot be imported, though those can have significant *creation time effort impact*.

Also notice:

* **Xenko** only imports the first frame of **animated image files**, such as animated **gif**s or **PNG**s. They don't animate in Xenko; they appear as static images.
* In the (doc)[https://doc.xenko.com/latest/en/manual/graphics/textures/index.html#supported-file-types] it's claimed *"Xenko currently doesn't support movie files"* which seems not the case as somewhere else in the doc about (movie setup)[#]

## Run-time Asset (Already Covered in Snippets)

## Entities (Already Covered above)

## Scripting

**Scripts** are written in **C#** and created as **C# script source files** (regular text files) and imported (or created directly inside game editor) as **Script Source File** **assets**. Scripts are assets and not automatically used but after **solution compilation** they will be registered as available **entity components**.

To use a script, **attach the script to an entity in the scene** (e.g. for non-game object related scripts e.g. high level game logic related scripts, just attach to an **empty entity**). For universal **shared game functions** e.g. game mode management and other misc. stuff, write a **master script** and include that script in **all scenes** that are affected.

As remark on this approach, i.e. setting the script on a *per scene* as a distinct **scene entity** versus setting scripts as **Scene asset properties** or at the **engine level** - for instance in Unreal Engine we set **Start/Defaut GameMode (Class)** and **Default Game Controller (Class)** and **Default Map (Asset)** etc. - in sacrifice of **one Entity Transform Component** (for the **empty entity**), we obtain great flexibility and localized managebility because things are relevant where they need to be. The same pattern is observed for **Global Game Settings** which is implemented as a **Game Settings asset** (instead of, for instance, through some UI-bound **configuration menus**). We observe this pattern as resource-based configuration, and it promotes flexibility, like how **Visual Studio Code** and **Sublime Text** configures its components using **text files** (and both are very useful text editors).

**Types of Scripts**

There are three [types of user-created scripts](https://doc.xenko.com/latest/en/manual/scripts/types-of-script.html) in Xenko: startup scripts, synchronous scripts, and asynchronous scripts. Besides these, normal c# classes can be created but those *won't be registered with **script system***.

One important thing about scripts is **script execution order**, especially when one script depend on another, e.g. for **sync scripts** which need to be updated every frame and use `EventKey` and `EventReceiver` to communicate data. In this case, **script priority** can be sent in **entity components**. Also see [Scheduling and Priorities](https://doc.xenko.com/latest/en/manual/scripts/scheduling-and-priorities.html).

By the way **events** are used like **buffers**, and it's a convinient way to **share data** without explicitly **specify a target** or **shared data structure** (when execution order is well-defined). Otherwise it's used for signaling purpose. Example see **FPSTemplate** `PlayerController`, `InputController` and `AnimationController`.

## Rendering Pipeline (Request for Review; Request for Simplification)

Easily customizable items throughout the rendering pipeline includes: ... (windows border)

There are **three major aspects** to graphics presentation: **procedural generation**, **2D UI**, and **3D meshes rendering**. In addition, the **design language** and **graphics style**, along with the **modelling details**, **realism or accuracy** of the contents are a subject of each game's unique elements.

The above three aspects correspond to the following functions in the game engine:

1. Window Configuration:
2. Rendering Capability Configuration:
2. Asset Preparation:
2. 2D UI Programming:
3. Shader Programming:
4. Post-Processing Effects:

One cool thing about Xenko is all those can be issued in a text-based DSL (domain specific language), thus avoid dependency on specialized editors and promotes data transparanct and easily allow version control and collabotartion, letting alone smaller in packaging size.

Fundamental to the utilization of all **rendering capabilities** is the understanding of underlying process and **rendering representations**. A brief summary shall be provided below. For detailed information on Xenko's Rendering Pipeline refer to (official doc)[https://doc.xenko.com/latest/en/manual/graphics/rendering-pipeline/index.html].

More technical and operational treatments are available in **Xenko Shading Langauge Reference.md**.

**Rendering Representation**

All various things constituting what the player sees and ultimately interacts with (unless it's a visualess sound-only keyboard-based game), need to be somehow **stored**, **represented** and finally **presented**. The **storage** refers to **raw graphical assets** including texturs and materials and meshes, and also includes instruction-level details like shaders; **Representation** means the *abstract construct* used to measure and make calculable the various graphical assets, for instance a texture is represented as a 2D grid of **RGBA** **floating point values**, a shader is represented as multiple lines of **shader language statements** which will ultimately be translated into target processor (for GPUs) executable **binary machine code**; Presentation refers to finally getting the pixels presented to the player, which may be further abstracted into multiple targets including **rendering to texture** or **rendering to a window**.

Rendering representation can be considered the connecting bridge between **game data** and **user experience**: all the graphical assets need to be processed somehow before they can show as tangible pixels on scren. From a programmer's perspective, this concerns 3 things: **assets**, **game objects**, and **rendering effects** (*Post-Processing?*). Those three things are already defined by the engine as mentioned above, where assets are **assets** in the engine (lives on disk), game objects are **entities** in the scene (lives in memory), and rendering effects are what are presented **on screen** (lives in memory as well but can be considered just a flat screen).

The first step is to convert assets to game objects, including the ones that are visible and the ones that are not visible. This includes things like **Light Entity** which is not visible by itself but illumintates everything else. All game entities are merely **object-oriented abstraction** of what they are trying to represent in the real world, and stored as **a structure of various basic values**. For instance, A **mesh asset** is converted to a **mesh entity** in the scene with a **mesh component**; The mesh itself is a representation, or **approximation**, or the real world shape of the object it's trying to mimic (potentially with some artisitic style); The **mesh component** contains **detailed data** of the mesh including **vertices** and **indices** and **texture coordinates** information, which are just a bunch of floating point numbers. Another simple example would be a **light entity** which represents a **light source** in reality and has parameters like **intensity** and **color**.

The second step is to process all the **representation data** and **compute** the final pixel value for player to see. In order to achieve this efficiently and in an organized fashion, various graphical entities of **similar nature** are grouped and handled together by dedicated game engine components. This kind of categorization is called **Render Features** and **Rendering Objects**. In the end those features are drawn using system functions or graphics cards, in which case Xenko engine is responsible for converting the features to API calls of the supported drivers/hardwares. During this phase, **Xenko shaders** are converted to the **target graphics platform** (**GPU Executable Programs**), either plain HLSL for Direct3D, GLSL for OpenGL, or SPIR-V for Vulkan platforms. Shaders manipulate **geometrical data** (both 2D and 3D) and generate **pixels**.

**Render Objects**

meshes, sprites, particles

RenderObjects are registered with a VisibilityGroup. During the collect phase, the visibility group culls and filters them based on the RenderView and RenderStage (Represents a way to instantiate a given RenderObject for rendering, esp. concerning effect selection and permutation.).

**Render Features**

Xenko executes features in phases: collect, extract, prepare and draw. This means each step of the pipeline can be isolated, parallelized and optimized separately.

**Render Views**

Render views are a frustum and some camera parameters, including parameters like clipping plane distance. Details available in (API doc)[https://doc.xenko.com/latest/en/api/Xenko.Rendering.RenderView.html].

**Shaders**

Shaders can be created at runtime.

**On Photorealism**

Xenko uses shadow mapping to render shadows, which means it's weak at **dynamic shadows**. Since Xenko uses PBR shaders, it is good at mimicing statis scenearies.

## Engine Configuration and Global Game Settings

As mentioned above, **Global Game Settings** are stored in a text-based (it used **YAML** format and a custom suffix `.xkgamesettings`) **Game Settings asset**. Worth noting is **all aspects** of the game engine, including **editor behavior** and actual **game runtime** are configured by this settings. Game settings are crucial for an efficient game development expeirence and gameplay experience, so we will provide **extensive treatments** on a **selective set of settings**, and provide **contexts** as to how they affect our **decisions**.

For more details see this (documentation page)[https://doc.xenko.com/latest/en/manual/game-studio/game-settings.html].

**Editor**

Editor settings don't affect game.

# Project Structure

I will not talk about specifics of each file but briefly mention the organization of Project items in general.

There are fource aspects for each Xenko project, or **solution**: 1) Each game is conssited of multiple **c# projects**, each producing some binary assembly and linked together for the whole game. 2) Libraries and engine components are referneced as **C# assebmlies** or dlls. 3) Assets can be treated as a seperate assembly/project (equivalent in terms of c# project); 4) Codes can be treated as a seperate assembly/project.

Besides that, each game will have an **Windows project** an **entry point**, which then references that custom game assets and game codes, as a seperate **module/assembly/project**.
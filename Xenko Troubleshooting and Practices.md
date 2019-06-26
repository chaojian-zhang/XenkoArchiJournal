# Overview

Sometimes erros occur that stops the game from running properly (during testing etc.).

In this case check **Output log**, and use **Debug** in **Visual Studio**.

# Pre-Cautions

1. Don't put **GameSettings** files inside folders, don't change its location
2. "When declare 'dependencies' (no idea what that means, maybe it just mean properties) of an object as public. Because of course objects should not hide their dependencies so that they are mockable and injectable by IoC DI containers. (no idea what that means either). That exposing property on script in editor means that the engine will try to **serialize** it. There can be some "serializer warning". To **attribute public dependency** with `[DataMemberIgnore]`` so engine won’t try to serialize it.

# Troubles

## \[Game Setting\] Game won't start, build succeeded; Run will instantly close

**Symptom:** 

If debug the solution inside Visual Studio will receive following error at `game.Run()` inside `Main()`:

```
Xenko.Core.Serialization.Contents.ContentManagerException: 'Unexpected exception while loading asset [GameSettings]. Reason: No serializer available for type id f6e78a35184f6fe1d27096e38ec09340 and base type Xenko.Data.Configuration. Check inner-exception for details.'

Inner Exception:
ArgumentException: No serializer available for type id f6e78a35184f6fe1d27096e38ec09340 and base type Xenko.Data.Configuration.
```

**Similar Issue:** [Xenko.Core.Serialization.Contents.ContentManagerException when trying to run project](https://github.com/xenko3d/xenko/issues/299)

**Cause and (Failed) Solution:**

Search through the **whole solution** for text appearance `f6e78a35184f6fe1d27096e38ec09340` and `f6e7` because I realized it's not an **actual GUID format**, but no result, could it be from some some sort of "Cache"? Clean solution doesn't help. I think it would be helpful if I can see what actually happened inside this function call but I don't know how I can **possibly step into engine code** to see what happened?

People see online talking about Serialization error mentioned something related to scripts or project settings, but I didn't change those? **Wrong name space** could have affected but all namespaces I use in scripts are correct. The **game settings** and **project settings** are almost exact as **default FPS tmeplate**. Except I ***moved folders around a bit***.

**How to re-create:**

1. Create a **first person shooter template** (does it work with anything else?)
2. Click **Run**, this works
2. Create folder **Scenes**
3. Move **MainScen** into **Scenes**, Click run, Game Studio outputs `(0,0)]: Info:Deployment of XenkoSerializationError.Windows successful.` but the game can no longer run
4. Open the solution using **VS2019** and **debug** we will see the error: `Xenko.Core.Serialization.Contents.ContentManagerException: 'Unexpected exception while loading asset [GameSettings]. Reason: No serializer available for type id f6e78a35184f6fe1d27096e38ec09340 and base type Xenko.Data.Configuration. Check inner-exception for details.'`. Notice the very specific ID string: `f6e78a35184f6fe1d27096e38ec09340` - usually when we create a new template **all assets** have a **new set of ID**, but this ID is always the same.

**Expected behavior**

Everything should just work.

**Notice:**

1. At this time if we move **MainScene** back to **Assets** folder and delete **Scenes** folder, the game still have the same `Xenko.Core.Serialization.Contents.ContentManagerException`: and there seems no way to fix this
2. Notice in general **GameSettings** asset also cannot be moved into subfolders of **Assets**, this is irrelevant of FPS template, but applicable to empty **New Game** as well
3. If we create **Scenes** folder then create a new scene directinside then assign that as startup scene in **GameSettings**, do a save, then move **MainScene** into **Scenes**, then change **GameSettings** to **Scenes/MainScene**
4. Apply the same procedure to an **Empty Game**, there is no issue at all; So I suspect it has something to do with First Person Shooter template
5. What's more, it appears you cannot move any of the **Crosshair256**, **Skybox Texture**, **Skybox**, **Bullets** assets into subfolder, otherwise there can be exceptions like `[RouterClient]: Error: Could not connect to connection router using mode Connect. Xenko.Engine.Network.SimpleSocketException: Connection router did not connect back to our listen socket at Xenko.Engine.Network.RouterClient.<InitiateConnectionToRouter>d__5.MoveNext() Xenko.Engine.Network.SimpleSocketException: Connection router did not connect back to our listen socket at Xenko.Engine.Network.RouterClient.<InitiateConnectionToRouter>d__5.MoveNext()` and some other issues.

**Remark**

This is very **bad** (not morally, but **confusing**), because I once setup several scenes with **First Personal Template** and moved **MainScene** into **Scenes** folder, only to discover the *whole game cannot no longer run* and there is *no way to fix it*, and I would have to **start creating all those scenes again** because apprantly there is currently no way to **migrate assets between projects** and I am not sure whether it's safe to **just copy assets around**. Please help! Thanks.

**Final Remark**

Apparently, reverting all changes in `Assets\GameSettings.xkgamesettings` will resolve the problem, specifically, it's below lines that's causing the issue:

```
    - !Xenko.Navigation.NavigationSettings,Xenko.Navigation
        EnableDynamicNavigationMesh: false
        IncludedCollisionGroups: AllFilter
        BuildSettings:
            CellHeight: 0.2
            CellSize: 0.3
            TileSize: 32
            MinRegionArea: 2
            RegionMergeArea: 20
            MaxEdgeLen: 12.0
            MaxEdgeError: 1.3
            DetailSamplingDistance: 6.0
            MaxDetailSamplingError: 1.0
        Groups:
            -   Id: 20c6d98c-4c9b-40c0-aa04-138543a9eedd
                Name: New group
                AgentSettings:
                    Height: 1.0
                    MaxClimb: 0.25
                    MaxSlope: {Radians: 0.7853982}
                    Radius: 0.5
```

## \[UI\] The Added ImageElement display of Texture seems dimmer than the original Texture's color

Sympotom: I have a very bright texture but when it's added to UI it looks dimmer.

Cause and Solution: (Not available yet)

## \[Building;Packging;Testing\] Game builds and plays in Game Studio but cannot compile in Visual Studio

**Symptom:** 

Below compile error is thrown:

```
Severity    Code    Description Project File    Line    Suppression State
Error   4.812s  [AssetCompiler] Unhandled exception. Exception: Could not find asset Materials/Planet for bundle default    RoboGo.Windows  H:\ProjectHephaestus\RoboGo\RoboGo.Windows\EXEC 1   
```

**Scenario/Cause:**

Just added the **asset** and added a **script** and trying to debug the script from Visual Studio.

**Solution:**

1. I am not sure the exact cause, but one temporary solution is to mark the mentioned asset as **Root Asset**, which will solve this issue

**Reference:** [Root Asset](https://doc.xenko.com/latest/en/manual/game-studio/manage-assets.html#include-assets-in-the-build)

**Related:** [AssetCompiler errors such as missing sources are silenced (build marked as success and failing during packing) #246](https://github.com/xenko3d/xenko/issues/246)


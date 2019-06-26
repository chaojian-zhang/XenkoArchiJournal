# Overview

**Snippets** are important: they are short and succint illustrative pieces of code that can sometimes reveal the internal workings of the system; their context and appropriate use are critical and they can just get the things done.

In this document I will record both the **statements**, appropriate **usage** (where, when, how), and underlying meaning/implication/interaction with the **game systems**. This document should not be read as a receipe (although it definitely servers such a reference purpose), instead it is considered a complementray to the architecture doucment.

For graphics and shader related snippets, see **Xenko Shading Language Reference.md**.

**Debugging Tip**

1. Watch out for Game Studio output panel logging info
2. Watch out for Console window error output (for the game instance)
3. Use VS IDE to step through code

**Engine Features**

1. Public properties of a script is automatically exposed inside Game Studio
    * Public properties of a script should be data only and shouldn't access runtime functions or properties e.g. Entity
2. Use `[DataMemberIgnore]` to hide public properties in GameStudio properties panel
3. Normal C++ classes can also be defined and used but they won't show as scripts


# Snippets

All snippets will be presented in an **ordered list** manner orderly **alphabetically** be their usage or **name** (in the format of an **action**), and tagged with **keywords**.

Notice in **Add Assets**, some premade scripts are available for things like **camera control** and **event handling**.

1. \[Scripting\] Add a Script to Entity

```c#
// myEntity is an existing entity in the scene; myAsyncScript is the script you want to add to the entity
myEntity.Add(new myAsyncScript());
```

2. [Debug, Testing] Output quick test output

```c#
Console.WriteLine(entity.Name);
```

Remark: Seriously, set breakpoint inside Visual Studio is more efficient.

2. \[Scripting\] Async Script Looping Per Frame

```c#
public class BasicAsyncScript : AsyncScript
{   
    public override async Task Execute()
    {
        while(Game.IsRunning)
        {
            // Do some stuff every frame
            await Script.NextFrame();
        }
    }
}

// In comparison, here is how Sync Script does it
public class SampleSyncScript : SyncScript
{           
    public override void Update()
    {
        if (Game.IsRunning)
        {
            // Do something every frame
        }
    }
}
```

Remark: Async Scripts can also execute everyframe.

Remark: Xenko doesn't run scripts **simultaneously**; they run **one at a time**. Where scripts depend on each other, you should make sure they run in the correct order by giving them **priorities**.

Remark: Notice each **script component** is a **seperate instance** of the same script (class); This allows some concurrency.

1. Access Files

```c#
Use VirtualFileSystem: https://doc.xenko.com/latest/en/api/Xenko.Core.IO.VirtualFileSystem.html

// Open a file through VirtualFileSystem
var gamesave1 = VirtualFileSystem.OpenStream("/roaming/gamesave001.dat", VirtualFileMode.Open, VirtualFileAccess.Read);

// Alternatively, directly access the same file through its file system provider (mount point)
var gamesave2 = VirtualFileSystem.ApplicationRoaming.OpenStream("gamesave001.dat", VirtualFileMode.Open, VirtualFileAccess.Read);
```

Remark: It's OK but we should be aware if some of the libraries we reference actually accessed files using raw runtime functions e.g. System.Database.SQLite.dll

2. Create an Entity and Some Components

```c#
// Create entity
var myEntity = new Entity();

// Create a model component (so that model is rendered)
var modelComponent = new ModelComponent { Model = model };
myEntity.Set(ModelComponent.Key, modelComponent);

// Set entity position
myEntity.Transformation.Translation = new Vector3(100.0f, 100.0f, 0.0f);

// Add entity to scene; from now on its model will be rendered
Entities.Add(myEntity);
```

2. Load Asset from Code

```c#
// Load a model (relative path; replace URL with valid URL)
var model = Content.Load<Model>("AssetFolder/MyModel");

// Create a new entity to add to the scene
Entity entity = new Entity(position, "Entity Added by Script") { new ModelComponent { Model = model } };

// Add a new entity to the scene
SceneSystem.SceneInstance.RootScene.Entities.Add(entity);
```

Remark: **Asset URL** is visible when hovering mouse on asset.

Remark: `Xenko.Core.Serialization.Assets.AssetManager` is the class responsible for loading, unloading and saving assets.

Remark: 1) **Arbitary binary assets** can be importes as raw assets; 2) Not sure whether **writing** is permitted; 3) Not sure the recommended way to interact directly with **arbitary on-disk files** (e.g. unpackaged/unbuilt files) - maybe just through regular C# methods

It's also always a good practice to unload resources that are not longer used to avoid wasting **GPU memory**:

```c#
To unload an asset, use Content.Unload(myAsset).
```

More examples:

```c#
// Load an asset directly from a file:
var texture = Content.Load<Texture>("texture1");

// Load a Scene asset
var scene = Content.Load<Scene>("scenes/scene1");

// Load an Entity asset
var entity = Content.Load<Entity>("entity1");
```

Remark: Asset load and unload are working in pairs. For each call to `load`, a corresponding call to `unload` is expected. An asset is loaded only **during the first call** to `load`. All subsequent calls only result to an **asset reference increment**. An asset is unload only when the number of call to unload match the number of call the load (i.e. reference count = 0).

More Example:

```c#
The 'Xenko.Core.Serialization.Assets.AssetManager.Get' method returns the reference to a loaded asset but does not increment the asset reference counter.

var firstReference = Content.Load<Texture>("MyTexture"); // load the asset and increase the reference counter (ref count = 1)

// the texture can be used here
var secondReference = Content.Load<Texture>("MyTexture"); // only increase the reference counter (ref count = 2)

// the texture can still be used here
Asset.Unload(firstReference); // decrease the reference counter (ref count = 1)

// the texture can still be used here
Asset.Get<Texture>("MyTexture"); // return the loaded asset without increasing the reference counter (ref count = 1)

// the texture can still be used here
Asset.Unload(secondReference); // decrease the reference counter and unload the asset (ref count = 0)

// The texture has been unloaded, it cannot be used here any more.
```

1. Set window to borderless

```c#
Add this line to any StartupScript after base.Start():
Game.Window.IsBorderLess = true;
```

RemarK: Notice one way to set window fullscreen is to reset backbuffer, search "enter fullscreen" in this page.

Remark: Other window related settings:

```c#
Game.Window.AllowUserResizing = true;
```

2. \[Physics\] Show colliders at runtime

```c#
// Intended for Debug Use
this.GetSimulation().ColliderShapesRendering = true;
```

Remark: **Collider shapes** for infinite planes are always invisible.

Remark: Notice **types** of physics/collision objects - **Kinematic objects** (animated, script and input controlled), **Kinematic triggers**, **Rigidbody colliders** (Physics simulation), **Rigidbody triggers**, **Static colliders**, **Static triggers**; Those objects *interact with each other*.

3. \[Physics\] Move a Static Collider

```c#
PhysicsComponent.Entity.Transform.Position += PhysicsComponent.Entity.Transform.Position + Vector3.UnitX;
PhysicsComponent.Entity.Transform.UpdateWorldMatrix();
PhysicsComponent.UpdatePhysicsTransformation();
```

Remark: Static collider provides **collision** and doesn't participate in **physics simulation** (notice collision and physics are interconnected but two seperate systems) but can be **moved**.

4. \[Physics\] Add a Constraint

```c#
// Link rigid body to the world at its current location
CreateConstraint(ConstraintTypes type, RigidbodyComponent rigidBodyA, Matrix frameA, bool useReferenceFrameA);

// Link two rigid bodies
CreateConstraint(ConstraintTypes type, RigidbodyComponent rigidBodyA, RigidbodyComponent rigidBodyB, Matrix frameA, Matrix frameB, bool useReferenceFrameA)

//After creating a constraint, add it to the simulation from a script by calling
this.GetSimulation().AddConstraint(constraint);

// Or equivalently call this:
var disableCollisionsBetweenLinkedBodies = true;
this.GetSimulation().AddConstraint(constraint, disableCollisionsBetweenLinkedBodies);

// To remove a constraint from the simulation, use:
this.GetSimulation().RemoveConstraint(constraint);
```

Remark: Constraints can either link two **rigidbodies** together, or link a single rigidbody to a **point in the world**.

Remark: Constraints can only be added in scripts.

5. \[Physics/Collision\] Cast a Ray from the **mouse/cursor's screen position**

```c#
// To get and set the Simulation instance properties
this.GetSimulation().ColliderShapesRendering = true;

// To get camera component, either assign in Game Studio assign a public property, or obtain reference through the owning Entity (if we just need transformation information then we can use the entity itself if the entity contains only a camera component and the script), or use below:


public static bool ScreenPositionToWorldPositionRaycast(Vector2 screenPos, CameraComponent camera, Simulation simulation)
{
    Matrix invViewProj = Matrix.Invert(camera.ViewProjectionMatrix);

    // Reconstruct the projection-space position in the (-1, +1) range.
    //    Don't forget that Y is down in screen coordinates, but up in projection space
    Vector3 sPos;
    sPos.X = screenPos.X * 2f - 1f;
    sPos.Y = 1f - screenPos.Y * 2f;

    // Compute the near (start) point for the raycast
    // It's assumed to have the same projection space (x,y) coordinates and z = 0 (lying on the near plane)
    // We need to unproject it to world space
    sPos.Z = 0f;
    var vectorNear = Vector3.Transform(sPos, invViewProj);
    vectorNear /= vectorNear.W;

    // Compute the far (end) point for the raycast
    // It's assumed to have the same projection space (x,y) coordinates and z = 1 (lying on the far plane)
    // We need to unproject it to world space
    sPos.Z = 1f;
    var vectorFar = Vector3.Transform(sPos, invViewProj);
    vectorFar /= vectorFar.W;

    // Raycast from the point on the near plane to the point on the far plane and get the collision result
    var result = simulation.Raycast(vectorNear.XYZ(), vectorFar.XYZ());
    return result.Succeeded;
}

// To get the current position of mouse cursor
Use Input.MousePosition or Input.AbsoluteMousePosition
// MousePosition returns the mouse pointer position in normalized X, Y coordinates instead of actual screen sizes in pixels.
// Returns the mouse pointer position in absolute X and Y coordinates (the actual screen size in pixels)
// Mouse position from top-left to bottom-right: (0,0) -> (+X,+Y)

// To get the size of screen, use
var surfaceSize = Input.Mouse.SurfaceSize;
```

Remark: Dpending on context, this operation can be **quite cheap** to perform.

Remark: Inside **Game Settings**, physics initialization can be set to **Collision** only.

Remark: Raytracing is used for - check which objects are in a gun's **line of fire** (or character **line of sight**), or are under the **mouse cursor** when the user clicks, or the **surface** underwhich the player is step onto.

Remark: Raycasting uses **colliders** to calculate intersections. It *ignores entities* that have no collider component.

6. \[Camera\] Create a camera and assign a camera slot from a script

```c#
var camera = new CameraComponent();
camera.Slot = SceneSystem.GraphicsCompositor.Cameras[0].ToSlotId();
```

Remark: Always have at least one enabled camera

Remark: Don't have multiple cameras enabled and assigned to the same slot at the same time

Reference: [Camera Slots](https://doc.xenko.com/latest/en/manual/graphics/cameras/camera-slots.html)

7. \[Input\] Detect Mouse Movement Since Last Frame

```c#
public class MouseInputScript : SyncScript
{
    public override void Update()
    {
        // If the left mouse button is pressed in this update, do something.
        if (Input.IsMouseButtonDown(MouseButton.Left))
        {   
        }
        // If the middle mouse button has been pressed since the last update, do something.
        if (Input.IsMouseButtonPressed(MouseButton.Middle))
        {  
        }
        //If the mouse moved more than 0.2 units of the screen size in X direction, do something.
        if (Input.MouseDelta.X > 0.2f)
        {
        }
    }
}
```

6. \[Physics; Collision\] Create a Static Collision Trigger That Changes Scale when collided

```c#
Create a static collider trigger then use following Aync Script to detect events (through polling):

using Xenko.Engine;
using Xenko.Physics;
using System.Threading.Tasks;
using Xenko.Core.Mathematics;

namespace TransformTrigger
{
    public class Trigger : AsyncScript
    {
        public override async Task Execute()
        {
            var trigger = Entity.Get<PhysicsComponent>();
            trigger.ProcessCollisions = true;

            // Start state machine
            while (Game.IsRunning)
            {
                // 1. Wait for an entity to collide with the trigger
                var firstCollision = await trigger.NewCollision();

                // Get collider
                var otherCollider = trigger == firstCollision.ColliderA
                    ? firstCollision.ColliderB
                    : firstCollision.ColliderA;
                otherCollider.Entity.Transform.Scale = new Vector3(2.0f, 2.0f, 2.0f);   // new scale

                // 2. Wait for the entity to exit the trigger
                Collision collision;
                do
                {
                    collision = await trigger.CollisionEnded();
                }
                while (collision != firstCollision);    // Wait for corresponding collision event

                // Set property
                otherCollider.Entity.Transform.Scale= new Vector3(1.0f, 1.0f, 1.0f);
            }
        }
    }
}
```

Remark: Notice instead of **subscribe** to some **collision event**, we use an **Async Script** to **poll** the event.

7. \[Rendering; Material\] Switch Material of Entity

```c#
private Material material1;
private Material material2;

// Make sure the materials are loaded 
material1 = Content.Load<Material>("Sphere Material");
material2 = Content.Load<Material>("Ground Material");

// Change the material on the entity
var trigger = Entity.Get<PhysicsComponent>(); // Get a component of the attached entity for the script
trigger.Entity.Get<ModelComponent>().Materials[0] = material2;  // Get entity through component then get a different component
```

8. \[Rendering; Lighting; Texture\] Changes Skybox Light and its intensity

```c#
public Skybox skybox;
public void ChangeSkyboxParameters()
{
    // Get the light component from an entity
    var light = Entity.Get<LightComponent>();

    // Get the Skybox Light settings from the light component
    var skyboxLight = light.Type as LightSkybox;

    // Replace the existing skybox
    skyboxLight.Skybox = skybox;

    // Change the skybox light intensity
    light.Intensity = 1.5f;
}
```

8. \[Rendering; Lighting; Texture\] Change the skybox at runtime

```
The following code changes the cubemap in a background:

public Texture cubemapTexture;
public void ChangeBackgroundParameters()
{
    // Get the background component from an entity
    var background = directionalLight.Get<BackgroundComponent>();

    // Replace the existing background
    background.Texture = cubemapTexture;

    // Change the background intensity
    background.Intensity = 1.5f;
}
```

8. \[Physics\] Detect trigger collisions

```
// Wait for an entity to collide with the trigger
var firstCollision = await trigger.NewCollision();
var otherCollider = trigger == firstCollision.ColliderA ? firstCollision.ColliderB : firstCollision.ColliderA;

// Alternatively, directly access the TrackingHashSet:
var trigger = Entity.Get<PhysicsComponent>();
foreach (var collision in trigger.Collisions)
{
    //do something with the collision
}

// Or use the TrackingHashSet events:
var trigger = Entity.Get<PhysicsComponent>();
trigger.Collisions.CollectionChanged += (sender, args) =>
{
    if (args.Action == NotifyCollectionChangedAction.Add)
    {
        //new collision
        var collision = (Collision) args.Item;
        //do something
    }
    else if (args.Action == NotifyCollectionChangedAction.Remove)
    {
        //old collision
        var collision = (Collision)args.Item;
        //do something
    }
};
```

9. \[UI\] Assign a **UI page** to a **UI page component** in code

```
// This property can be assigned from a UI page asset in Game Studio
public UIPage MyPage { get; set; }

protected override void LoadScene()
{
    InitializeUI();
}

public void InitializeUI()
{
    var rootElement = MyPage.RootElement;
    // to look for a specific element in the UI page, extension methods can be used
    var button = rootElement.FindVisualChildOfType<Button>("buttonOk");

    // if there's no element named "buttonOk" in the UI tree or the type doesn't match,
    // the previous method returns null
    if (button != null)
    {
        // attach a delegate to the Click event
        button.Click += delegate
        {
            // do something here...
        };
    }

    // assign the page to the UI component
    var uiComponent = Entity.Get<UIComponent>();
    uiComponent.Page = MyPage;
}
```


9. \[UI\] Assign a UI library in code

```
// This property can be assigned from a UI library asset in Game Studio
public UILibrary MyLibrary { get; set; }

public Button CreateButton()
{
    // assuming there is a root element named "MyButton" of type (or derived from) Button
    var button = MyLibrary.InstantiateElement<Button>("MyButton");

    // if there is no root named "MyButton" in the library or the type does not match,
    // the previous method will return null
    if (button != null)
    {        
        // attach a delegate to the Click event
        someButton.Click += delegate
        {
            // do something here...
        };
    }

    return button;
}
```

Remark: **UI pages** have only one root element. **UI libraries** can have multiple root elements.

9. \[UI\] Dynamically 

```
/// <summary>
/// Spritesheet containing the sprites of the main scene. (UI Type)
/// </summary>
public SpriteSheet MainSceneImages { get; set; }

private readonly List<int> starSpriteIndices = new List<int>();
protected override void LoadScene()
{
    // Preload stars
    starSpriteIndices.Add(MainSceneImages.FindImageIndex("star0"));
    starSpriteIndices.Add(MainSceneImages.FindImageIndex("star1"));
    starSpriteIndices.Add(MainSceneImages.FindImageIndex("star2"));
    starSpriteIndices.Add(MainSceneImages.FindImageIndex("star3"));

    // Initialize UI
    page = Entity.Get<UIComponent>().Page;
    // ...
    InitializeShipSelectionPopup();
    InitializeWelcomePopup();

    // Add pop-ups to the overlay
    var overlay = (UniformGrid) page.RootElement;
    overlay.Children.Add(shipSelectPopup);
    overlay.Children.Add(welcomePopup);

    Script.AddTask(FillLifeBar);
}

private TextBlock bonusCounter;
bonusCounter = rootElement.FindVisualChildOfType<TextBlock>("bonusCounter");
```

10. \[Video\] Play a Video on Surface of Mesh

```
// After you set up the video component, play it from a script using:
myVideoComponent.Instance.Play();

// Example
public class VideoScript : StartupScript
{
    // Game Studio displays the public member fields and properties you declare in this script

    public override void Start()
    {
        // Initialization of the script.
        Entity.Get<VideoComponent>().Instance.Play();
    }
}
```

Remark: Video component can be put on arbitary entity, just like **UI** or **scripts** - they are not bound to the owner script but rather exist in world.

Remark: Video component has a **Target Texture** setting, models that use this texture will display the video.

Remark: When the video isn't playing, Xenko displays the texture instead.

Remark: For any given video, its **video** and **sound** are imported as seperate assets; But played together.

10. \[Game System Scripting\] Event Broading Casting and Receiving

```
// Create an event
public static class GlobalEvents
{
    public static EventKey GameOverEventKey = new EventKey("Global", "Game Over");
    public static EventKey<string> GameOverWithDataEventKey = new EventKeyKey<string>("Global", "Game Over With Data");

    public static void SendGameOverEvent()
    {
        GameOverEventKey.Broadcast();   // Non-Generic parameter-less event
        GameOverWithDataEventKey.Broadcast("Player 1"); // Generic event with parameter
    }
}

// Receive/Listen to an event
var gameOverListener = new EventReceiver(GlobalEvents.GameOverEventKey);
var gameIsOver = gameOverListener.TryReceive();

var gameOverListenerWithData = new EventReceiver<string>(GlobalEvents.GameOverWithDataEventKey);
if(gameOverListenerWithData.TryReceive(out string data))
{
    //data == "Player 1"
}

// Or in Async
await gameOverListener.ReceiveAsync();
string asyncData = await gameOverListenerWithData.ReceiveAsync();
// asyncData == "Player 1"
```

Remark: Event broading casting is one way from sender to all scripts; Listeners will catch the event and respond to it.

12. \[Script\] Per Frame Update Object Property in Real Time (rather than framerate dependent)

```
public override void Update()
{
    var elapsedTime = (float)Game.UpdateTime.Elapsed.TotalSeconds;
    var moveDist = GameSpeed * elapsedTime;
}
```

13. \[Rendering; Shader\] Create Parametrically Controlled Shader

```
// First create a custom shader for the slot of material; See Xenko Shading Language Reference.
// To modify the value at runtime, access and set it in the material parameter collection. For example, to change the Frequency, use:
myMaterial.Passes[myPassIndex].Parameters.Set(ComputeColorWaveKeys.Frequency, MyFrequency);
```

13. \[Input\] Detect Keyboard Inputs

```
public class KeyboardEventsScript : SyncScript
{
    public override void Update()
    {
        //Perform an action in every update.
        if (Game.IsRunning)
        {
            if (Input.IsKeyDown(Keys.Left)) // Or equivalently: Use InputManager.DownKeys
            {
                this.Entity.Transform.Position.X -= 0.1f;
            }
            if (Input.IsKeyDown(Keys.Right))
            {
                this.Entity.Transform.Position.X += 0.1f;
            }
        }
    }
}
```

14. \[Input\] Create virtual buttons

```
public override void Start()
{
    base.Start();

    // Create a new VirtualButtonConfigSet if none exists. 
    Input.VirtualButtonConfigSet = Input.VirtualButtonConfigSet ?? new VirtualButtonConfigSet();

    // Bind "M" key, GamePad "Start" button and left mouse button to a virtual button "MyVButton".
    VirtualButtonBinding b1 = new VirtualButtonBinding("MyVButton", VirtualButton.Keyboard.M);
    VirtualButtonBinding b2 = new VirtualButtonBinding("MyVButton", VirtualButton.GamePad.Start);
    VirtualButtonBinding b3 = new VirtualButtonBinding("MyVButton", VirtualButton.Mouse.Left);

    // Define behavior for the virtual button
    VirtualButtonConfig c = new VirtualButtonConfig();
    c.Add(b1);
    c.Add(b2);
    c.Add(b3);

    // Add virtual button
    Input.VirtualButtonConfigSet.Add(c);
}

public override void Update() {
    // Use virtual button
    float button = Input.GetVirtualButton(0, "MyVButton");  // Equivalent as, for example: Input.
}
```

11. \[UI; Scene; Scene Management\] Load next Scene

```
public class SplashScript: SyncScript
{
    public override void Start()
    {
        // Next scene
        SceneSystem.SceneInstance.RootScene = Content.Load<Scene>("MainScene");
    }
}
```

Remark: Get game instance

```
protected Game UIGame;

public abstract class MySync : SyncScript
{
    public override void Start()
    {
        UIGame = (Game)Services.GetServiceAs<IGame>();
    }
}
```

12. \[Scene; Scene Management\] Load and Unload Scenes at Runtime

```
// The following code loads three scenes and adds them as children:
var myChildScene0 = Content.Load<Scene>(url0);
var myChildScene1 = Content.Load<Scene>(url1);
var myChildScene2 = Content.Load<Scene>(url2);

myParentScene.Children.Add(myChildScene0);
myParentScene.Children.Add(myChildScene1);
myChildScene1.Add(myChildScene2);

// To get the parent scene at first place:

```

Remark: Make sure all the **scenes** you want to load are included in the build as **root assets** in **Game Studio** **asset view** (otherwise they won't be packaged).

Remark: When creating a new asset, a **Scene Streaming Script** is provided to steam a scene through trigger.

Reference: [Use the scene streaming script](https://doc.xenko.com/latest/en/manual/game-studio/load-scenes.html#use-the-scene-streaming-script)

15. \[UI\] Adjust UI Backbuffer Size

```
var backBufferSize = new Vector2(GraphicsDevice.Presenter.BackBuffer.Width, GraphicsDevice.Presenter.BackBuffer.Height);
Entity.Get<UIComponent>().Resolution = new Vector3(backBufferSize, 1000);
```

16. \[Scene Management; Global State; Entity Management\] Get entity by name; Get entity name

```
// Add an entity:
var myEntity = new Entity();
engine.EntityManager.AddEntity(myEntity);

// Iterate through added entities:
foreach (var entity in engine.EntityManager.Entities)
{
    Console.WriteLine(entity.Name);
}
```

Remark: Entity are grouped together in an `EntityManager`.

17. \[Subscene; Scene Management\]

```
// Move a scene at runtime
myScene.Offset = new Vector3(x, y, z);
```

Remark: Move a **scene** using its **offset** property. Scenes aren't **entities**, they don't have **transform components**.

Reference: [Manage Scenes](https://doc.xenko.com/latest/en/manual/game-studio/manage-scenes.html)

18. \[UI; Rendering\] Enter Fullscreen (Unresolved)

```
// Do this in startup code
if (GoFullscreen)
{
    // Obtain current (physical) display spec (thus through a GraphicsAdapter, which represents specs of real device)
    var gfxOutput = GraphicsAdapterFactory.Adapters[0].Outputs;
    var displayMode = gfxOutput[0].CurrentDisplayMode;
    // Compute max dimensions (Bound by 1920x1080)
    var screenWidth = Math.Min(displayMode.Width, 1920);
    var maxHeight = displayMode.AspectRatio < 1.7f ? 1200 : 1080;
    var screenHeight = Math.Min(displayMode.Height, maxHeight);

    // Below lines doesn't work in 3.1: Game.GraphicsDeviceManager doesn't exist, see next code snippet for solution
    // Set target screen resolution and enter fullscreen
    Game.GraphicsDeviceManager.PreferredBackBufferWidth = screenWidth;
    Game.GraphicsDeviceManager.PreferredBackBufferHeight = screenHeight;
    Game.GraphicsDeviceManager.IsFullScreen = true; // Either use this or use Game.Window.IsBorderLess = true; See next next code snippet
    Game.GraphicsDeviceManager.ApplyChanges();
}
```

```
// Use this way to get GraphicsDeviceManager
if (Services.GetService<IGraphicsDeviceManager>() is GraphicsDeviceManager gdm)
{
    Game.Window.BeginScreenDeviceChange(true);    // Enter native fullscreen
    gdm.PreferredBackBufferWidth = width;
    gdm.PreferredBackBufferHeight = height;
    gdm.IsFullScreen = true;
    gdm.ApplyChanges();                
    Game.Window.EndScreenDeviceChange();
    // Someone suggested there is a bug here.  If you don't set the window visibility to true
    // In full screen mode mouse input does not work. i.e. Input.MousePosition does not update.
    // If you click the mouse you go back to the desktop and the game is still running but
    // does not appear in task manager or on the task bar etc. And if you are switching back to windowed mode
    // then the window is not visible. Does not even apear in task manager.
    // So always setting window to visible.
    Game.Window.Visible = true;
}
```

```
// Minimum code to enter borderless fullscreen with screen centered properly
public override void Start()
{
    if(true)
    {
        if (Services.GetService<IGraphicsDeviceManager>() is GraphicsDeviceManager gdm)
        {
            // Set border and position here first
            Game.Window.IsBorderLess = true;
            Game.Window.Position = new Int2(0);
            // Set backbuffer, without specifying "native fullscreen" here
            Game.Window.BeginScreenDeviceChange(false);
            gdm.PreferredBackBufferWidth = screenWidth;
            gdm.PreferredBackBufferHeight = screenHeight;
            gdm.ApplyChanges();
            Game.Window.EndScreenDeviceChange();
        }
    }
}
```

Remark: Also use `Alt+Enter` to enter fullscreen, notice this will stretch the UI.

Remark: To properly handle fullscreen, use 1) window resize event to handle UI redraw; 2) *Very likely (or not) there is a layout based, absolute/relative pixel resolution way of laying out UIs and scene elements*.

19. \[Animation; Skeletal Animation\] Move the knight arm; Move bone from code; Animate skeleton in code

```
Working
var modelComponent = Entity.Get<ModelComponent>();

for (int nodeIndex = 0; nodeIndex < modelComponent.Model.Skeleton.Nodes.Length; nodeIndex++)
{
    var nodeName = modelComponent.Skeleton.Nodes[nodeIndex].Name;

    if (nodeName.Contains("Hand"))
    {
        modelComponent.Skeleton.NodeTransformations[nodeIndex].Transform.Rotation *= Quaternion.RotationX(rotationSpeed * elapsedTime * 4.0f);
    }
}
```

```
// Not working
var modelComponent = Entity.Get();
var transformComponent = Entity.Get();
var mvhto = new ModelViewHierarchyTransformOperation(modelComponent);

foreach (var mesh in modelComponent.Model.Meshes)
{
    var nodeIndex = mesh.NodeIndex;

    if(mesh.Name.Contains("Arm"))
    {
        var node1 = modelComponent.Skeleton.Nodes[nodeIndex];
        node1.Transform.Rotation *= Quaternion.RotationY(rotationSpeed * elapsedTime); 
        mvhto.Process(transformComponent);
    }
}`
```

Remark: Notice this doesn't work if animation is playing, in which case it will overwrite any values that you set on any property that has animation.

Notice: Notice there is a way to **bind objects** to **skeleton** bones using (Model Node Links)[https://doc.xenko.com/latest/jp/manual/animation/model-node-links.html]

Reference: (This forum post)[https://forums.xenko.com/t/just-want-to-move-the-knight-arm/535]

20. \[Animation; Skeletal Animation\] Setup Animation; Play default animation

```
public class SimpleAnimationScript : StartupScript
{
    public override void Start()
    {
        Entity.Get<AnimationComponent>().Play("Walk");
    }
}
```

Reference: See (Setup Animation)[https://doc.xenko.com/latest/jp/manual/animation/set-up-animations.html]

21. \[Animation; Property Animation\] Create property procedural animation

```
// Instead of updating the property manually every frame, we could create an animation curve
// This snippet is operating on TransformComponent; LightComponent and RigidBodyComponent can also be animated this way
public class AnimationScript : StartupScript
{
    public override void Start()
    {
        // Create an AnimationClip. Make sure you set its duration properly.
        var animationClip = new AnimationClip { Duration = TimeSpan.FromSeconds(1) };

        // Add a curves specifying the path to the transformation property.
        // - You can index components using a special syntax to their key.
        // - Properties can be qualified with a type name in parenthesis.
        // - If a type isn't serializable, its fully qualified name must be used.

        animationClip.AddCurve("[TransformComponent.Key].Rotation", CreateRotationCurve());

        // Optional: pack all animation channels into an optimized interleaved format.
        animationClip.Optimize();

        // Add an AnimationComponent to the current entity and register our custom clip.
        const string animationName = "MyCustomAnimation";
        var animationComponent = Entity.GetOrCreate<AnimationComponent>();
        animationComponent.Animations.Add(animationName, animationClip);

        // Play the animation right away and loop it.
        var playingAnimation = animationComponent.Play(animationName);
        playingAnimation.RepeatMode = AnimationRepeatMode.LoopInfinite;
        playingAnimation.TimeFactor = 0.1f; // slow down
        playingAnimation.CurrentTime = TimeSpan.FromSeconds(0.6f); // start at different time
    }

    // Set custom linear rotation curve.
    private AnimationCurve CreateRotationCurve()
    {
        return new AnimationCurve<Quaternion>
        {
            InterpolationType = AnimationCurveInterpolationType.Linear,
            KeyFrames =
            {
                CreateKeyFrame(0.00f, Quaternion.RotationX(0)),
                CreateKeyFrame(0.25f, Quaternion.RotationX(MathUtil.PiOverTwo)),
                CreateKeyFrame(0.50f, Quaternion.RotationX(MathUtil.Pi)),
                CreateKeyFrame(0.75f, Quaternion.RotationX(-MathUtil.PiOverTwo)),
                CreateKeyFrame(1.00f, Quaternion.RotationX(MathUtil.TwoPi))
            }
        };
    }

    private static KeyFrameData<T> CreateKeyFrame<T>(float keyTime, T value)
    {
        return new KeyFrameData<T>((CompressedTimeSpan)TimeSpan.FromSeconds(keyTime), value);
    }
}
```


See: (Procedural Animation)[https://doc.xenko.com/latest/en/manual/animation/procedural-animation.html]

22. \[Animation; Skeletal Animation\] Custom Blend Tree
```c#
public class AnimationBlendTree : SyncScript, IBlendTreeBuilder
{
    /// <summary>
    /// The animation component is required
    /// </summary>
    [Display("Animation Component")]
    public AnimationComponent AnimationComponent { get; set; }

    [Display("Walk")]
    public AnimationClip AnimationWalk { get; set; }

    [Display("Run")]
    public AnimationClip AnimationRun { get; set; }

    [Display("Lerp Factor")]
    public float LerpFactor = 0.5f;

    private AnimationClipEvaluator animEvaluatorWalk;
    private AnimationClipEvaluator animEvaluatorRun;
    private double currentTime = 0;

    public override void Start()
    {
        base.Start();

        // IMPORTANT STEP
        // By setting a custom blend tree builder we can override the default behavior of the animation system.
        // Instead, BuildBlendTree(FastList<AnimationOperation> blendStack) will be called each frame.
        // We need to update the animation state in Update() and then
        // pass the new animation state (stack = blend tree) to the animation system.
        AnimationComponent.BlendTreeBuilder = this;

        // As we override the animation system, we need to create an AnimationClipEvaluator for each clip we want to use.
        animEvaluatorWalk = AnimationComponent.Blender.CreateEvaluator(AnimationWalk);
        animEvaluatorRun = AnimationComponent.Blender.CreateEvaluator(AnimationRun);
    }

    public override void Cancel()
    {
        // When the script is cancelled, don't forget to release all animation resources created in Start() - AnimationClipEvaluators
        AnimationComponent.Blender.ReleaseEvaluator(animEvaluatorWalk);
        AnimationComponent.Blender.ReleaseEvaluator(animEvaluatorRun);
    }

    public override void Update()
    {
        // Use DrawTime rather than UpdateTime because the animations are updated only when they are drawn.
        var time = Game.DrawTime;

        // This update function accounts for animation with different durations,
        // keeping a current time relative to the blended maximum duration.
        long blendedMaxDuration = (long)MathUtil.Lerp(AnimationWalk.Duration.Ticks, AnimationRun.Duration.Ticks, LerpFactor);

        var currentTicks = TimeSpan.FromTicks((long)(currentTime * blendedMaxDuration));

        currentTicks = blendedMaxDuration == 0
            ? TimeSpan.Zero
            : TimeSpan.FromTicks((currentTicks.Ticks + (long)(time.Elapsed.Ticks)) % blendedMaxDuration);

        currentTime = ((double)currentTicks.Ticks / (double)blendedMaxDuration);
    }

    /// BuildBlendTree is called every frame from the animation system when the AnimationComponent needs to be evaluated.
    /// It overrides the default behavior of the AnimationComponent by setting a custom blend tree.
    public void BuildBlendTree(FastList<AnimationOperation> blendStack)
    {
        var timeWalk = TimeSpan.FromTicks((long) (currentTime * AnimationWalk.Duration.Ticks));
        var timeRun  = TimeSpan.FromTicks((long) (currentTime * AnimationRun.Duration.Ticks));

        // Build the animation blend tree (stack)
        blendStack.Add(AnimationOperation.NewPush(animEvaluatorWalk, timeWalk));    // Will PUSH animation state to be evaluated at the specified Time.
        blendStack.Add(AnimationOperation.NewPush(animEvaluatorRun, timeRun));      // Will PUSH another animation state to be evaluated at the specified Time.
        blendStack.Add(AnimationOperation.NewBlend(AnimationBlendOperation.LinearBlend, LerpFactor));   // Will POP the last two states, blend them with the factor and PUSH back the result.

        // NOTE
        // Because the blending operations are laid out in a stack you have to pack the operations in this manner.
        // In general, traversing a binary tree depth-first and adding operations as you *leave* precessed nodes should be sufficient.
        // For non-binary trees, you have to properly weight the blending factors as well

        // DONE
        // The top of the stack now contains the final state used for the animated model
    }
}
```

Remark: The **AnimationComponent** has the property `AnimationComponent.BlendTreeBuilder`.

Remark: If you want absolute control over which animations are played, how are they blended and what weights they have, you can create a script which inherits from IBlendTreeBuilder and assign it to the BlendTreeBuilder under your animation component. When the animation component is updated, it calls void BuildBlendTree(FastList<AnimationOperation> animationList) on your script instead of updating the animations itself. This allows you to choose any combination of animation clips, speeds and blends, but is also more difficult, as all the heavy lifting is now on the script side.

Remark: The templates First-person shooter, Third-person platformer and Top-down RPG, included with Xenko, are examples of how to use custom blend trees.

See: (Custom Blend Tree)[https://doc.xenko.com/latest/jp/manual/animation/custom-blend-trees.html]


23. \[Animation; Skeletal Animation\] Control custom attributes with a script

```
See (Control custom attributes with a script)[https://doc.xenko.com/latest/en/manual/animation/custom-attributes.html#2-control-custom-attributes-with-a-script]
```

24. \[Animation; Skeletal Animation\] Example Animation Script

```c#
// This sample script assigns a simple animation to a character based on its walking speed
using Xenko.Engine;

namespace AdditiveAnimation
{
    public class AnimationClipExample : SyncScript
    {
        public float MovementSpeed { get; set; } = 0f;

        private float walkingSpeedLimit = 1.0f;

        // Assuming the script is attached to an entity which has an animation component
        private AnimationComponent animationComponent;

        public override void Start()
        {
            // Cache some variables we'll need later
            animationComponent = Entity.Get<AnimationComponent>();
            animationComponent.Play("Idle");
        }

        protected void PlayAnimation(string name)
        {
            if (!animationComponent.IsPlaying(name))
                animationComponent.Play(name);
        }

        public override void Update()
        {
            if (MovementSpeed <= 0)
            {
                PlayAnimation("Idle");
            }
            else if (MovementSpeed <= walkingSpeedLimit)
            {
                PlayAnimation("Walk");
            }
            else 
            {
                PlayAnimation("Run");
            }
        }
    }
}
```

Remark: Xenko includes a pre-built AnimationStart script that can be used as a template to write your own animation scripts. Create it when create new asset choosing Add asset > Scripts > Animation start.

See: (Animation Scripts)[https://doc.xenko.com/latest/jp/manual/animation/animation-scripts.html]

22. \[UI, Font\] Create Texts using Fonts from Script

```c#
//  Load a SpriteFont
var myFont = Content.Load<SpriteFont>("MyFont");

// Write text on screen

// create the SpriteBatch
var spriteBatch = new SpriteBatch(GraphicsDevice);
// don't forget the begin
spriteBatch.Begin(GraphicsContext);
// draw the text "Helloworld!" in red from the center of the screen
spriteBatch.DrawString(myFont, "Helloworld!", new Vector2(0.5, 0.5), Color.Red);
// don't forget the end
spriteBatch.End();

// Advanced text drawing: Draw the text "Hello world!" upside-down in red from the center of the screen
spriteBatch.Begin(GraphicsContext);
spriteBatch.DrawString(myFont, "Hello world!", new Vector2(0.5, 0.5), Color.Red, 0, new Vector2(0, 0), new Vector2(1,1), SpriteEffects.FlipVertically, 0);
spriteBatch.End();
```


Remark: After a font asset is **compiled** it can be loaded as a `SpriteFont` instance using the `Xenko.Core.Serialization.Assets.ContentManager`. It contains all the options to display a text (**bitmaps**, **kerning**, **line spacing** etc).

Remark: Once the font is loaded, you can display any text with a `SpriteBatch`. The `Xenko.Graphics.SpriteBatch.DrawString` method performs the draw.

Remark: The various overloads of `DrawString` let you specify the text's **orientation**, **scale**, **depth**, **origin**, etc. You can also apply some `SpriteEffects` to the text (*None, FlipHorizontally, FlipVertically, FlipBoth*).

Reference: (SpriteBatch)[https://doc.xenko.com/latest/en/manual/graphics/low-level-api/spritebatch.html]

Reference (Sprite Fonts)[https://doc.xenko.com/latest/en/manual/graphics/sprite-fonts.html]

23. \[2D; UI; Animation; Sprite\] Use sprites in a script

```c#
// This script displays a sprite that advances to the next sprite in the index every second. After it reaches the end of the sprite index, it loops.

using Xenko.Rendering.Sprites;

public class Animation : SyncScript
{
   // Declared public member fields and properties are displayed in Game Studio.

   // Private members
   private SpriteFromSheet sprite;
   private DateTime lastFrame;

   public override void Start()
   {
       // Initialize the script.
       sprite = Entity.Get<SpriteComponent>().SpriteProvider as SpriteFromSheet;
       lastFrame = DateTime.Now;    // Using DateTime will suffice for now
   }

   public override void Update()
   {
      // Do something every new frame.
      if ((DateTime.Now - lastFrame) > new TimeSpan(0, 0, 1))
      {
         sprite.CurrentFrame += 1;
         lastFrame = DateTime.Now;
      }
   }
}
```

Remark: Attach the script to an entity with a sprite component.


24. \[Rendering, Camera\] Create a camera and assign a camera slot from a script

```c#
var camera = new CameraComponent();
camera.Slot = SceneSystem.GraphicsCompositor.Cameras[0].ToSlotId();
```

Reference: To define camera slots, see [this](https://doc.xenko.com/latest/jp/manual/graphics/cameras/camera-slots.html)

25 \[Rendering, Texturing \] Textures and render textures

```c#
// load a texture from an asset in Xenko
var myTexture = Content.Load<Texture>("duck");  // loads the texture called duck.dds (or .png etc.)
// This automatically generates a texture object with all its fields correctly filled.

// Create a texture without any assets (eg to be used as render target)
var myTexture = Texture.New2D(GraphicsDevice, 512, 512, false, PixelFormat.R8G8B8A8_UNorm, TextureFlags.ShaderResource);

// Create a custom render target and depth buffer
// render target: Don't forget the flag RenderTarget to enable the render target behavior.
var myRenderTarget = Texture.New2D(GraphicsDevice, 512, 512, false, PixelFormat.R8G8B8A8_UNorm, TextureFlags.ShaderResource | TextureFlags.RenderTarget);
// writable depth buffer: Make sure the PixelFormat is correct, especially for the depth buffer. Be careful of the available formats on the target platform.
var myDepthBuffer = Texture.New2D(GraphicsDevice, 512, 512, false, PixelFormat.D16_UNorm, TextureFlags.DepthStencil);

// Use a render target: Once these buffers are created, you can set them as current render textures.
// setting the render textures
CommandList.SetRenderTargetAndViewport(myDepthBuffer, myRenderTarget);
// setting the default render target
CommandList.SetRenderTargetAndViewport(GraphicsDevice.Presenter.DepthStencilBuffer, GraphicsDevice.Presenter.BackBuffer);
// You can set multiple render textures at the same time. See the overloads of SetRenderTargets(Texture, Texture[]) and SetRenderTargetsAndViewport(Texture, Texture[]) method.

// Clear a render target
CommandList.Clear(GraphicsDevice.Presenter.BackBuffer, Color.Black);
CommandList.Clear(GraphicsDevice.Presenter.DepthStencilBuffer, DepthStencilClearOptions.DepthBuffer); // only clear the depth buffer
// To clear render textures, call the Clear(Texture, Color4) and Clear(Texture, DepthStencilClearOptions, Single, Byte) methods.

// Set the viewports
// example of a full HD buffer
CommandList.SetRenderTarget(GraphicsDevice.Presenter.DepthStencilBuffer, GraphicsDevice.Presenter.BackBuffer); // no viewport set
// example of setting the viewport to have a 10-pixel border around the image in a full HD buffer (1920x1080)
var viewport = new Viewport(10, 10, 1900, 1060);
CommandList.SetViewport(viewport);
// You can bind multiple viewports using SetViewports(Viewport[]) and SetViewports(Int32, Viewport[]) overloads for use with a geometry shader.
// If you only want to render to a subset of the texture, set the render target and viewport separately using SetRenderTarget(Texture, Texture) and SetViewport(Viewport).
// SetRenderTargetAndViewport(Texture, Texture) adjusts the current Viewport to the full size of the render target.

// Set the scissor
// example of setting the scissor to crop the image by 10 pixel around it in a full hd buffer (1920x1080)
var rectangle = new Rectangle(10, 10, 1910, 1070);
CommandList.SetScissorRectangles(rectangle);
var rectangles = new[] { new Rectangle(10, 10, 1900, 1060), new Rectangle(0, 0, 256, 256) };
CommandList.SetScissorRectangles(rectangles);
// The SetScissorRectangle(Rectangle) and SetScissorRectangles(Rectangle[]) methods set the scissors. Unlike the viewport, you need to provide the coordinates of the location of the vertices defining the scissor instead of its size.
```

Reference: [Textures and render textures](https://doc.xenko.com/latest/jp/manual/graphics/low-level-api/textures-and-render-textures.html)

Remark: The **GraphicsPresenter** class always provides a **default render target** and a **depth buffer**. They are accessible through the `BackBuffer` and `DepthStencilBuffer` properties. 

Remark: The **presenter** is exposed by the `Presenter` property of the `GraphicsDevice`.

Remark: You might want to use your **own buffer** to perform **off-screen rendering** or **post-processes**. As a result, you can create textures that can act as **render textures** and a **depth buffers**.

Remark: Make sure both the **render target** and the **depth buffer** have the same size. Otherwise, the **depth buffer** isn't used.

Remark: Only the **BackBuffer** is displayed on screen, so you need to render it to display something.

Remark: Don't forget to clear the **BackBuffer** and the **DepthStencilBuffer** each frame (i.e. set to a constant value, e.g. white or black). If you don't, you might get unexpected behavior depending on the device. If you want to keep the contents of a frame, use an **intermediate render target**.

26. \[Rendering; Graphics Compositor\] Custom scene renderers

```c#
// Implement an ISceneRenderer
[DataContract("MyCustomRenderer")]
[Display("My Custom Renderer")]
public sealed class MyCustomRenderer : SceneRendererBase
{
    // Implements the DrawCore method
    protected override void DrawCore(RenderContext context, RenderFrame output)
    {
        // Access to the graphics device
        var graphicsDevice = context.GraphicsDevice;
        // Clears the currrent render target
        graphicsDevice.Clear(output.RenderTargets[0], Color.CornflowerBlue);
        // [...] 
    }
}

// Use a delegate
var sceneRenderer = new DelegateSceneRenderer(
    (renderContext, frame) =>
    {
        // Access to the graphics device
        var graphicsDevice = context.GraphicsDevice;
        // Clears the currrent render target
        graphicsDevice.Clear(output.RenderTargets[0], Color.CornflowerBlue);
        // [...] 
   });
// To develop a renderer and attach it to a method directly, use DelegateSceneRenderer
```


Remark: To create a **custom renderer**, directly implement the `ISceneRenderer` or use a delegate through the `DelegateSceneRenderer`.

Remark: The **SceneRendererBase** provides a default implementation of `ISceneRenderer`. It automatically binds the **output** defined on the renderer to the `GraphicsDevice` before calling the `DrawCore()` method.

27. \[Rendering; Graphics Compositor\] Use Debug Renderer

```c#
using System.Linq;
using System.Threading.Tasks;
using Xenko.Input;
using Xenko.Engine;
using Xenko.Physics;
using Xenko.Rendering;
using Xenko.Rendering.Compositing;

namespace MyGame
{
    public class DebugPhysicsShapes : AsyncScript
    {
        public RenderGroup RenderGroup = RenderGroup.Group7;

        public override async Task Execute()
        {
        //set up rendering in the debug entry point if we have it
        var compositor = SceneSystem.GraphicsCompositor;
        var debugRenderer =
            ((compositor.Game as SceneCameraRenderer)?.Child as SceneRendererCollection)?.Children.Where(
                x => x is DebugRenderer).Cast<DebugRenderer>().FirstOrDefault();
        if (debugRenderer == null)
            return;

        var shapesRenderState = new RenderStage("PhysicsDebugShapes", "Main");
            compositor.RenderStages.Add(shapesRenderState);
            var meshRenderFeature = compositor.RenderFeatures.OfType<MeshRenderFeature>().First();
            meshRenderFeature.RenderStageSelectors.Add(new SimpleGroupToRenderStageSelector
            {
                EffectName = "XenkoForwardShadingEffect",
                RenderGroup = (RenderGroupMask)(1 << (int)RenderGroup),
                RenderStage = shapesRenderState,
            });
            meshRenderFeature.PipelineProcessors.Add(new WireframePipelineProcessor { RenderStage = shapesRenderState });
            debugRenderer.DebugRenderStages.Add(shapesRenderState);

            var simulation = this.GetSimulation();
            if (simulation != null)
                simulation.ColliderShapesRenderGroup = RenderGroup;

            var enabled = false;
            while (Game.IsRunning)
            {
                if (Input.IsKeyDown(Keys.LeftShift) && Input.IsKeyDown(Keys.LeftCtrl) && Input.IsKeyReleased(Keys.P))
                {
                    if (simulation != null)
                    {
                        if (enabled)
                        {
                            simulation.ColliderShapesRendering = false;
                            enabled = false;
                        }
                        else
                        {
                            simulation.ColliderShapesRendering = true;
                            enabled = true;
                        }
                    }
                }

                await Script.NextFrame();
            }
        }
    }
}
```

Remark: To use the debug renderer, reference it in your **script** and add debug render stages (i.e. call related functions).

Remark: Xenko provides a **Debug physics shapes** script that uses the **debug renderer** to display **collider shapes** at runtime.

Remark: The **debug renderer** is a placeholder renderer you can use with scripts to print debug information. By default, the debug renderer is included in the **graphics compositor** as a child of the **game** *entry point*. 

Remark: Notice **entry points** are rendered according to **list order**.

Reference: [Debug Renderer](https://doc.xenko.com/latest/jp/manual/graphics/graphics-compositor/debug-renderers.html)

28. \[Scene\] Query/Search Scene Objects/Components
 
```c#
public class SolarSystemScript : SyncScript
{
    public override void Start()
    {
        // Search and initialize planet entities
        Planets = new List<ModelComponent>();
        foreach (Entity entity in SceneSystem.SceneInstance.RootScene.Entities)
        {
            // ...
        }
    }

    // ...
}
```

Remark: I haven't seen an official way (from doc) to do this

Remark: Notice **searching** in **runtime** can be more efficient than manually assign **object references** in Game Studio.
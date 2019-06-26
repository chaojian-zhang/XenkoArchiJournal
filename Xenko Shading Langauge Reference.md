Including conceptual review, operational details, architecture, and code snippets.

(Pending)[https://doc.xenko.com/latest/en/manual/graphics/effects-and-shaders/shading-language/index.html]

(Also)[https://doc.xenko.com/latest/en/manual/graphics/effects-and-shaders/effect-language.html]

(Effects)[https://doc.xenko.com/latest/en/manual/graphics/effects-and-shaders/index.html]
(Graphics Resource Binding)[https://doc.xenko.com/latest/en/manual/graphics/low-level-api/resources.html]

(Pipeline States)[https://doc.xenko.com/latest/en/manual/graphics/low-level-api/pipeline-state.html]

(Graphics compositor)[https://doc.xenko.com/latest/en/manual/graphics/graphics-compositor/index.html]

[Custom color transforms](https://doc.xenko.com/latest/en/manual/graphics/post-effects/color-transforms/custom-color-transforms.html)

# Shading Pipeline Overview, or Review

At least, the rendering engine need to collect all kinds of rendering primitives from scene objects before creating batches for rendering and properly setup rendering orders. This is affected by at least: material, rendering settings, particile effects, custom (Post-Processing) effects...


# Shading Customization and Manual Drawing and Procedural Generation

Those things can look complicated bacause: 1) They are dealing with binary data and thus rather low level; 2) hey are very specific; 3) A lot of computation is involved and it's hard to generalize.

## Primitives
Xenko provides the following set of built-in **primitives**:

1. Plane
2. Cube
3. Sphere
4. Geosphere
5. Cylinder
6. Torus
7. Teapot

Any geometry can be drawn by creating **custom vertex buffers**.  A **vertex declaration** is used to describes the elements of each vertex and their layout. See (VertexDeclaration)[https://doc.xenko.com/latest/en/api/Xenko.Graphics.VertexDeclaration.html] and (VertexElement)[https://doc.xenko.com/latest/en/api/Xenko.Graphics.VertexElement.html].

# Snippets

Snippets are always helpful, especially when they are collected together as below. Some necessary explanations, contexts, and term clarification should also be provided:

1. Binding and drawing vertex buffers

```
// Set the pipeline state
pipelineStateDescription.InputElements = vertexBufferBinding.Layout.CreateInputElements();
pipelineStateDescription.PrimitiveType = PrimitiveType.TriangleStrip;

// Create and set a PipelineState object
// ...

// Bind the vertex buffer to the pipeline
commandList.SetVertexBuffers(0, vertexBuffer, 0, vertexBufferBinding.Stride);

// Draw the vertices
commandList.Draw(vertexCount);
```

1. Create state objects

```
// Creating pipeline state object
var pipelineStateDescription = new PipelineStateDescription();
var pipelineState = PipelineState.New(GraphicsDevice, ref pipelineStateDescription);

// Applying the state to the pipeline
CommandList.SetPipelineState(pipelineState);    // CommandList is the container of sequence of states
```

Remark: The Xenko **graphics pipeline** ie divided into **states/stages**, including - **rasterizer state**, **depth and stencil state**, **blend state**, **effects** (post processing), **input layout** (???), **output description** (???).

Remark: Each state is represented as an immutable `PipelineState` obejct, containing parameters describing the state.

Reference: (Pipeline State Configurations Doc)[https://doc.xenko.com/latest/en/manual/graphics/low-level-api/pipeline-state.html]

2.  Creating and using a Primive

```
// creation
var myCube = GeometricPrimitive.Cube.New(GraphicsDevice);
var myTorus = GeometricPrimitive.Torus.New(GraphicsDevice);

// ...

// draw one on screen
myCube.Draw(CommandList, EffectInstance);
```

Remark: User has to provide an `EffectInstance` when drawing.

Reference: See (GraphicsDevice)[https://doc.xenko.com/latest/en/api/Xenko.Graphics.GraphicsDevice.html].

2. Create Mutable pipeline state

```
// Creating the pipeline state object
var mutablePipelineState = new MutablePipelineState();

// Setting values and rebuilding
mutablePipelineState.State.BlendState = BlendStates.AlphaBlend
mutablePipelineState.Update

// Applying the state to the pipeline
CommandList.SetPipelineState(mutablePipelineState.CurrentState);
```

RemarK: The `MutablePipelineState` class let you set states independently, while **caching** the underlying pipeline states.

2. Creating a vertex buffer

```
// Create a vertex layout with position and texture coordinate
var layout = new VertexDeclaration(VertexElement.Position<Vector3>(), VertexElement.TextureCoordinate<Vector2>()); 

// Create the vertex buffer from an array of vertices
var vertices = new VertexPositionTexture[vertexCount];
var vertexBuffer = Buffer.Vertex.New(GraphicsDevice, vertices);

// Create a vertex buffer binding
var vertexBufferBinding = new VertexBufferBinding(vertexBuffer, layout, vertexCount);
```

Reference: See (VertexBufferBinding)[https://doc.xenko.com/latest/en/api/Xenko.Graphics.VertexBufferBinding.html].

3. Using an EffectInstance

```
// Create a EffectInstance and use it to set up the pipeline
var effectInstance = new EffectInstance(EffectSystem.LoadEffect("MyEffect").WaitForResult());
pipelineStateDescription.EffectBytecode = effectInstance.Effect.Bytecode;
pipelineStateDescription.RootSignature = effectInstance.RootSignature;

// Update constant buffers and bind resources
effectInstance.Apply(context.GraphicsContext);
```

Remark: The `EffectInstance` class handles the details of enumerating **graphics resources** from a loaded **effect** as well as **binding** them. Specifically, **graphics resources** refer to (*exaustive list*): **textures**, **buffers**, **samplers**, **constant buffers**.

4. Drawing indexed vertices

```
// Create the index buffer
var indices = new short[indexCount];
var is32Bits = false;
var indexBuffer = Buffer.Index.New(GraphicsDevice, indices);

// set the VAO
commandList.SetIndexBuffer(indexBuffer, 0, is32Bits);

// Draw indexed vertices
commandList.DrawIndexed(indexBuffer.ElementCount);
```

5. \[Custom Shader\] Creates a green color

```
namespace MyGame
{
    shader MyShader : ComputeColor
    {
        override float4 Compute()
        {
            return float4(0, 1, 0, 1);
        }
    };
}
```

Remark: Written in `.xksl` file

Remark: Make sure the shader name in the file (eg **MyShader** above) is the same as the filename.

Remark: To be accessible from the **Game Studio** Property Grid, the shader must inherit from `ComputeColor`. As `ComputeColor` always returns a `float4` value, properties that take float values (eg metalness and gloss maps) use the first component (the red component) of the value returned by ComputeColor.

Reference: To create custom shaders, see [doc page](https://doc.xenko.com/latest/en/manual/graphics/effects-and-shaders/custom-shaders.html).

6. Create and Use Template Shader

```
shader TemplateShader<float speed, Texture2D myTexture, SamplerState mySampler, Semantic mySemantic, LinkType myLink>
{
    [Color]
    [Link("myLink")]
    float4 myColor;

    stream float2 texcoord : mySemantic;

    float4 GetValue()
    {
        return speed * myColor * myTexture.Sample(mySampler, streams.texcoord);
    }
};

// To instantiate the shader, use:
TemplateShader<1.0f, Texturing.Texture0, Texturing.Sampler0, TEXCOORD0, MyColorLink>
```

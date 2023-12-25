# Raw WebGPU

[https://alain.xyz/blog/raw-webgpu](https://alain.xyz/blog/raw-webgpu)

*An overview on how to write a WebGPU application. Learn what key data structures and types are needed to draw in WebGPU.*

# Setup

```bash
# 🐑 Clone the repo
git clone https://github.com/alaingalvan/webgpu-seed

# 💿 go inside the folder
cd webgpu-seed

# 🔨 Start building the project
npm start
```

## Project Layout

As your project becomes more complex, you'll want to separate files and organize your application to something more akin to a game or renderer

```bash
├─ 📂 node_modules/   # 👶 Dependencies
│  ├─ 📁 gl-matrix      # ➕ Linear Algebra
│  └─ 📁 ...            # 🕚 Other Dependencies (TypeScript, Webpack, etc.)
├─ 📂 src/            # 🌟 Source Files
│  ├─ 📄 renderer.ts    # 🔺 Triangle Renderer
│  └─ 📄 main.ts        # 🏁 Application Main
├─ 📄 .gitignore      # 👁️ Ignore Certain Files in Git Repo
├─ 📄 package.json    # 📦 Node Package File
├─ 📄 [[]]      # ⚖️ Your License (Unlicense)
└─ [[]]        # 📖 Read Me!
```

### Dependencies

- [gl-matrix](https://github.com/toji/gl-matrix) - A JavaScript library that allows users to write `glsl` like JavaScript code, with types for vectors, matrices, etc.
    - Not in use in this sample, useful for programming more advanced topics such as camera matrices
- [TypeScript](https://github.com/microsoft/typescript) - JavaScript with types, makes it significantly easier to program web apps with instant autocomplete and type checking
- [Webpack](https://github.com/webpack/webpack) - A JavaScript compilation tool to build minified outputs and test our apps faster

## Overview

1. **Initialize the API** - Check if `navigator.gpu` exists, and if it does, request a `GPUAdapter`, then request a `GPUDevice`, and get that device's default `GPUQueue`.
2. **Setup Frame Backings** - create a `GPUCanvasContext` and configure it to receive a `GPUTexture` for the current frame, as well as any other attachments you might need (such as a depth-stencil texture, etc.). Create `GPUTextureView`s for those textures.
3. **Initialize Resources** - Create your Vertex and Index `GPUBuffer`s, load your WebGPU Shading Language (WGSL) shaders as `GPUShaderModule`s, create your `GPURenderPipeline` by describing every stage of the graphics pipeline. Finally, build your `GPUCommandEncoder` with what what render passes you intend to run, then a `GPURenderPassEncoder` with all the draw calls you intend to execute for that render pass.
4. **Render** - Submit your `GPUCommandEncoder` by calling `.finish()`, and submitting that to your `GPUQueue`. Refresh the canvas context by calling `requestAnimationFrame`.
5. **Destroy** - Destroy any data structures after you're done using the API.

## Start

To access the WebGPU API, you need to see if there exists a `gpu` object in the global `navigator`

```jsx
// 🏭 Entry to WebGPU
const entry: GPU = navigator.gpu;
if (!entry) {
    throw new Error('WebGPU is not supported on this browser.');
}
```

### Adapter

An **Adapter** describes the physical properties of a given GPU, such as its name, extensions, and device limits

```jsx
// ✋ Declare adapter handle
let adapter: GPUAdapter = null;

// 🙏 Inside an async function...

// 🔌 Physical Device Adapter
adapter = await entry.requestAdapter();
```

### Device

A **Device** is how you access the *core* of the WebGPU API, and will allow you to create the data structures you'll need

```jsx
// ✋ Declare device handle
let device: GPUDevice = null;

// 🙏 Inside an async function...

// 💻 Logical Device
device = await adapter.requestDevice();
```

### Queue

A **Queue** allows you to send work asynchronously to the GPU. As of the writing of this post, you can only access a single default queue from a given `GPUDevice`

```jsx
// ✋ Declare queue handle
let queue: GPUQueue = null;

// 📦 Queue
queue = device.queue;
```

## Frame Backings

### Canvas Context

In order to see what you're drawing, you'll need an `HTMLCanvasElement` and to setup a **Canvas Context** from that canvas

- A Canvas Context manages a series of textures you'll use to present your final render output to your `<canvas>` element

```jsx
// ✋ Declare context handle
const context: GPUCanvasContext = null;

// ⚪ Create Context
context = canvas.getContext('webgpu');

// ⛓️ Configure Context
const canvasConfig: GPUCanvasConfiguration = {
    device: this.device,
    format: 'bgra8unorm',
    usage: GPUTextureUsage.RENDER_ATTACHMENT | GPUTextureUsage.COPY_SRC,
    alphaMode: 'opaque'
};

context.configure(canvasConfig);
```

### Frame Buffer Attachments

When executing different passes of your rendering system, you'll need output textures to write to

- Textures could be for depth testing or shadows, or attachments for various aspects of a deferred renderer such as view space normals, PBR reflectivity/roughness, etc

Frame buffers attachments are references to texture views, which you'll see later when we write our rendering logic

```jsx
// ✋ Declare attachment handles
let depthTexture: GPUTexture = null;
let depthTextureView: GPUTextureView = null;

// 🤔 Create Depth Backing
const depthTextureDesc: GPUTextureDescriptor = {
    size: [canvas.width, canvas.height, 1],
    dimension: '2d',
    format: 'depth24plus-stencil8',
    usage: GPUTextureUsage.RENDER_ATTACHMENT | GPUTextureUsage.COPY_SRC
};

depthTexture = device.createTexture(depthTextureDesc);
depthTextureView = depthTexture.createView();

// ✋ Declare canvas context image handles
let colorTexture: GPUTexture = null;
let colorTextureView: GPUTextureView = null;

colorTexture = context.getCurrentTexture();
colorTextureView = colorTexture.createView();
```

## Initialize Resources

### Vertex & Index Buffers

![[/Screen Shot 2023-05-29 at 5.40.38 PM.png]]

A **Buffer** is an array of data, such as a mesh's positional data, color data, index data, etc.

When rendering triangles with a raster based *graphics pipeline*, you'll need:

- 1 or more buffers of vertex data
    - (commonly referred to as *Vertex Buffer Objects* or *VBO*s)
- 1 buffer of the indices that correspond with each triangle vertex that you intend to draw
    - (otherwise known as an *Index Buffer Object* or *IBO*).

```jsx
// 📈 Position Vertex Buffer Data
const positions = new Float32Array([
    1.0, -1.0, 0.0,
	 -1.0, -1.0, 0.0,
	  0.0, 1.0, 0.0
]);

// 🎨 Color Vertex Buffer Data
const colors = new Float32Array([
    1.0, 0.0, 0.0, // 🔴
    0.0, 1.0, 0.0, // 🟢
		0.0, 0.0, 1.0 // 🔵
]);

// 📇 Index Buffer Data
const indices = new Uint16Array([0, 1, 2]);

// ✋ Declare buffer handles
let positionBuffer: GPUBuffer = null;
let colorBuffer: GPUBuffer = null;
let indexBuffer: GPUBuffer = null;

// 👋 Helper function for creating GPUBuffer(s) out of Typed Arrays
const createBuffer = (arr: Float32Array | Uint16Array, usage: number) => {
    // 📏 Align to 4 bytes (thanks @chrimsonite)
    let desc = {
        size: (arr.byteLength + 3) & ~3,
        usage,
        mappedAtCreation: true
    };
    let buffer = device.createBuffer(desc);

    const writeArray =
        arr instanceof Uint16Array
            ? new Uint16Array(buffer.getMappedRange())
            : new Float32Array(buffer.getMappedRange());
    writeArray.set(arr);
    buffer.unmap();
    return buffer;
};

positionBuffer = createBuffer(positions, GPUBufferUsage.VERTEX);
colorBuffer = createBuffer(colors, GPUBufferUsage.VERTEX);
indexBuffer = createBuffer(indices, GPUBufferUsage.INDEX);
```

### Shaders

With WebGPU comes a new shader language: **WebGPU Shading Language (WGSL)**:

WebGPU Shading Language is similar to other languages like Rust with JavaScript style decorators such as `@location(0)`

Rust style code formatting with `snake_case` functions/members, `CamelCase` structs, and functions following `fn my_func() -> i32`

Here's the vertex shader source:

```rust
struct VSOut {
    @builtin(position) nds_position: vec4<f32>,
    @location(0) color: vec3<f32>,
};

@vertex
fn main(@location(0) in_pos: vec3<f32>,
        @location(1) in_color: vec3<f32>) -> VSOut {
    var vs_out: VSOut;
    vs_out.nds_position = vec4<f32>(in_pos, 1.0);
    vs_out.color = inColor;
    return vsOut;
}
```

Here's the fragment shader source:

```rust
@fragment
fn main(@location(0) in_color: vec3<f32>) -> @location(0) vec4<f32> {
    return vec4<f32>(in_color, 1.0);
}
```

### Shader Modules

**Shader Module**s are *plain text WGSL files* that execute on the GPU when executing a given pipeline

- Can be imported to use in createShaderModule declaration:

```jsx
// 📄 Import or declare in line your WGSL code:
import vertShaderCode from './shaders/triangle.vert.wgsl';
import fragShaderCode from './shaders/triangle.frag.wgsl';

// ✋ Declare shader module handles
let vertModule: GPUShaderModule = null;
let fragModule: GPUShaderModule = null;

const vsmDesc = { code: vertShaderCode };
vertModule = device.createShaderModule(vsmDesc);

const fsmDesc = { code: fragShaderCode };
fragModule = device.createShaderModule(fsmDesc);
```

### Uniform Buffer

![[/Screen Shot 2023-05-29 at 5.57.18 PM.png]]

You'll often times need to feed data directly to your shader modules, and to do this you'll need to specify a **uniform**

- A **uniform** is a value from a buffer that is the same for every invocation, available at all shader stages in a pipeline

In order to create a **Uniform Buffer** in your shader, declare the following prior to your main function:

```rust
struct UBO {
  modelViewProj: mat4x4<f32>,
  primaryColor: vec4<f32>,
  accentColor: vec4<f32>
};

@group(0) @binding(0)
var<uniform> uniforms: UBO;

// ❗ Then in your Vertex Shader's main file,
// replace the 4th to last line with:
vsOut.Position = uniforms.modelViewProj * vec4<f32>(inPos, 1.0);
```

Then in your JavaScript code, create a uniform buffer as you would with an index/vertex buffer.

```jsx
// 👔 Uniform Data
const uniformData = new Float32Array([

    // ♟️ ModelViewProjection Matrix (Identity)
    1.0, 0.0, 0.0, 0.0
    0.0, 1.0, 0.0, 0.0
    0.0, 0.0, 1.0, 0.0
    0.0, 0.0, 0.0, 1.0

    // 🔴 Primary Color
    0.9, 0.1, 0.3, 1.0

    // 🟣 Accent Color
    0.8, 0.2, 0.8, 1.0
]);

// ✋ Declare buffer handles
let uniformBuffer: GPUBuffer = null;

uniformBuffer = createBuffer(uniformData, GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST);
```

### Pipeline Layout

![[/Screen Shot 2023-05-29 at 6.11.29 PM.png]]

Once you have a uniform, you can create a **Pipeline Layout** to describe where that uniform will be when executing a *Graphics Pipeline*

You have 2 options here

1. You can either let WebGPU create your pipeline layout for you
2. Get it from a pipeline that's already been created:

```jsx
let bindGroupLayout: GPUBindGroupLayout = null;
let uniformBindGroup: GPUBindGroup = null;

// 👨‍🔧 Create your graphics pipeline...

// 🧙‍♂️ Then get your implicit pipeline layout:
bindGroupLayout = pipeline.getBindGroupLayout(0);

// 🗄️ Bind Group
// ✍ This would be used when *encoding commands*
uniformBindGroup = device.createBindGroup({
    layout: bindGroupLayout,
    entries: [
        {
            binding: 0,
            resource: {
                buffer: uniformBuffer
            }
        }
    ]
});
```

Or if you know the layout in advance, you can describe it yourself and use it during pipeline creation:

```jsx
// ✋ Declare handles
let uniformBindGroupLayout: GPUBindGroupLayout = null;
let uniformBindGroup: GPUBindGroup = null;
let layout: GPUPipelineLayout = null;

// 📁 Bind Group Layout
uniformBindGroupLayout = device.createBindGroupLayout({
    entries: [
        {
            binding: 0,
            visibility: GPUShaderStage.VERTEX,
            buffer: {}
        }
    ]
});

// 🗄️ Bind Group
// ✍ This would be used when *encoding commands*
uniformBindGroup = device.createBindGroup({
    layout: uniformBindGroupLayout,
    entries: [
        {
            binding: 0,
            resource: {
                buffer: uniformBuffer
            }
        }
    ]
});

// 🗂️ Pipeline Layout
// 👩‍🔧 This would be used as a member of a GPUPipelineDescriptor when *creating a pipeline*
const pipelineLayoutDesc = { bindGroupLayouts: [uniformBindGroupLayout] };
layout = device.createPipelineLayout(pipelineLayoutDesc);
```

When encoding commands, you can use this uniform with `setBindGroup`:

```jsx
// ✍ Later when you're encoding commands:
passEncoder.setBindGroup(0, uniformBindGroup);
```

### Graphics Pipeline

![[/Screen Shot 2023-05-29 at 6.25.03 PM.png]]

A **Graphics Pipeline** describes all the data that's to be fed into the execution of a raster based graphics pipeline. This includes:

- 🔣 **Input Assembly** - What does each vertex look like? Which attributes are where, and how do they align in memory?
- 🖍️ **Shader Modules** - What shader modules will you be using when executing this graphics pipeline?
- ✏️ **Depth/Stencil State** - Should you perform depth testing? If so, what function should you use to test depth?
- 🍥 **Blend State** - How should colors be blended between the previously written color and current one?
- 🔺 **Rasterization** - How does the rasterizer behave when executing this graphics pipeline? Does it cull faces? Which direction should the face be culled?
- 💾 **Uniform Data** - What kind of uniform data should your shaders expect? In WebGPU this is done by describing a **Pipeline Layout**.

WebGPU features smart defaults for the graphics pipeline state, so most of the time you won't need to even set parts of it:

```jsx
// ✋ Declare pipeline handle
let pipeline: GPURenderPipeline = null;

// ⚗️ Graphics Pipeline

// 🔣 Input Assembly
const positionAttribDesc: GPUVertexAttribute = {
    shaderLocation: 0, // @location(0)
    offset: 0,
    format: 'float32x3'
};
const colorAttribDesc: GPUVertexAttribute = {
    shaderLocation: 1, // @location(1)
    offset: 0,
    format: 'float32x3'
};
const positionBufferDesc: GPUVertexBufferLayout = {
    attributes: [positionAttribDesc],
    arrayStride: 4 * 3, // sizeof(float) * 3
    stepMode: 'vertex'
};
const colorBufferDesc: GPUVertexBufferLayout = {
    attributes: [colorAttribDesc],
    arrayStride: 4 * 3, // sizeof(float) * 3
    stepMode: 'vertex'
};

// 🌑 Depth
const depthStencil: GPUDepthStencilState = {
    depthWriteEnabled: true,
    depthCompare: 'less',
    format: 'depth24plus-stencil8'
};

// 🦄 Uniform Data
const pipelineLayoutDesc = { bindGroupLayouts: [] };
const layout = device.createPipelineLayout(pipelineLayoutDesc);

// 🎭 Shader Stages
const vertex: GPUVertexState = {
    module: vertModule,
    entryPoint: 'main',
    buffers: [positionBufferDesc, colorBufferDesc]
};

// 🌀 Color/Blend State
const colorState: GPUColorTargetState = {
    format: 'bgra8unorm'
};

const fragment: GPUFragmentState = {
    module: fragModule,
    entryPoint: 'main',
    targets: [colorState]
};

// 🟨 Rasterization
const primitive: GPUPrimitiveState = {
    frontFace: 'cw',
    cullMode: 'none',
    topology: 'triangle-list'
};

const pipelineDesc: GPURenderPipelineDescriptor = {
    layout,
    vertex,
    fragment,
    primitive,
    depthStencil
};

pipeline = device.createRenderPipeline(pipelineDesc);
```

### Command Encoder

![[/Screen Shot 2023-05-29 at 6.28.07 PM.png]]

**Command Encoder**s encode all the draw commands you intend to execute in groups of **Render Pass Encoders**

Once you've finished encoding commands, you'll receive a **Command Buffer** that you could submit to your queue

- In that sense a command buffer is analogous to a *callback* that executes draw functions on the GPU once it's submitted to the queue

```jsx
// ✋ Declare command handles
let commandEncoder: GPUCommandEncoder = null;
let passEncoder: GPURenderPassEncoder = null;

// ✍️ Write commands to send to the GPU
const encodeCommands = () => {
    let colorAttachment: GPURenderPassColorAttachment = {
        view: this.colorTextureView,
        clearValue: { r: 0, g: 0, b: 0, a: 1 },
        loadOp: 'clear',
        storeOp: 'store'
    };

    const depthAttachment: GPURenderPassDepthStencilAttachment = {
        view: this.depthTextureView,
        depthClearValue: 1,
        depthLoadOp: 'clear',
        depthStoreOp: 'store',
        stencilClearValue: 0,
        stencilLoadOp: 'clear',
        stencilStoreOp: 'store'
    };

    const renderPassDesc: GPURenderPassDescriptor = {
        colorAttachments: [colorAttachment],
        depthStencilAttachment: depthAttachment
    };

    commandEncoder = device.createCommandEncoder();

    // 🖌️ Encode drawing commands
    passEncoder = commandEncoder.beginRenderPass(renderPassDesc);
    passEncoder.setPipeline(pipeline);
    passEncoder.setViewport(0, 0, canvas.width, canvas.height, 0, 1);
    passEncoder.setScissorRect(0, 0, canvas.width, canvas.height);
    passEncoder.setVertexBuffer(0, positionBuffer);
    passEncoder.setVertexBuffer(1, colorBuffer);
    passEncoder.setIndexBuffer(indexBuffer, 'uint16');
    passEncoder.drawIndexed(3);
    passEncoder.endPass();

    queue.submit([commandEncoder.finish()]);
};
```

### Render

**Rendering** in WebGPU is a simple matter of updating any uniforms you intend to update, getting the next attachments from your context, submitting your command encoders to be executed, and using the `requestAnimationFrame` callback to do all of that again

```jsx
const render = () => {
    // ⏭ Acquire next image from context
    colorTexture = context.getCurrentTexture();
    colorTextureView = colorTexture.createView();

    // 📦 Write and submit commands to queue
    encodeCommands();

    // ➿ Refresh canvas
    requestAnimationFrame(render);
};
```
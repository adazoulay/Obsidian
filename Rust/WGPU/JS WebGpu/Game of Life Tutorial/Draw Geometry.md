# Draw Geometry

## Understanding how GPUs Draw

Unlike an API like Canvas 2D that has lots of shapes and options ready for you to use, your GPU really only deals with a few different types of shapes

- (or *primitives* as they're referred to by WebGPU): points, lines, and triangles

GPUs work almost exclusively with triangles because triangles have a lot of nice mathematical properties that make them easy to process in a predictable and efficient way

- Almost everything you draw with the GPU needs to be split up into triangles before the GPU can draw

## Triangles

Triangles defined by their corner points

- These points, or *vertices*, are given in terms of X, Y, and (for 3D content) Z values that define a point on a [cartesian coordinate system](https://en.wikipedia.org/wiki/Cartesian_coordinate_system) defined by WebGPU or similar APIs
- Easiest to think about in terms of how it relates to the canvas on your page

### Plane: *Clip Space*

No matter how wide or tall your canvas is:

- The left edge is always at (-1, 0) on the X axis, and the right edge is always at (+1, 0) on the X axis
- The bottom edge is always (0, -1) on the Y axis, and the top edge is (0, 1) on the Y axis
- (0, 0) is always the center of the canvas, (-1, -1) is always the bottom-left corner, and (1, 1) is always the top-right corner etc…

This is known as *Clip Space*

![Untitled](Draw%20Geometry/Untitled.png)

The vertices are rarely defined in this coordinate system initially, so GPUs rely on small programs called *vertex shaders* to perform whatever math is necessary to transform the vertices into clip space, as well as any other calculations needed to draw the vertices

- Example: Shader may apply some animation or calculate the direction from the vertex to a light source

From there, the GPU takes all the triangles made up by these transformed vertices and determines which pixels on the screen are needed to draw them

- Then it runs another small program you write called a *fragment shader* that calculates what color each pixel should be
    - Can be as simple as return green or as complex as ray tracing

## Define Vertices

### In our Game of Life Example:

The Game of Life simulation is shown as a grid of *cells*. Your app needs a way to visualize the grid, distinguishing active cells from inactive cells

- Approach used by this codelab will be to draw colored squares in the active cells and leave inactive cells empty
- This means that you'll need to provide the GPU with four different points, one for each of the four corners of the square

For example, a square drawn in the center of the canvas, pulled in from the edges a ways, has corner coordinates like this:

![Untitled](Draw%20Geometry/Untitled%201.png)

In order to feed those coordinates to the GPU, you need to place the values in a **TypedArray**

### Note on TypedArrays

TypedArrays are a group of JavaScript objects that allows you to allocate contiguous blocks of memory and interpret each element in the series as a specific data type

For example, in a `Uint8Array`, each element in the array is a single, unsigned byte

TypedArrays are great for sending data back and forth with APIs that are sensitive to memory layout, like WebAssembly, WebAudio, and (of course) WebGPU

For the square example, because the values are fractional, a `[Float32Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float32Array)` is appropriate

- Create an array that holds all of the vertex positions in the diagram by placing the following array declaration in your code
- A good place to put it is near the top, just under the `context.configure()` call

```jsx
const vertices = new Float32Array([
//   X,    Y,
  -0.8, -0.8,
   0.8, -0.8,
   0.8,  0.8,
  -0.8,  0.8,
]);
```

**Note:** Values are treated as pairs: spacing and comment has no effect on the values

**Problem:** GPUs work in terms of triangles

- Means that you have to provide the vertices in groups of three
- We have one group of 4 pairs right now

**********************Solution:********************** Repeat two of the vertices to create two triangles sharing an edge through the middle of the square

![Untitled](Draw%20Geometry/Untitled%202.png)

To form the square from the diagram, you have to list the (-0.8, -0.8) and (0.8, 0.8) vertices twice, once for the blue triangle and once for the red one

We update previous `vertices` array to look something like this:

```jsx
const vertices = new Float32Array([
//   X,    Y,
  -0.8, -0.8, // Triangle 1 (Blue)
   0.8, -0.8,
   0.8,  0.8,

  -0.8, -0.8, // Triangle 2 (Red)
   0.8,  0.8,
  -0.8,  0.8,
]);
```

**Note Might be useful in DAW:** You don't *have* to repeat the vertex data in order to make triangles

- Using something called Index Buffers, you can feed a separate list of values to the GPU that tells it what vertices to connect together into triangles so that they don't need to be duplicated. It's like connect-the-dots! Because your vertex data is so simple, using Index Buffers is out of scope for this Codelab. But they're definitely something that you might want to make use of for more complex geometry.

## Create a vertex buffer

The GPU cannot draw vertices with data from a JavaScript array

- GPUs frequently have their own memory that is highly optimized for rendering, and so any data you want the GPU to use while it draws needs to be placed in that memory

For a lot of values, including vertex data, the GPU-side memory is managed through `[GPUBuffer](https://gpuweb.github.io/gpuweb/#gpubuffer)` objects

- A buffer is a block of memory that's easily accessible to the GPU and flagged for certain purposes
- Can think of it a little bit like a GPU-visible TypedArray

To create a buffer to hold your vertices, add the following call to `[device.createBuffer()](https://gpuweb.github.io/gpuweb/#dom-gpudevice-createbuffer)` after the definition of your `vertices` array

```jsx
const vertexBuffer = device.createBuffer({
  label: "Cell vertices",
  size: vertices.byteLength,
  usage: GPUBufferUsage.VERTEX | GPUBufferUsage.COPY_DST,
});
```

- Give the buffer a **label***: Every single WebGPU object you create can be given an optional label, and you definitely want to do so!*
    - Label is any string you want, as long as it helps you identify what the object is
    - Used in error messages to help you understand what went wrong
- Next, give a **size** for the buffer in bytes
    - You need a buffer with 48 bytes: Determine by multiplying the size of a 32-bit float ( [4 bytes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/BYTES_PER_ELEMENT)) by the number of floats in your `vertices` array (12)
    - **Note:** Get comfortable with this kind of byte-size math. Working with a GPU requires a fair amount of it!
- Finally, you need to specify the **usage** of the buffer:
    - This is one or more of the `[GPUBufferUsage](https://gpuweb.github.io/gpuweb/#buffer-usage)` flags, with multiple flags being combined with the `|` ( bitwise OR) operator
    - In this case, you specify that you want the buffer to be used for vertex data (`GPUBufferUsage.VERTEX`) and that you also want to be able to copy data into it (`GPUBufferUsage.COPY_DST`)

The buffer object that gets returned to you is:

- **Opaque:** You can't (easily) inspect the data it holds
- **********************immutable:********************** You can't resize a `GPUBuffer` after it's been created, nor can you change the usage flags
    - What you can change are the contents of its memory
    

When the buffer is initially created, the memory it contains will be initialized to zero

There are several ways to change its contents, but the easiest is to call `[device.queue.writeBuffer()](https://gpuweb.github.io/gpuweb/#dom-gpuqueue-writebuffer)` with a TypedArray that you want to copy in

To copy the vertex data into the buffer's memory, add the following code:

```jsx
device.queue.writeBuffer(vertexBuffer, /*bufferOffset=*/0, vertices);
```

## Defining the Vertex Layout

Now you have a buffer with vertex data in it, but as far as the GPU is concerned it's just a blob of bytes

- Need to supply a little bit more information if you're going to draw anything with it
- You need to be able to tell WebGPU more about the structure of the vertex data

To do so, define the vertex data structure with a `[GPUVertexBufferLayout](https://gpuweb.github.io/gpuweb/#dictdef-gpuvertexbufferlayout)` dictionary:

```jsx
const vertexBufferLayout = {
  arrayStride: 8,
  attributes: [{
    format: "float32x2",
    offset: 0,
    shaderLocation: 0, // Position, see vertex shader
  }],
};
```

- The first thing you give is the *`arrayStride`*
    - This is the number of bytes the GPU needs to skip forward in the buffer when it's looking for the next vertex
    - Each vertex of your square is made up of two 32-bit floating point numbers. As mentioned earlier, a 32-bit float is 4 bytes, so two floats is 8 bytes
- Next is the `attributes` property, which is an array
    - Attributes are the individual pieces of information encoded into each vertex
    - Your vertices only contain one attribute (the vertex position), but more advanced use cases frequently have vertices with multiple attributes in them like the color of a vertex or the direction the geometry surface is pointing
- In your single attribute, you first define the `format` of the data
    - This comes from a list of `[GPUVertexFormat](https://gpuweb.github.io/gpuweb/#enumdef-gpuvertexformat)` types that describe each type of vertex data that the GPU can understand
    - Your vertices have two 32-bit floats each, so you use the format `float32x2`
        - If your vertex data is instead made up of four [16-bit unsigned integers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint16Array) each, for example, you'd use `uint16x4` instead. See the pattern?
- Next, the `offset` describes how many bytes into the vertex this particular attribute starts
    - You really only have to worry about this if your buffer has more than one attribute in it, which won't come up during this codelab
- Finally, you have the **`shaderLocation`
    - This is an arbitrary number between 0 and 15 and must be unique for every attribute that you define
    - It links this attribute to a particular input in the vertex shader

## Start with Shaders

Now you have the data you want to render, but you still need to tell the GPU exactly how to process it. A large part of that happens with shaders.

Shaders are small programs that you write and that execute on your GPU

Each shader operates on a different *stage* of the data:

- *Vertex* processing, *Fragment* processing, or general *Compute*

Because they're on the GPU, they are structured more rigidly than your average JavaScript

- But that structure allows them to execute very fast and, crucially, in parallel

Shaders in WebGPU are written in a shading language called [WGSL](https://gpuweb.github.io/gpuweb/wgsl.html) (WebGPU Shading Language) 

- WGSL is, syntactically, a bit like Rust, with features aimed at making common types of GPU work (like vector and matrix math) easier and faster

The shaders themselves get passed into WebGPU as strings.

- Create a place to enter your shader code by copying the following into your code below the `vertexBufferLayout`:

```jsx
const cellShaderModule = device.createShaderModule({
  label: "Cell shader",
  code: `
    // Your shader code will go here
  `
});
```

To create the shaders you call `device.createShaderModule()`, to which you provide an optional *`label`* and WGSL *`code`* as a string

- Once you add some valid WGSL code, the function returns a `[GPUShaderModule](https://gpuweb.github.io/gpuweb/#gpushadermodule)` object with the compiled results

## Define the Vertex Shader

Start with the vertex shader because that's where the GPU starts, too!

A vertex shader is defined as a function, and the GPU calls that function once for every vertex in your `vertexBuffer`

Since your `vertexBuffer` has six positions (vertices) in it, the function you define gets called six times

- Each time it is called, a different position from the `vertexBuffer` is passed to the function as an argument, and it's the job of the vertex shader function to return a corresponding position in clip space

**Note:** Important to understand that they won't necessarily get called in sequential order

- Instead, GPUs excel at running shaders like these in parallel, potentially processing hundreds (or even thousands!) of vertices at the same time
- In order to ensure extreme parallelization, vertex shaders cannot communicate with each other
    - Each shader invocation can only *see* data for a single vertex at a time, and is only able to output values for a single vertex

In WGSL, a vertex shader function can be named whatever you want, but it must have the `@vertex` *attribute* in front of it in order to indicate which shader stage it represents

Uses **Rust** like syntax

```rust
WON'T WORK
@vertex
fn vertexMain() {
}
```

Not valid as a vertex shader must return *at least* the final position of the vertex being processed in clip space

- This is always given as a 4-dimensional vector
- Vectors are such a common thing to use in shaders that they're treated as first-class primitives in the language, with their own types like `vec4f` for a 4-dimensional vector
    - There are similar types for 2D vectors (`vec2f`) and 3D vectors (`vec3f`)
- You can construct a new `vec4f` to return, using the syntax `vec4f(x, y, z, w)`
    - The `x`, `y`, and `z` values are all floating point numbers that, in the return value, indicate where the vertex lies in clip space.
    - What’s `w`? the fourth component of a three-dimensional homogeneous vertex. Don’t have to wory about it much: Makes math with 4x4 matrices work

We want to make use of the data from the buffer that we created,

- Done by declaring an argument for our function with a `@location()` attribute and type that match what we described in the `vertexBufferLayout`
    - Specified a `shaderLocation` of `0`, so in our WGSL code, mark the argument with `@location(0)`
    - Also defined the format as a `float32x2`, which is a 2D vector, so in WGSL your argument is a `vec2f`

```rust
@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {
  return vec4f(0, 0, 0, 1);
}
```

And now you need to return that position. Since the position is a 2D vector and the return type is a 4D vector, you have to alter it a bit

- What you want to do is take the two components from the position argument and place them in the first two components of the return vector, leaving the last two components as `0` and `1`, respectively

```rust
@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {
  return vec4f(pos.x, pos.y, 0, 1);
}
```

*However*, because these kinds of mappings are so common in shaders, you can also pass the position vector in as the first argument in a convenient shorthand and it means the same thing

- `return vec4f(pos, 0, 1);`

And that's your initial vertex shader!

## Define the Fragment Shader

Fragment shaders operate in a very similar way to vertex shaders, but rather than being invoked for every vertex, they're invoked for every pixel being drawn

Fragment shaders are always called after vertex shaders

- The GPU takes the output of the vertex shaders and *triangulates* it, creating triangles out of sets of three points
- Then *rasterizes* each of those triangles by figuring out which pixels of the output color attachments are included in that triangle
- Then calls the fragment shader once for each of those pixels
    - The fragment shader returns a color
    - Typically calculated from values sent to it from the vertex shader and assets like textures

Just like vertex shaders, fragment shaders are executed in a massively parallel fashion

- A little more flexible than vertex shaders in terms of their inputs and outputs, but you can consider them to simply return one color for each pixel of each triangle

A WGSL fragment shader function is denoted with the `@fragment` attribute and it also returns a `vec4f`

- In this case, though, the vector represents a color, not a position

The return value needs to be given a `@location` attribute in order to indicate which `colorAttachment` from the `beginRenderPass` call the returned color is written to

- Since we only had one attachment, the location is 0

```rust
@fragment
fn fragmentMain() -> @location(0) vec4f {

}
```

The four components of the returned vector are the red, green, blue, and alpha color values

- Are interpreted in exactly the same way as the `clearValue` you we in `beginRenderPass` earlier

Set the returned color vector, like this

- `vec4f(1, 0, 0, 1)` is bright red

```rust
@fragment
fn fragmentMain() -> @location(0) vec4f {
  return vec4f(1, 0, 0, 1); // (Red, Green, Blue, Alpha)
}
```

And that's a complete fragment shader! it just sets every pixel of every triangle to red

In summary, `createShaderModule` call now looks like this:

```jsx
const cellShaderModule = device.createShaderModule({
  label: 'Cell shader',
  code: `
    @vertex
    fn vertexMain(@location(0) pos: vec2f) ->
      @builtin(position) vec4f {
      return vec4f(pos, 0, 1);
    }

    @fragment
    fn fragmentMain() -> @location(0) vec4f {
      return vec4f(1, 0, 0, 1);
    }
  `
});
```

**Note:** You can also create a separate shader module for your vertex and fragment shaders, if you want

- Can be useful if, for example, you want to use several different fragment shaders with the same vertex shader

## Create a render pipeline

A shader module can't be used for rendering on its own

Instead, you have to use it as part of a `[GPURenderPipeline](https://gpuweb.github.io/gpuweb/#gpurenderpipeline)`, created by calling [device.createRenderPipeline()](https://gpuweb.github.io/gpuweb/#dom-gpudevice-createrenderpipeline)

- render pipeline controls *how* geometry is drawn, including things like which shaders are used, how to interpret data in vertex buffers, which kind of geometry should be rendered (lines, points, triangles...), and more
- The render pipeline is the most complex object in the entire API
    - Most of the values you can pass to it are optional, and you only need to provide a few to start

```jsx
const cellPipeline = device.createRenderPipeline({
  label: "Cell pipeline",
  layout: "auto",
  vertex: {
    module: cellShaderModule,
    entryPoint: "vertexMain",
    buffers: [vertexBufferLayout]
  },
  fragment: {
    module: cellShaderModule,
    entryPoint: "fragmentMain",
    targets: [{
      format: canvasFormat
    }]
  }
});
```

- Every pipeline needs a *`layout`* that describes what types of inputs (other than vertex buffers) the pipeline needs
    - but we don't really have any; fortunately, you can pass `"auto"` for now, and the pipeline builds its own layout from the shaders
- Next, you have to provide details about the `vertex` stage
    - The `module` is the GPUShaderModule that contains your vertex shader, and the `entryPoint` gives the name of the function in the shader code that is called for every vertex invocation
    - (You can have multiple `@vertex` and `@fragment` functions in a single shader module!)
    - The *buffers* is an array of `GPUVertexBufferLayout` objects that describe how your data is packed in the vertex buffers that you use this pipeline with
        - Already defined this earlier in your `vertexBufferLayout`
- Lastly, you have details about the `fragment` stage
    - Also includes a shader *module* and *entryPoint*, like the vertex stage
- The last bit is to define the *`targets`* that this pipeline is used with
    - Array of dictionaries giving details—such as the texture `format`—of the color attachments that the pipeline outputs to
    - These details need to match the textures given in the `colorAttachments` of any render passes that this pipeline is used with
    - Your render pass uses textures from the canvas context, and uses the value you saved in `canvasFormat` for its format, so you pass the same format here

## Draw the Square

To draw the square, jump back down to the `encoder.beginRenderPass()` and `pass.end()` pair of calls, and then add these new commands between them:

```jsx
// After encoder.beginRenderPass()

pass.setPipeline(cellPipeline);
pass.setVertexBuffer(0, vertexBuffer);
pass.draw(vertices.length / 2); // 6 vertices

// before pass.end()
```

This supplies WebGPU with all the information necessary to draw your square

- First, you use **`[setPipeline()](https://gpuweb.github.io/gpuweb/#dom-gpurendercommandsmixin-setpipeline)`** to indicate which pipeline should be used to draw with
    - This includes the shaders that are used, the layout of the vertex data, and other relevant state data
- Next, you call **`[setVertexBuffer()](https://gpuweb.github.io/gpuweb/#dom-gpurendercommandsmixin-setvertexbuffer)`** with the buffer containing the vertices for your square
    - Call it with `0` because this buffer corresponds to the 0th element in the current pipeline's `vertex.buffers` definition
- And last, you make the **`[draw()](https://gpuweb.github.io/gpuweb/#dom-gpurendercommandsmixin-draw)`** call, which seems strangely simple after all the setup that's come before
    - Only thing you need to pass in is the number of vertices that it should render, which it pulls from the currently set vertex buffers and interprets with the currently set pipeline
    - Could just hard-code it to `6`, but calculating it from the vertices array (12 floats / 2 coordinates per vertex == 6 vertices)
        - Means that if you ever decided to replace the square with, for example, a circle, there's less to update by hand
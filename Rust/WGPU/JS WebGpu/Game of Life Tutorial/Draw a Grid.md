# Draw a Grid

## Define the Grid

We’ll start with a 4x4 grid to make some of the math easier. Scale later

```jsx
const GRID_SIZE = 4;
```

Next, we need to update how you render your square so that you can fit `GRID_SIZE` times `GRID_SIZE` of them on the canvas

- *Could* approach this is by making the vertex buffer significantly bigger and defining `GRID_SIZE` times `GRID_SIZE` worth of squares inside it at the right size and position
- But that's also not making the best use of the GPU and using more memory than necessary to achieve the effect

## Create a Uniform Buffer

First, you need to communicate the grid size you've chosen to the shader, since it uses that to change how things display

Could just hard-code the size into the shader

- But then that means that any time you want to change the grid size you have to re-create the shader and render pipeline, which is expensive

better way is to provide the grid size to the shader as **uniforms**

Learned earlier that a different value from the vertex buffer is passed to every invocation of a vertex shader

A uniform is a value from a buffer that is the same for every invocation

- Useful for communicating values that are common for a piece of geometry (like its position), a full frame of animation (like the current time), or even the entire lifespan of the app (like a user preference)

We can create a uniform buffer like so:

```jsx
// Create a uniform buffer that describes the grid.
const uniformArray = new Float32Array([GRID_SIZE, GRID_SIZE]);
const uniformBuffer = device.createBuffer({
  label: "Grid Uniforms",
  size: uniformArray.byteLength,
  usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST,
});
device.queue.writeBuffer(uniformBuffer, 0, uniformArray);
```

Almost exactly the same code that you used to create the vertex buffer earlier

- Main difference being that the *`usage`* this time includes `GPUBufferUsage.UNIFORM` instead of `GPUBufferUsage.VERTEX`

## Access Uniforms in a Shader

We can define a uniform by adding the following code

```rust
// At the top of the `code` string in the createShaderModule() call
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {
  return vec4f(pos / grid, 0, 1);
}

// ...fragmentMain is unchanged
```

- This defines a uniform in your shader called `grid`, which is a 2D float vector that matches the array that you just copied into the uniform buffer
- It also specifies that the uniform is bound at `@group(0)` and `@binding(0)`

Then, elsewhere in the shader code, you can use the grid vector however you need.

- In this code you divide the vertex position by the grid vector

Since `pos` is a 2D vector and `grid` is a 2D vector, WGSL performs a component-wise division.

- In other words, the result is the same as saying `vec2f(pos.x / grid.x, pos.y / grid.y)`

What this means in your case is that (if you used a grid size of 4) the square that you render would be one-fourth of its original size. That's perfect if you want to fit four of them to a row or column!

## Create a Bind Group

Declaring the uniform in the shader doesn't connect it with the buffer that you created, though. In order to do that, you need to create and set a *bind group*

A **bind group** is a collection of resources that you want to make accessible to your shader at the same time

- It can include several types of buffers, like your uniform buffer, and other resources like textures and samplers that are not covered here

Create a bind group with your uniform buffer by adding the following code after the creation of the uniform buffer and render pipeline

```jsx
const bindGroup = device.createBindGroup({
  label: "Cell renderer bind group",
  layout: cellPipeline.getBindGroupLayout(0),
  entries: [{
    binding: 0,
    resource: { buffer: uniformBuffer }
  }],
});
```

In addition to your now-standard `label`, you also need a `layout` that describes which types of resources this bind group contains

- Dig into further in a future step, but for the moment you can happily ask your pipeline for the bind group layout because you created the pipeline with `layout: "auto"`.
- That causes the pipeline to create bind group layouts automatically from the bindings that you declared in the shader code itself

After specifying the layout, we can provide an array of `entries`

Each entry is a dictionary with at least the following values:

- **`binding`**, which corresponds with the `@binding()` value you entered in the shader. In this case, `0`.
- **`resource`**, which is the actual resource that you want to expose to the variable at the specified binding index. In this case, your uniform buffer.

The function returns a `[GPUBindGroup](https://gpuweb.github.io/gpuweb/#gpubindgroup)`, which is an opaque, immutable handle

- Can't change the resources that a bind group points to after it's been created, though you *can* change the contents of those resources

## Bind the Bind Group

Now that the bind group is created, you still need to tell WebGPU to use it when drawing. Fortunately this is pretty simple

Hop back down to the render pass and add this new line before the `draw()` method:

```jsx
pass.setPipeline(cellPipeline);
pass.setVertexBuffer(0, vertexBuffer);

pass.setBindGroup(0, bindGroup); // New line!

pass.draw(vertices.length / 2);
```

- The `0` passed as the first argument corresponds to the `@group(0)` in the shader code
- Saying that each `@binding` that's part of `@group(0)` uses the resources in this bind group

Your square is now one-fourth the size it was before!

## Manipulate Geometry in the Shader

So now that you can reference the grid size in the shader, you can start doing some work to manipulate the geometry you're rendering to fit your desired grid pattern

To do that, consider exactly what you want to achieve

- Divide up your canvas into individual cells
- In order to keep the convention that the X axis increases as you move right and the Y axis increases as you move up, say that the first cell is in the bottom left corner of the canvas

That gives you a layout that looks like this, with your current square geometry in the middle:

![Untitled](Draw%20a%20Grid/Untitled.png)

Challenge is to find a method in the shader that lets you position the square geometry in any of those cells given the cell coordinates

First, you can see that your square isn't nicely aligned with any of the cells because it was defined to surround the center of the canvas

- You'd want to have the square shifted by half a cell so that it would line up nicely inside them

One way you could fix this is to update the square's vertex buffer

- By shifting the vertices so that the bottom-right corner is at, for example, (0.1, 0.1) instead of (-0.8, -0.8), you'd move this square to line up with the cell boundaries more nicely
- But, since you have full control over how the vertices are processed in your shader, it's just as easy to simply nudge them into place using the shader code

We alter the vertex shader module with the following code:

```rust
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {

  // Add 1 to the position before dividing by the grid size.
  let gridPos = (pos + 1) / grid;

  return vec4f(gridPos, 0, 1);
}
```

- **Caution:** For the sake of making it easier to read, the position calculation is broken out into its own variable. Notice that it's declared using `let`, but be aware that `let` in WGSL has a different meaning than `let` in JavaScript, behaves like rust!
    - In WGSL, `let` behaves more like JavaScript's `const`
    - If you do need to change a variable after declaring it in WGSL, use `var` instead.

This moves every vertex up and to the left by one (which, remember, is half of the clip space) *before* dividing it by the grid size

The result is a nicely grid-aligned square just off of the origin

![Untitled](Draw%20a%20Grid/Untitled%201.png)

Next, because your canvas's coordinate system places (0, 0) in the center and (-1, -1) in the lower left, and you want (0, 0) to be in the lower left, you need to translate your geometry's position by (-1, -1) *after* dividing by the grid size in order to move it into that corner

```rust
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {

  // Subtract 1 after dividing by the grid size.
  let gridPos = (pos + 1) / grid - 1;

  return vec4f(gridPos, 0, 1); 
}
```

And now your square is nicely positioned in cell (0, 0)!

![Untitled](Draw%20a%20Grid/Untitled%202.png)

What if you want to place it in a different cell?

you want to move the square only by one grid unit (one-fourth of the canvas) for each cell

Sounds like you need to do another divide by `grid`!

```rust
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {

  let cell = vec2f(1, 1); // Cell(1,1) in the image above
  let cellOffset = cell / grid; // Compute the offset to cell
  let gridPos = (pos + 1) / grid - 1 + cellOffset; // Add it here!

  return vec4f(gridPos, 0, 1);
}
```

If you refresh now, you see the following:

![Untitled](Draw%20a%20Grid/Untitled%203.png)

Hm. Not quite what you wanted.

The reason for this is that since the canvas coordinates go from -1 to +1, it's *actually 2 units across*. That means if you want to move a vertex one-fourth of the canvas over, you have to move it 0.5 units

We just need to multiply your offset by 2, like this

```rust
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f) ->
  @builtin(position) vec4f {

  let cell = vec2f(1, 1);
  let cellOffset = cell / grid * 2; // Updated
  let gridPos = (pos + 1) / grid - 1 + cellOffset;

  return vec4f(gridPos, 0, 1);
}
```

That gives exactly what we want:

![Untitled](Draw%20a%20Grid/Untitled%204.png)

Furthermore, you can now set `cell` to any value within the grid bounds, and then refresh to see the square render in the desired location

## Drawing Instances

Now that you can place the square where you want it with a bit of math, the next step is to render one square in each cell of the grid

- One way you could approach it is to write cell coordinates to a uniform buffer, then call *draw* once for each square in the grid, updating the uniform every time
    - Would be very slow, since the GPU has to wait for the new coordinate to be written by JavaScript every time

Instead, you can use a technique called instancing

**Instancing** is a way to tell the GPU to draw multiple copies of the same geometry with a single call to `draw`, which is much faster than calling `draw` once for every copy

Each copy of the geometry is referred to as an *instance*

To tell the GPU that you want enough instances of your square to fill the grid, add one argument to your existing draw call:

```jsx
pass.draw(vertices.length / 2, GRID_SIZE * GRID_SIZE);
```

This tells the system that you want it to draw the six (`vertices.length / 2`) vertices of your square 16 (`GRID_SIZE * GRID_SIZE`) times

But if you refresh the page, you still see the same output: A single square

- This is because you draw all 16 of those squares in the same spot

We need to add additional logic in the shader that repositions the geometry on a per-instance basis

In the shader, in addition to the vertex attributes like `pos` that come from your vertex buffer, you can also access what are known as WGSL's *[built-in values](https://gpuweb.github.io/gpuweb/wgsl/#builtin-values)*.

****************************************Built in values**************************************** are calculated by WebGPU, and one such value is the `instance_index`

`instance_index` is an unsigned 32-bit number from `0` to `number of instances - 1` that you can use as part of your shader logic

- Its value is the same for every vertex processed that's part of the same instance
- That means your vertex shader gets called six times with an `instance_index` of `0`, once for each position in your vertex buffer
    - Then six more times with an `instance_index` of `1`, then six more with `instance_index` of `2`...

To see this in action, you have to add the `instance_index` built-in to your shader inputs

Do this in the same way as the position, but instead of tagging it with a `@location` attribute, use `@builtin(instance_index)`, and then name the argument whatever you want

- (You can call it `instance` to match the example code.) Then use it as part of the shader logic

```rust
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f,
              @builtin(instance_index) instance: u32) ->
  @builtin(position) vec4f {
  
  let i = f32(instance); // Save the instance_index as a float
  let cell = vec2f(i, i);
  let cellOffset = cell / grid * 2; // Updated
  let gridPos = (pos + 1) / grid - 1 + cellOffset;

  return vec4f(gridPos, 0, 1);
}
```

**Caution:** This is an area where WGSL's strong typing may trip up developers who are more familiar with JavaScript than other strongly typed languages. The `instance_index` is a `u32`, or unsigned 32-bit integer. But you want `cell` to be a vector of floating point values because that's how the position math is done. As a result, you have to explicitly tell the shader to treat `instance` as a float when creating the `cell` vector by wrapping it in a `f32()` function. This is known as *casting* the type.

If you refresh now you see that you do indeed have more than one square! But you can't see all 16 of them.

![Untitled](Draw%20a%20Grid/Untitled%205.png)

That's because the cell coordinates you generate are (0, 0), (1, 1), (2, 2)... all the way to (15, 15), but only the first four of those fit on the canvas

To make the grid that you want, you need to transform the `instance_index` such that each index maps to a unique cell within your grid, like this

![Untitled](Draw%20a%20Grid/Untitled%206.png)

- The math for that is reasonably straightforward. For each cell's X value, you want the [modulo](https://en.wikipedia.org/wiki/Modulo) of the `instance_index` and the grid width, which you can perform in WGSL with the `%` operator
- for each cell's Y value you want the `instance_index` divided by the grid width, discarding any fractional remainder. You can do that with WGSL's `[floor()](https://gpuweb.github.io/gpuweb/wgsl/#floor-builtin)` function

```rust
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f,
              @builtin(instance_index) instance: u32) ->
  @builtin(position) vec4f {

  let i = f32(instance);
  // Compute the cell coordinate from the instance_index
  let cell = vec2f(i % grid.x, floor(i / grid.x));

  let cellOffset = cell / grid * 2;
  let gridPos = (pos + 1) / grid - 1 + cellOffset;

  return vec4f(gridPos, 0, 1);
}
```

We now have a grid of cells:

![Untitled](Draw%20a%20Grid/Untitled%207.png)

We can now modify the `GRID_SIZE` variable to create a `GRID_SIZE`**x**`GRID_SIZE` grid

# Make it more Colorful

## Use structs in shaders

Until now, you've passed one piece of data out of the vertex shader: the transformed position. But you can actually return a lot more data from the vertex shader and then use it in the fragment shader!

The only way to pass data out of the vertex shader is by returning it

A vertex shader is always required to return a position, so if you want to return any other data along with it, you need to place it in a struct

Structs in WGSL are named object types that contain one or more named properties. The properties can be marked up with attributes like `@builtin` and `@location` too.

You declare them outside of any functions, and then you can pass instances of them in and out of functions, as needed

```rust
// Current Code 
@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(@location(0) pos: vec2f,
              @builtin(instance_index) instance: u32) -> 
  @builtin(position) vec4f {

  let i = f32(instance);
  let cell = vec2f(i % grid.x, floor(i / grid.x));
  let cellOffset = cell / grid * 2;
  let gridPos = (pos + 1) / grid - 1 + cellOffset;
  
	return  vec4f(gridPos, 0, 1);
};

// ------- Can be expressed using structs for the function input and output ---------

struct VertexInput {
  @location(0) pos: vec2f,
  @builtin(instance_index) instance: u32,
};

struct VertexOutput {
  @builtin(position) pos: vec4f,
};

@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(input: VertexInput) -> VertexOutput  {
  let i = f32(input.instance);
  let cell = vec2f(i % grid.x, floor(i / grid.x));
  let cellOffset = cell / grid * 2;
  let gridPos = (input.pos + 1) / grid - 1 + cellOffset;
  
  var output: VertexOutput;
  output.pos = vec4f(gridPos, 0, 1);
  return output;
}
```

**Note:** This requires you to refer to the input position and instance index with `input`, and the struct that you return first needs to be declared as a variable and have its individual properties set

In this case, it doesn't make too much difference, and in fact makes the shader function a bit longer, but as your shaders grow more complex, using structs can be a great way to help organize your data

## Pass Data Between The Vertex and Fragment Functions

The curent fragment function doesn’t take any inputs and returns a solid color (red) as the output

If the shader knew more about the geometry that it's coloring, though, you could use that extra data to make things a bit more interesting

- For instance, what if you want to change the color of each square based on its cell coordinate

The `@vertex` stage knows which cell is being rendered; you just need to pass it along to the `@fragment` stage

- To pass any data between the vertex and fragment stages, you need to include it in an output struct with a `@location` of our choice

Since you want to pass the cell coordinate, add it to the `VertexOutput` struct from earlier, and then set it in the `@vertex` function before you return

```rust
struct VertexInput {
  @location(0) pos: vec2f,
  @builtin(instance_index) instance: u32,
};

struct VertexOutput {
  @builtin(position) pos: vec4f,
  @location(0) cell: vec2f, // New line!
};

@group(0) @binding(0) var<uniform> grid: vec2f;

@vertex
fn vertexMain(input: VertexInput) -> VertexOutput  {
  let i = f32(input.instance);
  let cell = vec2f(i % grid.x, floor(i / grid.x));
  let cellOffset = cell / grid * 2;
  let gridPos = (input.pos + 1) / grid - 1 + cellOffset;
  
  var output: VertexOutput;
  output.pos = vec4f(gridPos, 0, 1);
  output.cell = cell; // New line!
  return output;
}
```

In the `@fragment` function, receive the value by adding an argument with the same `@location`

```rust
@fragment
fn fragmentMain(@location(0) cell: vec2f) -> @location(0) vec4f {
  // Remember, fragment return values are (Red, Green, Blue, Alpha)
  // and since cell is a 2D vector, this is equivalent to:
  // (Red = cell.x, Green = cell.y, Blue = 0, Alpha = 1)
  return vec4f(cell, 0, 1);
}
```

Alternatively, could reuse the `VertexOutput` a struct as well:

```rust
@fragment
fn fragmentMain(input: VertexOutput) -> @location(0) vec4f {
  return vec4f(input.cell, 0, 1);
}
```

If you want a smoother transition between colors, you need to return a fractional value for each color channel, ideally starting at zero and ending at one along each axis, which means yet another divide by `grid`!

```rust
@fragment
fn fragmentMain(input: VertexOutput) -> @location(0) vec4f {
  return vec4f(input.cell/grid, 0, 1);
}
```

![Untitled](Draw%20a%20Grid/Untitled%208.png)

While that's certainly an improvement, there's now an unfortunate dark corner in the lower left, where the grid becomes black

Fortunately, you have a whole unused color channel—blue—that you can use

The effect that you ideally want is to have the blue be brightest where the other colors are darkest, and then fade out as the other colors grow in intensity

The easiest way to do that is to have the channel *start* at 1 and subtract one of the cell values. It can be either `c.x` or `c.y`. Try both, and then pick the one you prefer!

```rust
@fragment
fn fragmentMain(input: VertexOutput) -> @location(0) vec4f {
  let c = input.cell / grid;
  return vec4f(c, 1-c.x, 1);
}
```
# Physics Simulation

## Exchanging Data

Won’t be using WebGPU for graphics directly, so instead I thought it’d be fun to run a physics simulation on the GPU and visualize it using Canvas2D

**For this to work, we need to push some simulation parameters and the initial state *to* the GPU, run the simulation *on* the GPU and read the simulation results *from* the GPU**

- A**rguably the hairiest part of WebGPU, as there’s a bunch of data acrobatics but this is what allows WebGPU to be a device-agnostic API running at the highest level of performance**

### Bind Group Layouts

To exchange data with the GPU, we need to **extend our pipeline definition with a bind group layout**

- The **bind group *layout*** pre-defines the types, purposes and uses of these GPU entities, which allows the GPU figure out how to run a pipeline most efficiently ahead of time

A **bind group** is a collection of GPU entities (memory buffers, textures, samplers, etc) that are made accessible during the execution of the pipeline

Let’s keep it simple in this initial step and give our pipeline access to a single memory buffer

```jsx
const bindGroupLayout =
 device.createBindGroupLayout({
    entries: [{
      binding: 1,
      visibility: GPUShaderStage.COMPUTE,
      buffer: {
        type: "storage",
      },
    }],
  });

const pipeline = device.createComputePipeline({
  layout: device.createPipelineLayout({
    bindGroupLayouts: [bindGroupLayout],
  }),
  compute: {
    module,
    entryPoint: "main",
  },
});
```

- Bind Group Layout:
    - The **binding number** (1) can be freely chosen and is used to tie a variable in our WGSL code to the contents of the buffer in this slot of the bind group layout
    - Our `bindGroupLayout` also defines the purpose for each buffer, which in this case is "`storage`"
        - Another option is "`read-only-storage`", which is read-only (duh!), and allows the GPU to make further optimizations on the basis that this buffer will never be written to and as such doesn’t need to be synchronized
        - The last possible value for the buffer type is "`uniform`", which in the context of a compute pipeline is mostly functionally equivalent to a storage buffer
- Bind Group:
    - Contains the actual instances of the GPU entities the bind group layout expects
    - Once that bind group with the buffer inside is in place, the compute shader can fill it with data and we can read it from the GPU. But there’s a hurdle: Staging Buffers.

### Staging Buffers

To accommodate today’s graphics demands, (multiple GB/s for low resolution) GPUs need shovel data even faster

- This is only possible to achieve if the memory of the GPU is very tightly integrated with the cores
- This tight integration makes it hard to also expose the same memory to the host machine for reading and writing

GPUs have additional memory banks that are accessible to both the host machine as well as the GPU, but are not as tightly integrated and can’t provide data as fast

**Staging buffers** are buffers that are allocated in this intermediate memory realm and can be mapped to the host system for reading and writing

- To read data from the GPU, we copy data from an internal, high-performance buffer to a staging buffer, and then map the staging buffer to the host machine so we can read the data back into main memory ↔ The reverse for writing

**Back to our code:**

We will create a **writable buffer** and add it to the **bind group,** so that it can be **written to by the compute shader**

We will also create a second buffer with the same size that will act as a **staging buffer**

```jsx
const BUFFER_SIZE = 1000;

const output = device.createBuffer({
  size: BUFFER_SIZE,
  usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_SRC
});

const stagingBuffer = device.createBuffer({
  size: BUFFER_SIZE,
  usage: GPUBufferUsage.MAP_READ | GPUBufferUsage.COPY_DST,
});

const bindGroup = device.createBindGroup({
  layout: bindGroupLayout,
  entries: [{
    binding: 1,
    resource: {
      buffer: output,
    },
  }],
});
```

- Each buffer is created with a **usage bitmask**, where you can declare how you intend to use that buffer
- The GPU will then figure out where the buffer should be located to fulfill all these use-cases or throw an error if the combination of flags is unfulfillable

**Note:**`createBuffer()` returns a `GPUBuffer`, not an `ArrayBuffer.`They can’t be read or written to just yet

- For that, they need to be mapped, which is a separate API call and will only succeed for buffers that have `GPUBufferUsage.MAP_READ` or 
`GPUBufferUsage.MAP_WRITE`

Now that we not only have the bind group *layout*, but even the actual bind group itself, we need to update our dispatch code to make use of this bind group

- Afterwards we map our staging buffer to read the results back into JavaScript

```jsx
const commandEncoder = device.createCommandEncoder();
const passEncoder = commandEncoder.beginComputePass();
passEncoder.setPipeline(pipeline);
passEncoder.setBindGroup(0, bindGroup);
passEncoder.dispatchWorkgroups(1);
passEncoder.dispatchWorkgroups(Math.ceil(BUFFER_SIZE / 64));
passEncoder.end();
commandEncoder.copyBufferToBuffer(
  output,
  0, // Source offset
  stagingBuffer,
  0, // Destination offset
  BUFFER_SIZE
);
const commands = commandEncoder.finish();
device.queue.submit([commands]);

await stagingBuffer.mapAsync(
  GPUMapMode.READ,
  0, // Offset
  BUFFER_SIZE // Length
 );
const copyArrayBuffer =
  stagingBuffer.getMappedRange(0, BUFFER_SIZE);
const data = copyArrayBuffer.slice();
stagingBuffer.unmap();
console.log(new Float32Array(data));
```

- Since we added a bind group layout to our pipeline, any invocation without providing a bind group would now fail
- After we define our “pass”, we add an additional command via our command encoder to copy the data from our output buffer to the staging buffer and submit our command buffer to the queue
- The GPU will start working through the command queue
- We don’t know when the GPU will be done exactly, but we can already submit our request for the `stagingBuffer` to be mapped
    - This function is async as it needs to wait until the command queue has been fully processed
- When the returned promise resolves, the buffer is mapped, but not exposed to JavaScript yet
    - `stagingBuffer.getMappedRange()` let’s us request for a subsection (or the entire buffer) to be exposed to JavaScript as a good ol’  `ArrayBuffer`
    - This is real, mapped GPU memory, meaning the data will disappear (the `ArrayBuffer` will be “detached”), when 
    `stagingBuffer`gets unmapped, so I’m using `slice()` to create JavaScript-owned copy
    

![Untitled](Physics%20Simulation/Untitled.png)

Something other than zeroes would probably be a bit more convincing Before we start doing any advanced calculation on our GPU, let’s put some hand-picked data into our buffer as proof that our pipeline is *indeed* working as intended

**This is our new compute shader code:**

```rust
@group(0) @binding(1)
var<storage, read_write> output: array<f32>;

@compute @workgroup_size(64)
fn main(

  @builtin(global_invocation_id)
  global_id : vec3<u32>,

  @builtin(local_invocation_id)
  local_id : vec3<u32>,

) {
  output[global_id.x] =
    f32(global_id.x) * 1000. + f32(local_id.x);
}
```

The first two lines declare a module-scope variable called `output`, which is a dynamically-sized array of **f32**

- The attributes declare where the data comes from: From the **buffer in our first (0th) binding group**, the entry with **binding value 1**
- The length of the array will automatically reflect the length of the underlying buffer (rounded down)

The signature of our `main()` function has been augmented with two parameters: 

- `global_id`
- `local_id`

Could have chosen any name — their value is determined by the attributes associated with them:

- The `global_invocation_id` is a built-in value that corresponds to the global x/y/z coordinates of this shader invocation in the work 
*load*
- The `local_invocation_id` is the x/y/z coordinates of this shader vocation in the work *group*

**An example of three work items a, b and c marked in the workload.**

![Untitled](Physics%20Simulation/Untitled%201.png)

This image shows one possible interpretation of the coordinate system for a workload with `@workgroup_size(4, 4, 4)`

If we agreed on the axes as drawn above, we’d see the following `main()` parameters for a, b and c:

- **a:**
    - `**local_id=(x=0, y=0, z=0)**`
    - `**global_id=(x=0, y=0, z=0)**`
- **b:**
    - `**local_id=(x=0, y=0, z=0)**`
    - `**global_id=(x=4, y=0, z=0)**`
- **c:**
    - `**local_id=(x=1, y=1, z=0)**`
    - `**global_id=(x=5, y=5, z=0**)`

In our shader, we have `@workgroup_size(64, 1, 1)`, so `local_id.x` will range from 0 to 63

- To be able to inspect both values, I am “encoding” them into a single number

![Untitled](Physics%20Simulation/Untitled%202.png)

**Note:** Actual values filled in by the GPU. Notice how the local invocation ID starts wrapping around after 63, while the global invocation ID keeps going

And this proves that our compute shader is indeed invoked for each value in the output memory and fills it with a unique value

We won’t know in which order this data has been filled in, as that’s intentionally unspecified and left up to the GPU’s scheduler

### Overdispatching

The astute observer might have noticed that the total number of shader invocations `(Math.ceil(BUFFER_SIZE / 64) * 64)` will result in `global_id.x` getting bigger than the length of our array, as each f32 takes up 4 bytes

- Luckily, accessing an array is safe-guarded by an implicit clamp, so every write past the end of the array will end up writing to the last element of the array
- That avoids memory access faults, but might still generate unusable data

And indeed, if you check the last 3 elements of the returned buffer, you’ll find the numbers 247055, 248056 and 608032. 

- It’s up to us to prevent that from happening in our shader code with an early exit:

```rust
fn main( /* ... */) {
  if(global_id.x >= arrayLength(&output)) {
    return;
  }
  output[global_id.x] =
    f32(global_id.x) * 100. + f32(local_id.x);
}
```

### A structure for the madness

Our goal here is to have a whole lotta balls moving through 2D space and have happy little collisions

For that, each ball needs to have a radius, a position and a velocity vector

- We could just continue working on array<f32>, and say the first float is the first ball’s x position, the second float is the first ball’s y position and so on
- But that’s not ergonomic!

Luckily, WGSL allows us to define our own structs to tie multiple pieces of data together in a neat bag.

So it makes sense to define a `struct Ball` with all these components and turn our `array<f32>` into `array<Ball>`

- Problem: **alignment**

### Alignment

If we try running this code:

```rust
struct Ball {
  radius: f32,
  position: vec2<f32>,
  velocity: vec2<f32>,
}

@group(0) @binding(1)
var<storage, read_write> output: array<f32>;
var<storage, read_write> output: array<Ball>;

@compute @workgroup_size(64)
fn main(
  @builtin(global_invocation_id) global_id : vec3<u32>,
  @builtin(local_invocation_id) local_id : vec3<u32>,
) {
  let num_balls = arrayLength(&output);
  if(global_id.x >= num_balls) {
    return;
  }

  output[global_id.x].radius = 999.;
  output[global_id.x].position = vec2<f32>(global_id.xy);
  output[global_id.x].velocity = vec2<f32>(local_id.xy);
}
```

We should see this in the console: (I put 999 the first field of the struct to make it easy to see where the struct begins in the buffer)

![Untitled](Physics%20Simulation/Untitled%203.png)

- There’s a total of 6 numbers until we reach the next 999, which is a bit surprising because the struct really only has 5 numbers to store: radius, position.x, position.y, velocity.x and velocity.y
- Taking a closer look, it is clear that the number after `radius` is always 0. This is because of alignment.

Each WGSL data type has well-defined **[alignment requirements](https://gpuweb.github.io/gpuweb/wgsl/#alignment-and-size)**

![Untitled](Physics%20Simulation/Untitled%204.png)

If a data type has an alignment of *N*, it means that a value of that data type can only be stored at a memory address that is a multiple of *N*

- `f32` has an alignment of **4**, while `vec2<f32>` has an alignment of **8**
- **If we assume our struct starts at address 0, then the radius field can be stored at address 0, as 0 is a multiple of 4**
- The next field in the struct is `vec2<f32>`, which has an alignment of **8**. But, the first free address after `radius` is **4**, which is ***not* a multiple of** **8**
- To remedy this, the compiler adds padding of 4 bytes to get to the next address that is a multiple of 8

This explains what we see an unused field with the value 0 in the DevTools console

Now that we know how our struct is laid out in memory, we can populate it from JavaScript to generate our initial state of balls and also read it back to visualize it

### Input & Output

We have successfully managed to read data from the GPU, bring it to JavaScript and “decode” it

It’s now time to tackle the other direction

- We need to generate the initial state of all our balls in JavaScript and give it to the GPU so it can run the compute shader on it

Generating the initial state is fairly straight forward:

```jsx
let inputBalls = new Float32Array(new ArrayBuffer(BUFFER_SIZE));
for (let i = 0; i < NUM_BALLS; i++) {
  inputBalls[i * 6 + 0] = randomBetween(2, 10); // radius
  inputBalls[i * 6 + 1] = 0; // padding
  inputBalls[i * 6 + 2] = randomBetween(0, ctx.canvas.width); // position.x
  inputBalls[i * 6 + 3] = randomBetween(0, ctx.canvas.height); // position.y
  inputBalls[i * 6 + 4] = randomBetween(-100, 100); // velocity.x
  inputBalls[i * 6 + 5] = randomBetween(-100, 100); // velocity.y
}
```

We also already know how to expose a buffer to our shader

We just need to adjust our pipeline bind group layout to expect another buffer:

```jsx
const bindGroupLayout = device.createBindGroupLayout({
  entries: [
    {
      binding: 0,
      visibility: GPUShaderStage.COMPUTE,
      buffer: {
        type: "read-only-storage",
      },
    },
    {
      binding: 1,
      visibility: GPUShaderStage.COMPUTE,
      buffer: {
        type: "storage",
      },
    },
  ],
});
```

... and create a GPU buffer that we can bind using our bind group:

```jsx
const input = device.createBuffer({
  size: BUFFER_SIZE,
  usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_DST,
});

const bindGroup = device.createBindGroup({
  layout: bindGroupLayout,
  entries: [
    {
      binding: 0,
      resource: {
        buffer: input,
      },
    },
    {
      binding: 1,
      resource: {
        buffer: output,
      },
    },
  ],
});
```

### Sending Data to the GPU

Now for the new part: Sending data to the GPU

Just like with reading data, we technically have to create a staging buffer that we can map, copy our data into the staging buffer and then issue a command to copy our data from the staging buffer into the storage buffer

However, WebGPU offers a convenience function that will choose the most efficient way of getting our data into the storage buffer for us, even if that involves creating a temporary staging buffer on the fly

```jsx
device.queue.writeBuffer(input, 0, inputBalls);
```

That’s it! We don’t even need a command encoder

We can just put this command directly into the **command queue.** 

- `device.queue` offers some other, similar convenience functions for textures as well.

Now we need to bind this new buffer to a variable in WGSL and do something with it:

```rust
struct Ball {
  radius: f32,
  position: vec2<f32>,
  velocity: vec2<f32>,
}

@group(0) @binding(0)
var<storage, read> input: array<Ball>;

@group(0) @binding(1)
var<storage, read_write> output: array<Ball>;

const TIME_STEP: f32 = 0.016;

@compute @workgroup_size(64)
fn main(
  @builtin(global_invocation_id)
  global_id : vec3<u32>,
) {
  let num_balls = arrayLength(&output);
  if(global_id.x >= num_balls) {
    return;
  }
  output[global_id.x].position =
    input[global_id.x].position +
    input[global_id.x].velocity * TIME_STEP;
}
```

Lastly, all we need to do is read the output buffer back into JavaScript, write some Canvas2D code to visualize the contents of the buffer and put it all in a `requestAnimationFrame()` loop
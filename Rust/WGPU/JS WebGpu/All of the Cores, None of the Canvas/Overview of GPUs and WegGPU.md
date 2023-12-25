# Overview of GPUs and WegGPU

# Summary

To get access to an adapter, you call  `navigator.gpu.requestAdapter()`

- `requestAdapter()`takes very few options
- The options allow you to request a high-performance or low-energy adapter

If this succeeds, i.e. the returned adapter is non-null:

You can inspect the adapter’s capabilities and request a logical device from the adapter using `adapter.requestDevice()`

- Without any options, `requestDevice()`will return a device that does *not*  necessarily match the physical device’s capabilities
    - Rather what the WebGPU team considers a reasonable, lowest common denominator of all GPUs
- If necessary, you can inspect the real limits of the physical GPU via `adapter.limits` and request a  device with raised limits by passing an options object to `requestDevice()`

## Shaders

Note: in the olden days (i.e. late 1980s), the term shaders was appropriate: A small piece of code that ran on the GPU to decide for each pixel what color it should be so that you could *shade* the objects being rendered

- Nowadays, shaders loosely refer to any program that runs on the GPU

### Rundown of how shaders work:

- You upload a data buffer to your GPU and tell it how to interpret that data as a series of triangles
- Each vertex occupies a chunk of that data buffer, describing that vertex’ position in 3D space, but probably also auxillary data like color, texture IDs, normals and other things
- Each vertex in the list is processed by the GPU in the *vertex stage*, running the *vertex shader* on each vertex, which will apply translation, rotation or perspective distortion
- The GPU now rasterizes the triangles, meaning the GPU figures out which pixels each triangle covers on the screen
- Each pixel is then processed by the *fragment shader*, which has access to the pixel coordinates but also the auxillary data to decide which color that pixel should be

This system of passing data to a vertex shader, then to a fragment shader and then outputting it directly onto the screen is called a *pipeline*

- In WebGPU you have to explicitly define your pipeline

### Pipelines

Currently, WebGPU allows you to create two types of pipelines: 

- Render Pipeline
    - Render Pipeline renders something, meaning it creates a 2D image
    - That image needn’t be on screen, but could just be rendered to memory (which is called a Framebuffer)
- Compute Pipeline
    - A Compute Pipeline is more generic in that it returns a buffer, which can contain any sort of data

With WebGPU, a pipeline consists of one (or more) programmable stages, where each stage is defined by a shader and an entry point

- A Compute Pipline has a single compute stage, while a Render Pipeline would have a vertex and a  fragment stage

**Example:** Compute Shader and pipeline

```jsx
const module = device.createShaderModule({
  code: `
    @compute @workgroup_size(64)
    fn main() {
      // Pointless!
    }
  `,
});

const pipeline = device.createComputePipeline({
  compute: {
    module,
    entryPoint: "main",
  },
});
```

- In the shader module above we are just creating a function called main and marking it as an entry point for the compute stage by using the `@compute` attribute
- You can have multiple functions marked as an entry point in a shader module, as you can reuse the same shader module for multiple pipelines and choose different functions to invoke via the entryPoint options

But what is that `@workgroup_size(64)` attribute?

### Parrallelism

GPUs are optimized for throughput at the cost of latency

To understand this, we have to look a bit at the architecture of GPUs

- GPUs have an extensive number of cores that allow for massively parallel work
- However, the cores are not as independent as you might be used to from when programming for a multi-core CPU
    - Firstly, GPU cores are grouped hierarchically
        - In the case of Intel, the lowest level in the hierarchy is the “**Execution Unit**” (EU), which has multiple (in this case seven) SIMT cores
        - Means it has seven cores that operate in lock-step and always execute the same instructions
        - However, each core has its own set of registers and stack pointer
        - So while they *have* to execute the same operation, they can execute it on different data
        - Avoid looping and if else statements: all cores have to execute *both* branches, unless all cores happen to take the same branch
    - Despite the core’s frequency, getting data from memory (or pixels from textures) still takes relatively long: A couple hundred cycles
        - Whenever an EU would end up idling (e.g. to wait for a value from memory), it instead switches to another work item and will only switch back once the new work item needs to wait for something
        - This is the key trick how GPUs optimize for throughput at the cost of latency: Individual work items will take longer as a switch to another work item might stop execution for longer than necessary, but the overall utilization is higher and results in a higher throughput
- EUs are just the lowest level in the hierarchy, though. Multiple EUs are grouped into what Intel calls a “**SubSlice**”
    - All the EUs in a SubSlice have access to a small amount of **Shared Local Memory** (SLM)
    - If the program to be run has any synchronization commands, it has to be executed within the same SubSlice, as only they have shared memory for synchronization
    - In the last layer multiple SubSlices are grouped into a Slice, which forms the GPU
    
    To fully exploit the benefits of this architecture, programs need to be specifically set up for this architecture so that a purely programmatic GPU scheduler can maximize utilization
    
    As a result, graphics APIs expose a threading model that naturally allows for work to be dissected this way
    
    In WebGPU, the important primitive here is the “**workgroup**”.
    

### Workgroups

In the traditional setting, the vertex shader would get invoked once for each vertex, and the fragment shader once for each pixel

In the GPGPU setting, your compute shader will be **invoked** **once** **for** **each work item that you schedule**

- It is up to you to define what a work item is

The **collection of all work items** (which I will call the “**workload**”) is broken down into **workgroups**

All **work items** in a **workgroup** are scheduled to run together

In WebGPU, the work load is modelled as a 3-dimensional grid, where **each “cube” is a work item**, and **work items are grouped** into bigger cuboids to form a **workgroup**

This is a workload. White-bordered cubes are a work item. Red-bordered cuboids are a workgroup

![Untitled](Overview%20of%20GPUs%20and%20WegGPU/Untitled.png)

Finally, we have enough information to talk about the `@workgroup_size(x, y, z)` attribute, and it might even be mostly self-explanatory at this point:

- The attribute allows you to tell the GPU what the the size of a **workgroup** for this shader should be
- In the language of the picture above, the `@workgroup_size` attribute defines the size of the red-bordered cubes →  $x * y * z$ is the **numberof work items per work group**
- **Any skipped parameter is assumed to be 1, so `@workgroup_size(64)` is equivalent to  `@workgroup_size(64, 1, 1)`**

T**he actual EUs are not arranged in the 3D grid on the chip**

- A**im of modelling work items in a 3D grid is to increase locality**

A**ssumption is that it is likely that neighboring work groups will access similar areas in memory**

- S**o when running neighboring workgroups sequentially, the chances of already having values in the cache are higher, saving a couple of hundred cycles by not having to grab them from memory**
- **However, most hardware seemingly just runs workgroups in a serial order as the difference between running a shader with `@workgroup_size(64)` or `@workgroup_size(8, 8)` is negligible.**
    - **So this concept is considered somewhat legacy.**

**However, workgroups are restricted in multiple ways: `device.limits` has a bunch of properties that are worth knowing:**

```jsx
// device.limits
{
  // ...
  maxComputeInvocationsPerWorkgroup: 256,
  maxComputeWorkgroupSizeX: 256,
  maxComputeWorkgroupSizeY: 256,
  maxComputeWorkgroupSizeZ: 64,
  maxComputeWorkgroupsPerDimension: 65535,
  // ...
}
```

**Note:** The size of each dimension of a workgroup size is restricted, but even if x, y and z individually are within the limits, their product $x * y * z$ might not be

**So what *is* the right workgroup size?**

- Use [a workgroup size of] 64 unless you know what GPU you are targeting or that your workload needs something different

### Commands

We have written our shader and set up the pipeline. All that’s left to do is actually invoke the GPU to execute it all

As a GPU *can* be a completely separate card with it’s own memory chip, you control it via a so-called **command buffer** or **command queue**

- The **command queue** is a chunk of memory that contains encoded commands for the GPU to execute

```jsx
const commandEncoder = device.createCommandEncoder();
const passEncoder = commandEncoder.beginComputePass();
passEncoder.setPipeline(pipeline);
passEncoder.dispatchWorkgroups(1);
passEncoder.end();
const commands = commandEncoder.finish();
device.queue.submit([commands]);
```

`commandEncoder` has multiple methods that allows you to copy data from one GPU buffer to another and manipulate textures

- It also allows you to create PassEncoder, which encodes the setup and invocation of pipelines

In this case, we have a compute pipline, so we have to create a compute pass, set it to use our pre-declared pipeline

Finally call `dispatchWorkgroups(w_x, w_y, w_z)` to tell the GPU how many workgroups to create along each dimension

- In other words: The number of times our compute shader will be invoked is equal to:
    - $**w_x * w_y * w_z * z * y * z**$

Running this code, we are in fact spawning 64 threads on the GPU and they do *absolutely nothing*. 
But it works, so that’s cool. Let’s talk about how we give the GPU some data to work
# Initializing WebGPU

WebGPU can be used without showing anything on the screen if all you want is to use it to do computations

But if you want to render anything, like we're going to be doing in the codelab, you need a canvas

Create a new HTML document with a single `<canvas>` element in it, as well as a `<script>` tag where we query the canvas element

```html
<body>
  <canvas width="512" height="512"></canvas>
  <script src="index.js"></script>
</body>
```

## Request an Adapter and Device

To check if the `[navigator.gpu](https://gpuweb.github.io/gpuweb/#navigator-gpu)` object, which serves as the entry point for WebGPU, exists, add the following code:

```jsx
if (!navigator.gpu) {
  throw new Error("WebGpu not supported on this browser");
} else {
  console.log("WGPU supported");
}
```

Once you know that WebGPU is supported by the browser, the first step in initializing WebGPU for your app is to request a `[GPUAdapter](https://gpuweb.github.io/gpuweb/#gpuadapter)`

- Can think of an adapter as WebGPU's representation of a specific piece of GPU hardware in your device

To get an adapter, use the `[navigator.gpu.requestAdapter()](https://gpuweb.github.io/gpuweb/#dom-gpu-requestadapter)` method

- It returns a promise, so it's most convenient to call it with `await`

```jsx
const adapter = await navigator.gpu.requestAdapter();
if (!adapter) {
  throw new Error("No appropriate GPUAdapter found.");
}
```

If no appropriate adapters can be found, the returned `adapter` value might be `null`, so you want to handle that possibility

- It might happen if the user's browser supports WebGPU but their GPU hardware doesn't have all the features necessary to use WebGPU
- Most of the time it's OK to simply let the browser pick a default adapter
    - For more advanced needs there are [arguments that can be passed](https://gpuweb.github.io/gpuweb/#adapter-selection) to `requestAdapter()` that specify whether you want to use low-power or high-performance hardware on devices with multiple GPUs

Once you have an adapter, the last step before you can start working with the GPU is to request a [GPUDevice](https://gpuweb.github.io/gpuweb/#gpudevice)

- The device is the main interface through which most interaction with the GPU happens

Get the device by calling `[adapter.requestDevice()](https://gpuweb.github.io/gpuweb/#dom-gpuadapter-requestdevice)`, which also returns a promise

```jsx
const device = await adapter.requestDevice();
```

- As with `requestAdapter()`, there are [options that can be passed](https://gpuweb.github.io/gpuweb/#gpudevicedescriptor) here for more advanced uses like enabling specific hardware features or requesting higher limits

## Configure the Canvas

To do this, first request a `[GPUCanvasContext](https://gpuweb.github.io/gpuweb/#canvas-context)` from the canvas by calling `canvas.getContext("webgpu")`

- Same call used to initialize Canvas 2D or WebGL contexts, using the `2d` and `webgl` context types

The `context` that it returns must then be associated with the device using the `[configure()](https://gpuweb.github.io/gpuweb/#dom-gpucanvascontext-configure)` method, like so:

```jsx
const context = canvas.getContext("webgpu");
const canvasFormat = navigator.gpu.getPreferredCanvasFormat();
context.configure({
  device: device,
  format: canvasFormat,
});
```

### Note on Devices, Textures and Formats

There are a few options that can be passed here, but the most important ones are 

- The `device` that you are going to use the context with and,
- The `format`, which is the **texture format** that the context should use

**Textures** are the objects that WebGPU uses to store image data, and **each texture has a format** that lets the GPU know how that data is laid out in memory

Details out of scope but important thing to note is:

- The canvas context provides textures for your code to draw into and,
- The format that you use can have an impact on how efficiently the canvas shows those images

Different types of devices perform best when using different texture formats

- Fortunately, you don't have to worry much about any of that because WebGPU tells you which format to use for your canvas

In almost all cases, you want to pass the value returned by calling  `navigator.gpu.getPreferredCanvasFormat()`, as shown above

**Note:** One big difference in how WebGPU works compared to WebGL is that because canvas configuration is separate from device creation you can have any number of canvases that are all being rendered by a single device

- Makes certain use cases, like multi-pane 3D editors, much easier to develop

## Clear the Canvas

Now that you have a device and the canvas has been configured with it, you can start using the device to change the content of the canvas

We’ll start with a solid color:

In order to do that—or pretty much anything else in WebGPU—you need to provide some commands to the GPU instructing it what to do

- To do this, have the device create a `[GPUCommandEncoder](https://gpuweb.github.io/gpuweb/#gpucommandencoder)`, which provides an interface for recording GPU commands

```jsx
const encoder = device.createCommandEncoder();
```

The commands you want to send to the GPU are related to rendering (in this case, clearing the canvas), so the next step is to use the `encoder` to begin a Render Pass

### Render Passes

Render passes are when all drawing operations in WebGPU happen

Each one starts off with a `[beginRenderPass()](https://gpuweb.github.io/gpuweb/#dom-gpucommandencoder-beginrenderpass)` call, which defines the textures that receive the output of any drawing commands performed

- More advanced uses can provide several textures, called *attachments*, with various purposes such as storing the depth of rendered geometry or providing antialiasing
1. Get the texture from the canvas context you created earlier by calling `[context.getCurrentTexture()](https://gpuweb.github.io/gpuweb/#dom-gpucanvascontext-getcurrenttexture)`, which returns a texture with a pixel width and height matching the canvas's `width` and `height` attributes and the `format` specified when you called `context.configure()`.

```jsx
const pass = encoder.beginRenderPass({
  colorAttachments: [{
     view: context.getCurrentTexture().createView(),
     loadOp: "clear",
     storeOp: "store",
  }]
});
```

The texture is given as the `view` property of a `colorAttachment`

- Render passes require that you provide a `GPUTextureView` instead of a `GPUTexture`, which tells it which parts of the texture to render to
- This only really matters for more advanced use cases, so here you call `createView()` with no arguments on the texture, indicating that you want the render pass to use the entire texture

You also have to specify what you want the render pass to do with the texture when it starts and when it ends:

- A `loadOp` value of `"clear"` indicates that you want the texture to be cleared when the render pass starts.
- A `storeOp` value of `"store"` indicates that once the render pass is finished you want the results of any drawing done during the render pass saved into the texture.

The act of starting the render pass with `loadOp: "clear"` is enough to clear the texture view and the canvas.

End the render pass by adding the following call immediately after `beginRenderPass()`:

```jsx
pass.end();
```

**Note:** simply making these calls does not cause the GPU to actually do anything. They're just recording commands for the GPU to do later

In order to create a `GPUCommandBuffer`, call `finish()` on the command encoder. The command buffer is an opaque handle to the recorded commands

```jsx
const commandBuffer = encoder.finish();
```

Submit the command buffer to the GPU using the `queue` of the `GPUDevice`

- The queue performs all GPU commands, ensuring that their execution is well ordered and properly synchronized

The queue's `submit()` method takes in an array of command buffers, though in this case you only have one

```jsx
device.queue.submit([commandBuffer]);
```

- Once you submit a command buffer, it cannot be used again, so there's no need to hold on to it
- If you want to submit more commands, you need to build another command buffer
- That's why it's fairly common to see those two steps collapsed into one, as is done in the sample pages for this codelab: `device.queue.submit([encoder.finish()]);`

Now, canvas displays the texture as an image (In this case a black square)

If you want to update the canvas contents again after that, you need to record and submit a new command buffer, calling `context.getCurrentTexture()` again to get a new texture for a render pass.

## Pick a Color

1. In the `device.beginRenderPass()` call, add a new line with a `clearValue` to the `colorAttachment`, like this:

```jsx
const pass = encoder.beginRenderPass({
  colorAttachments: [{
    view: context.getCurrentTexture().createView(),
    loadOp: "clear",
    clearValue: { r: 0, g: 0, b: 0.4, a: 1 }, // New line
    storeOp: "store",
  }],
});
```

The `clearValue` instructs the render pass which color it should use when performing the `clear` operation at the beginning of the pass

The dictionary passed into it contains four values: 

- `r` for *red*, `g` for *green*, `b` for *blue*, and `a` for *alpha* (transparency)
- Each value can range from `0` to `1`, and together they describe the value of that color channel

**Tip:** Shorthand: Passing `[0, 0.5, 0.7, 1]` is the same as passing `{ r: 0, g: 0.5, b: 0.7, a: 1 }`
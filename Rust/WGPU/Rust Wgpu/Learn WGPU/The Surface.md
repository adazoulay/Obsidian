# The Surface

# First, State

```rust
use winit::window::Window;

struct State {
    surface: wgpu::Surface,
    device: wgpu::Device,
    queue: wgpu::Queue,
    config: wgpu::SurfaceConfiguration,
    size: winit::dpi::PhysicalSize<u32>,
    window: Window,
}
```

## **`State::new()`**

Now we want to `imp State` 

- We start with `new`:

```rust
async fn new(window: Window) -> Self {
        let size = window.inner_size();

        // The instance is a handle to our GPU
        // Backends::all => Vulkan + Metal + DX12 + Browser WebGPU
        let instance = wgpu::Instance::new(wgpu::InstanceDescriptor {
            backends: wgpu::Backends::all(),
            dx12_shader_compiler: Default::default(),
        });
        
        // # Safety
        //
        // The surface needs to live as long as the window that created it.
        // State owns the window so this should be safe.
        let surface = unsafe { instance.create_surface(&window) }.unwrap();

        let adapter = instance.request_adapter(
            &wgpu::RequestAdapterOptions {
                power_preference: wgpu::PowerPreference::default(),
                compatible_surface: Some(&surface),
                force_fallback_adapter: false,
            },
        ).await.unwrap();
    }
```

### ****Instance and Adapter****

- The `instance` is the first thing you create when using wgpu. Its main purpose is to create `Adapter`s and `Surface`s
- The `adapter` is a handle to our actual graphics card. You can use this to get information about the graphics card such as its name and what backend the adapter uses
    - We use this to create our `Device` and `Queue` later.
    - Let's discuss the fields of `RequestAdapterOptions`
        - `power_preference` has two variants: `LowPower`, and `HighPerformance`
            - `LowPower` will pick an adapter that favors battery life, such as an integrated GPU.
            - `HighPerformance` will pick an adapter for more power-hungry yet more performant GPU's such as a dedicated graphics card
            - WGPU will favor `LowPower` if there is no adapter for the `HighPerformance` option
        - The `compatible_surface` field tells wgpu to find an adapter that can present to the supplied surface
        - The `force_fallback_adapter` forces wgpu to pick an adapter that will work on all hardware
    - **Note:** Selected options will work for most devices. If wgpu can't find an adapter with the required permissions, `request_adapter` will return `None`
        - If you want to get all adapters for a particular backend you can use `enumerate_adapters`
        - Can be looped over to find a compatible one:
        
        ```rust
        let adapter = instance
            .enumerate_adapters(wgpu::Backends::all())
            .find(|adapter| {
                // Check if this adapter supports our surface
                adapter.is_surface_supported(&surface)
            })
            .unwrap()
        ```
        
        - `enumerate_adapters` isn't available on WASM, so you have to use `request_adapter`

### The Surface

The `surface` is the part of the window that we draw to. We need it to draw directly to the screen

Our `window` needs to implement **[raw-window-handle (opens new window)](https://crates.io/crates/raw-window-handle)**'s `HasRawWindowHandle` trait to create a surface. Fortunately, winit's `Window` fits the bill. We also need it to request our `adapter`.

### Device and Queue

We can use `adapter` to create the device and queueu

```rust
let (device, queue) = adapter.request_device(
            &wgpu::DeviceDescriptor {
                features: wgpu::Features::empty(),
                // WebGL doesn't support all of wgpu's features, so if
                // we're building for the web we'll have to disable some.
                limits: if cfg!(target_arch = "wasm32") {
                    wgpu::Limits::downlevel_webgl2_defaults()
                } else {
                    wgpu::Limits::default()
                },
                label: None,
            },
            None, // Trace path
        ).await.unwrap();
```

The `features` field on `DeviceDescriptor`, allows us to specify what extra features we want

- For this simple example, I've decided not to use any extra features.
    
    The graphics card you have limits the features you can use. If you want to use certain features you may need to limit what devices you support or provide workarounds.
    
    You can get a list of features supported by your device using `adapter.features()`, or `device.features()`.
    
    You can view a full list of features **[here (opens new window)](https://docs.rs/wgpu/latest/wgpu/struct.Features.html)**.
    

`limits` field describes the limit of certain types of resources that we can create

- We'll use the defaults for this tutorial, so we can support most devices. You can view a list of limits **[here (opens new window)](https://docs.rs/wgpu/latest/wgpu/struct.Limits.html)**.

```rust
let surface_caps = surface.get_capabilities(&adapter);
        // Shader code in this tutorial assumes an sRGB surface texture. Using a different
        // one will result all the colors coming out darker. If you want to support non
        // sRGB surfaces, you'll need to account for that when drawing to the frame.
        let surface_format = surface_caps.formats.iter()
            .copied()
            .find(|f| f.describe().srgb)            
            .unwrap_or(surface_caps.formats[0]);
        let config = wgpu::SurfaceConfiguration {
            usage: wgpu::TextureUsages::RENDER_ATTACHMENT,
            format: surface_format,
            width: size.width,
            height: size.height,
            present_mode: surface_caps.present_modes[0],
            alpha_mode: surface_caps.alpha_modes[0],
            view_formats: vec![],
        };
        surface.configure(&device, &config);
```

Here we are defining a config for our surface. This will define how the surface creates its underlying `SurfaceTexture`s

- Config fields:
- The `usage` field describes how `SurfaceTexture`s will be used
    - `RENDER_ATTACHMENT` specifies that the textures will be used to write to the screen (we'll talk about more `TextureUsages`s later)
- The `format` defines how `SurfaceTexture`s will be stored on the gpu.
    - We can get a supported format from the `SurfaceCapabilities`
- `width` and `height` are the width and the height in pixels of a `SurfaceTexture`.
    - This should usually be the width and the height of the window.
- `present_mode` uses `wgpu::PresentMode` enum which determines how to sync the surface with the display
    - The option we picked, `PresentMode::Fifo`, will cap the display rate at the display's framerate (essentially VSync)
    - If you want to let your users pick what `PresentMode` they use, you can use **[SurfaceCapabilities::present_modes (opens new window)](https://docs.rs/wgpu/latest/wgpu/struct.SurfaceCapabilities.html#structfield.present_modes)**
- `alpha_mode` has something to do with transparent windows
- `view_formats` is a list of `TextureFormat`s that you can use when creating `TextureView`s
    - writing this means that if your surface is srgb color space, you can create a texture view that uses a linear color space
    - more in depth **[in the texture tutorial](https://sotrh.github.io/learn-wgpu/beginner/tutorial5-textures)**
    

Now that we have configured the surface, we can return the object

```rust
async fn new(window: &Window) -> Self {
        // ...

        Self {
            window,
            surface,
            device,
            queue,
            config,
            size,
        }
    }
```

Since our `State::new()` method is async we need to change `run()` to be async as well so that we can await it.

```rust
pub async fn run() {
    // Window setup...

    let mut state = State::new(window).await;

    // Event loop...
}
```

Now that `run()` is async, `main()` will need some way to await the future

- could use a crate like **[tokio (opens new window)](https://docs.rs/tokio)**, or **[async-std (opens new window)](https://docs.rs/async-std)**, but I'm going to go with the much more lightweight **[pollster (opens new window)](https://docs.rs/pollster)**

Add the following to `Cargo.toml`

```rust
[dependencies]
# other deps...
pollster = "0.2"
```

We then use the `block_on` function provided by pollster to await our future:

```rust
fn main() {
    pollster::block_on(run());
}
```

Don't use `block_on` inside of an async function if you plan to support WASM. Futures have to be run using the browser's executor. If you try to bring your own your code will crash when you encounter a future that doesn't execute immediately.

If we try to build WASM now it will fail because `wasm-bindgen` doesn't support using async functions as `start` methods

- could switch to calling `run` manually in javascript, but for simplicity, we'll add the **[wasm-bindgen-futures (opens new window)](https://docs.rs/wasm-bindgen-futures)**crate to our WASM dependencies as that doesn't require us to change any code

## `resize()`

If we want to support resizing in our application, we're going to need to reconfigure the `surface` every time the window's size changes

- That's the reason we stored the physical `size` and the `config` used to configure the `surface`

With all of these, the resize method is very simple.

```rust
// impl State
pub fn resize(&mut self, new_size: winit::dpi::PhysicalSize<u32>) {
    if new_size.width > 0 && new_size.height > 0 {
        self.size = new_size;
        self.config.width = new_size.width;
        self.config.height = new_size.height;
        self.surface.configure(&self.device, &self.config);
    }
}
```

There's nothing different here from the initial `surface` configuration, so I won't get into it

We call this method in `run()` in the event loop for the following events

```rust
match event {
    // ...

    } if window_id == state.window().id() => if !state.input(event) {
        match event {
            // ...

            WindowEvent::Resized(physical_size) => {
                state.resize(*physical_size);
            }
            WindowEvent::ScaleFactorChanged { new_inner_size, .. } => {
                // new_inner_size is &&mut so we have to dereference it twice
                state.resize(**new_inner_size);
            }
            // ...
        }
    }
}
```

## `input()`

`input()` returns a `bool` to indicate whether an event has been fully processed

If the method returns `true`, the main loop won't process the event any further

We're just going to return false for now because we don't have any events we want to capture

```rust
// impl State
fn input(&mut self, event: &WindowEvent) -> bool {
    false
}
```

We need to do a little more work in the event loop. We want `State` to have priority over `run()`

Doing that (and previous changes) should have your loop looking like this:

```rust
// run()
event_loop.run(move |event, _, control_flow| {
    match event {
        Event::WindowEvent {
            ref event,
            window_id,
        } if window_id == state.window().id() => if !state.input(event) { // UPDATED!
            match event {
                WindowEvent::CloseRequested
                | WindowEvent::KeyboardInput {
                    input:
                        KeyboardInput {
                            state: ElementState::Pressed,
                            virtual_keycode: Some(VirtualKeyCode::Escape),
                            ..
                        },
                    ..
                } => *control_flow = ControlFlow::Exit,
                WindowEvent::Resized(physical_size) => {
                    state.resize(*physical_size);
                }
                WindowEvent::ScaleFactorChanged { new_inner_size, .. } => {
                    state.resize(**new_inner_size);
                }
                _ => {}
            }
        }
        _ => {}
    }
});
```

## `update()`

We don't have anything to update yet, so leave the method empty.

```rust
fn update(&mut self) {
    // remove `todo!()`
}
```

We'll add some code here later on to move around objects

## `render()`

Here's where the magic happens. First, we need to get a frame to render to

```rust
// impl State
fn render(&mut self) -> Result<(), wgpu::SurfaceError> {
    let output = self.surface.get_current_texture()?;
```

The `get_current_texture` function will wait for the `surface` to provide a new `SurfaceTexture` that we will render to

We'll store this in `output` for later

```rust
let view = output.texture.create_view(&wgpu::TextureViewDescriptor::default());
```

This line creates a `TextureView` with default settings

- We need to do this because we want to control how the render code interacts with the texture.

We also need to create a `CommandEncoder` to create the actual commands to send to the gpu

- Most modern graphics frameworks expect commands to be stored in a command buffer before being sent to the gpu
- The `encoder` builds a command buffer that we can then send to the gpu

```rust
let mut encoder = self.device.create_command_encoder(&wgpu::CommandEncoderDescriptor {
        label: Some("Render Encoder"),
    });
```

Now we can get to clearing the screen (long time coming)

- We need to use the `encoder` to create a `RenderPass`
- The `RenderPass` has all the methods for the actual drawing

```rust
{
        let _render_pass = encoder.begin_render_pass(&wgpu::RenderPassDescriptor {
            label: Some("Render Pass"),
            color_attachments: &[Some(wgpu::RenderPassColorAttachment {
                view: &view,
                resolve_target: None,
                ops: wgpu::Operations {
                    load: wgpu::LoadOp::Clear(wgpu::Color {
                        r: 0.1,
                        g: 0.2,
                        b: 0.3,
                        a: 1.0,
                    }),
                    store: true,
                },
            })],
            depth_stencil_attachment: None,
        });
    }

    // submit will accept anything that implements IntoIter
    self.queue.submit(std::iter::once(encoder.finish()));
    output.present();

    Ok(())
}
```

First things first, let's talk about the extra block (`{}`) around `encoder.begin_render_pass(...)`

- `begin_render_pass()` borrows `encoder` mutably (aka `&mut self`)
- can't call `encoder.finish()` until we release that mutable borrow
- The block tells rust to drop any variables within it when the code leaves that scope thus releasing the mutable borrow on `encoder` and allowing us to `finish()` it
    - If you don't like the `{}`, you can also use `drop(render_pass)` to achieve the same effect

The last lines of the code tell `wgpu` to finish the command buffer, and to submit it to the gpu's render queue

We need to update the event loop again to call this method. We'll also call `update()` before it too

### What’s going on with?`RenderPassDescriptor`

```rust
&wgpu::RenderPassDescriptor {
    label: Some("Render Pass"),
    color_attachments: &[
        // ...
    ],
    depth_stencil_attachment: None,
}
```

A `RenderPassDescriptor` only has three fields:

- `label`: Debug info
- `color_attachments`: describe where we are going to draw our color to
    - We use the `TextureView` we created earlier to make sure that we render to the screen
- `depth_stencil_attachment`: We'll use `depth_stencil_attachment` later, but we'll set it to `None` for now

```rust
Some(wgpu::RenderPassColorAttachment {
    view: &view,
    resolve_target: None,
    ops: wgpu::Operations {
        load: wgpu::LoadOp::Clear(wgpu::Color {
            r: 0.1,
            g: 0.2,
            b: 0.3,
            a: 1.0,
        }),
        store: true,
    },
})
```

- The `RenderPassColorAttachment` has the `view` field which informs `wgpu` what texture to save the colors to.
    - In this case we specify the `view` that we created using `surface.get_current_texture()`
    - This means that any colors we draw to this attachment will get drawn to the screen
- The `resolve_target` is the texture that will receive the resolved output
    - This will be the same as `view` unless multisampling is enabled.
    - We don't need to specify this, so we leave it as `None`
- The `ops` field takes a `wpgu::Operations` object
    - This tells wgpu what to do with the colors on the screen (specified by `view`)
    - The `load` field tells wgpu how to handle colors stored from the previous frame
    - Currently, we are clearing the screen with a bluish color
    - The `store` field tells wgpu whether we want to store the rendered results to the `Texture` behind our `TextureView`
        - (in this case it's the `SurfaceTexture`)
        - We use `true` as we do want to store our render results

Now our window should be a light blue
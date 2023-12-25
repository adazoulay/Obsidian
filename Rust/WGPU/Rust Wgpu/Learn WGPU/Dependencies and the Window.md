# Dependencies and the Window

# For Desktop

Just paste this into your `lib.rs`

```rust
use winit::{
    event::*,
    event_loop::{ControlFlow, EventLoop},
    window::WindowBuilder,
};

pub fn run() {
    env_logger::init();
    let event_loop = EventLoop::new();
    let window = WindowBuilder::new().build(&event_loop).unwrap();

    event_loop.run(move |event, _, control_flow| match event {
        Event::WindowEvent {
            ref event,
            window_id,
        } if window_id == window.id() => match event {
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
            _ => {}
        },
        _ => {}
    });
}
```

- Create a window, and keep it open until the user closes it, or presses escape

Then in main:

```rust
use tutorial1_window::run;

fn main() {
    run();
}
```

# Add Support for Web

Make changes to `cargo.toml`

```toml
[lib]
crate-type = ["cdylib", "rlib"]
```

Tells cargo that we want to allow our crate to build a native Rust static library (rlib) and a C/C++ compatible library (cdylib). 

- Need `rlib` if we want to run wgpu in a desktop environment
- Need `cdylib` to create the Web Assembly that the browser will run

Now we need to add more WASM dependencies

```toml
[dependencies]
cfg-if = "1"
# the other regular dependencies...

[target.'cfg(target_arch = "wasm32")'.dependencies]
console_error_panic_hook = "0.1.6"
console_log = "0.2.0"
wgpu = { version = "0.15", features = ["webgl"]}
wasm-bindgen = "0.2"
wasm-bindgen-futures = "0.4.30"
web-sys = { version = "0.3", features = [
    "Document",
    "Window",
    "Element",
]}
```

## More code

First, we need to import `wasm-bindgen` in `lib.rs`

And need to tell wasm-bindgen to run our `run()` function when the WASM is loaded

```rust
#[cfg(target_arch="wasm32")]
use wasm_bindgen::prelude::*;

#[cfg_attr(target_arch="wasm32", wasm_bindgen(start))]
pub fn run() {
    // same as above for now...
}
```

Next, after we create our event loop and window, we need to add a canvas to the HTML document that we will host our application

```rust
#[cfg(target_arch = "wasm32")]
{
    // Winit prevents sizing with CSS, so we have to set
    // the size manually when on web.
    use winit::dpi::PhysicalSize;
    window.set_inner_size(PhysicalSize::new(450, 400));
    
    use winit::platform::web::WindowExtWebSys;
    web_sys::window()
        .and_then(|win| win.document())
        .and_then(|doc| {
            let dst = doc.get_element_by_id("wasm-example")?;  // <-- This doens't seem to work: Try: let dst = doc.body()?;
            let canvas = web_sys::Element::from(window.canvas());
            dst.append_child(&canvas).ok()?;
            Some(())
        })
        .expect("Couldn't append canvas to document body.");
}
```

The `"wasm-example"`  in `get_element_by_id` is specific to my project (aka. this tutorial)

- You can substitute this for whatever id you're using in your HTML
- Alternatively, you could add the canvas directly to the `<body>` as they do in the wgpu repo. This part is ultimately up to you.

All code:

```rust
#[cfg(target_arch = "wasm32")]
use wasm_bindgen::prelude::*;

use winit::{
    event::*,
    event_loop::{ControlFlow, EventLoop},
    window::WindowBuilder,
};

// ! ---- Tutorial 1: The window ----

#[cfg_attr(target_arch = "wasm32", wasm_bindgen(start))]
pub fn run() {
    cfg_if::cfg_if! {
        if #[cfg(target_arch = "wasm32")] {
            std::panic::set_hook(Box::new(console_error_panic_hook::hook));
            console_log::init_with_level(log::Level::Warn).expect("Couldn't initialize logger");
        } else {
            cfg_if::cfg_if! {
                if #[cfg(target_arch = "wasm32")] {
                    std::panic::set_hook(Box::new(console_error_panic_hook::hook));
                    console_log::init_with_level(log::Level::Warn).expect("Couldn't initialize logger");
                } else {
                    env_logger::init();
                }
            }
        }
    }

    let event_loop = EventLoop::new();
    let window = WindowBuilder::new().build(&event_loop).unwrap();

    #[cfg(target_arch = "wasm32")]
    {
        // Winit prevents sizing with CSS, so we have to set
        // the size manually when on web.
        use winit::dpi::PhysicalSize;
        window.set_inner_size(PhysicalSize::new(450, 400));

        use winit::platform::web::WindowExtWebSys;
        web_sys::window()
            .and_then(|win| win.document())
            .and_then(|doc| {
                let dst = doc.get_element_by_id("wasm-example")?;
                let canvas = web_sys::Element::from(window.canvas());
                dst.append_child(&canvas).ok()?;
                Some(())
            })
            .expect("Couldn't append canvas to document body.");
    }

    event_loop.run(move |event, _, control_flow| match event {
        Event::WindowEvent {
            ref event,
            window_id,
        } if window_id == window.id() => match event {
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
            _ => {}
        },
        _ => {}
    });
}
```
# Crates

```toml
[dependencies]
winit = "0.28.5"
env_logger = "0.10"
log = "0.4"
wgpu = "0.16.0"
```

## For WASM

```toml
[package]
name = "totorial1_window"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
cfg-if = "1"
winit = "0.28.5"
env_logger = "0.10"
log = "0.4"
wgpu = "0.16.0"

[target.'cfg(target_arch = "wasm32")'.dependencies]
console_error_panic_hook = "0.1.6"
console_log = "1.0.0"
wgpu = { version = "0.16.0", features = ["webgl"]}
wasm-bindgen = "0.2"
wasm-bindgen-futures = "0.4.30"
web-sys = { version = "0.3", features = [
    "Document",
    "Window",
    "Element",
]}

[lib]
crate-type = ["cdylib", "rlib"]
```

- The `[target.'cfg(target_arch = "wasm32")'.dependencies]` line tells cargo to only include these dependencies if we are targeting the `wasm32` architecture
- Next dependencies make interfacing with javascript a lot easier
    - `console_error_panic_hook`**:** configures the `panic!` macro to send errors to the javascript console
    - `console_log`: ****implements the `log`API.  It sends all logs to the javascript console
    - Need to enable WebGL feature on wgpu if we want to run on most current browsers (untill fully supported)
    - `wasm-bindgen`: The most important dependency in this list.
        - Responsible for generating the boilerplate code that will tell the browser how to use our crate
        - Also allows us to expose methods in Rust that can be used in Javascript, and vice-versa
    - `web-sys` ****is a crate that includes many methods and structures that are available in a normal javascript application: `get_element_by_id`, `append_child`

## To Build

- `wasm-pack build`
- `wasm-pack build --target web`

Then in JS

```jsx
const init = await import('./pkg/name.js');
init().then(() => console.log("WASM Loaded"));
```
# Debugging

## Building with Debug Symbols

When using a "debug" build (aka `wasm-pack build --debug` or `cargo build`) debug symbols are enabled by default

With a "release" build, debug symbols are not enabled by default

If no debug symbols:

- Stack traces will have function names like `wasm-function[42]` rather than the Rust name of the function, like `wasm_game_of_life::Universe::live_neighbor_count`

## Logging with the `console` APIs

On the Web, the `console.log` function is the way to log messages to the browser's developer tools console

We can use the `web-sys` crate to get access to the `console` logging functions:

```rust
extern crate web_sys;
web_sys::console::log_1(&"Hello, world!".into());
```

### **References**

- Using `console.log` with the `web-sys` crate:
    - `[web_sys::console::log` takes an array of values to log](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log.html)
    - `[web_sys::console::log_1` logs a single value](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_1.html)
    - `[web_sys::console::log_2` logs two values](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_2.html)
    - Etc...
- Using `console.error` with the `web-sys` crate:
    - `[web_sys::console::error` takes an array of values to log](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error.html)
    - `[web_sys::console::error_1` logs a single value](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_1.html)
    - `[web_sys::console::error_2` logs two values](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_2.html)
    - Etc...

### In Game of Life Example:

**Filename:** Cargo.toml

```toml
[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

For ergonomics, we'll wrap the `console.log` function up in a `println!`-style macro:

```rust
extern crate web_sys;

// A macro to provide `println!(..)`-style syntax for `console.log` logging.
macro_rules! log {
    ( $( $t:tt )* ) => {
        web_sys::console::log_1(&format!( $( $t )* ).into());
    }
}
```

## Enable Logging for Panics

If our code panics, we want informative error messages to appear in the developer console

`console_error_panic_hook`: Logs unexpected panics to the developer console via `console.error`

Rather than getting cryptic, difficult-to-debug `RuntimeError: unreachable executed` error messages, this gives you Rust's formatted panic message.

- Enabled-by-default dependency on the `console_error_panic_hook` crate that is configured in `wasm-game-of-life/src/utils.rs`
- All you need to do is install the hook by calling `console_error_panic_hook::set_once()` in an initialization function or common code path:

**Filename:** utils.rs

```rust
pub fn set_panic_hook() {
    // When the `console_error_panic_hook` feature is enabled, we can call the
    // `set_panic_hook` function at least once during initialization, and then
    // we will get better error messages if our code ever panics.
    #[cfg(feature = "console_error_panic_hook")]
    console_error_panic_hook::set_once();
}
```

- We can for example call it inside the `Universe::new` constructor in `wasm-game-of-life/src/lib.rs`

## Using a Debugger to Pause Between Each Tick

Browser's stepping debuggers are useful for inspecting the JavaScript that our Rust-generated WebAssembly interacts with.

For example, we can use the debugger to pause on each iteration of our `renderLoop` function by placing [a JavaScript `debugger;` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger) above our call to `universe.tick()`.

```jsx
const renderLoop = () => {
  debugger;
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};

```

This provides us with a convenient checkpoint for inspecting logged messages, and comparing the currently rendered frame to the previous one.
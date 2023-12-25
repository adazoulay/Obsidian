# Useful Crates and Resources

- **wasm-bindgen**: This library provides a way to define imports and exports between Rust and JavaScript. It consists of two components:
    - Rust macros that generate exports, imports, and glue code on the Rust side.
    - A command-line tool for generating a JavaScript wrapper that allows loading the WebAssembly as an ES module. wasm-bindgen simplifies the process of creating JavaScript bindings for your Rust code and makes it easier to interact with JavaScript APIs from Rust. [novatec-gmbh.de](https://www.novatec-gmbh.de/en/blog/look-ma-no-js-compiling-rust-to-webassembly/)
- **js-sys**: This library provides pre-defined bindings for global JavaScript APIs that are guaranteed to be present in all JavaScript runtimes, such as browsers, Node.js, and other environments. It eliminates the need to create custom bindings for common JavaScript functions, making it easier to work with JavaScript APIs from Rust. [novatec-gmbh.de](https://www.novatec-gmbh.de/en/blog/look-ma-no-js-compiling-rust-to-webassembly/)
- **web-sys**: This library provides pre-defined bindings for the Web API, which includes access to the DOM, console.log(), and other web-related functions. It simplifies the process of interacting with web-specific APIs from Rust, eliminating the need to create custom bindings for these functions. [novatec-gmbh.de](https://www.novatec-gmbh.de/en/blog/look-ma-no-js-compiling-rust-to-webassembly/)

### Templates:

[https://rustwasm.github.io/docs/book/reference/project-templates.html](https://rustwasm.github.io/docs/book/reference/project-templates.html)

### Repos

[https://github.com/rwasm/vite-plugin-rsw](https://github.com/rwasm/vite-plugin-rsw)

### Books

[https://rustwasm.github.io/wasm-bindgen/examples/char.html](https://rustwasm.github.io/wasm-bindgen/examples/char.html)
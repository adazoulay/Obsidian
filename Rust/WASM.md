# WASM

[https://rustwasm.github.io/docs/book/game-of-life/introduction.html](https://rustwasm.github.io/docs/book/game-of-life/introduction.html)

[[Setup and wasm-pack]]

[[Iterfacing Rust and JS]]

[[Rendering to Canvas Directly from Memory]]

[[Adding Interactivity]]

[[Time Profiling]]

[[Debugging]]

[[Shrinking .wasm size]]

[[Useful Crates and Resources]]

- With React:
    - [https://ppuzio.medium.com/c-in-the-browser-with-webassembly-via-emscripten-vite-and-react-bd82e0598a5e](https://ppuzio.medium.com/c-in-the-browser-with-webassembly-via-emscripten-vite-and-react-bd82e0598a5e)
- Wasm-bindgen book
    - [https://rustwasm.github.io/docs/wasm-bindgen/introduction.html](https://rustwasm.github.io/docs/wasm-bindgen/introduction.html)
- Wasm-pack book
    - [https://rustwasm.github.io/docs/wasm-pack/](https://rustwasm.github.io/docs/wasm-pack/)

### **To start a wasm porject:**

- `Cargo new name`
- In project root: `npm create vite@latest`
    - Can name project **www**
- Build project using: `wasm-pack build --target web`
    - Creates ****pkg**** directory
    - Unstable flags: `RUSTFLAGS=--cfg=web_sys_unstable_apis wasm-pack build --target web`
- in ******www:******
    - Install `vite-plugin-wasm` and `vite-plugin-top-level-await`
    - Make **vite.conf.js** file
    
    ```jsx
    import { defineConfig } from "vite";
    import wasm from "vite-plugin-wasm";
    import topLevelAwait from "vite-plugin-top-level-await";
    
    export default defineConfig({
      plugins: [wasm(), topLevelAwait()],
      server: {
        fs: {
          allow: [".."],
        },
      },
    });
    ```
    
    - Add generated wasm as dependency in **package.json** linking to ******pkg****** directory
    
    ```
    "dependencies": {
        "learn_wgpu": "file:../pkg",
    		...
    }
    ```
    
    - Then in **index.js** or **main.js**
    
    ```
    import init from "learn_wgpu";
    
    init().then((instance) => {
      console.log("it worked");
      instance.exports.test();
    });
    ```
    
    - In ******index.html******
    
    ```jsx
    <script type="module" src="/index.js"></script>
    ```
    
    - To run: `npm run dev`
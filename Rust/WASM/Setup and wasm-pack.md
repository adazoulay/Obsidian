# Setup and wasm-pack

# `wasm-pack`

`wasm-pack` is your one-stop shop for building, testing, and publishing Rust-generated WebAssembly.

- `wasm-pack build`
    - Builds project in pkg directory
    
    ```
    pkg/
    ├── package.json
    ├── [[]]
    ├── wasm_game_of_life_bg.wasm
    ├── wasm_game_of_life.d.ts
    └── wasm_game_of_life.js
    ```
    
- `wasm_game_of_life_bg.wasm`
    - `.wasm` file: WebAssembly binary that is generated by the Rust compiler from our Rust sources
    - Contains the compiled-to-wasm versions of all of our Rust functions and data
- `wasm_game_of_life.js`
    - Generated by `wasm-bindgen` and contains JavaScript glue for importing DOM and JavaScript functions into Rust
    - Exposes a nice API to the WebAssembly functions to JavaScript
- `wasm_game_of_life.d.ts`
    - Contains TypeScript type declarations for the JavaScript glue
    - If you are using TypeScript, you can have your calls into WebAssembly functions type checked, and your IDE can provide autocompletions and suggestions
- `package.json`
    - Contains metadata about the generated JavaScript and WebAssembly package
    - Used by npm and JavaScript bundlers to determine dependencies across packages, package names, versions, and a bunch of other stuff
    - Helps us integrate with JavaScript tooling and allows us to publish our package to npm

# Putting it into a Web Page

Run this command within the `wasm-game-of-life` directory:

- `npm init wasm-app www`
    - Builds more in `wasm-game-of-life/www` subdirectory

```
wasm-game-of-life/www/
├── bootstrap.js
├── index.html
├── index.js
├── LICENSE-APACHE
├── LICENSE-MIT
├── package.json
├── [[]]
└── webpack.config.js
```

- `package.json`
    - comes pre-configured with `webpack` and `webpack-dev-server` dependencies, as well as a dependency on `hello-wasm-pack`, which is a version of the initial `wasm-pack-template` package that has been published to npm
- `webpack.config.js`
    - Configures webpack and its local development server
    - Comes pre-configured, and you shouldn't have to tweak this at all to get webpack and its local development server working
- `index.html`
    - Root HTML file for the Web page
    - Doesn't do much other than load `bootstrap.js`, which is a very thin wrapper around `index.js`
- `index.js`
    - Main entry point for our Web page's JavaScript
    - Imports the `hello-wasm-pack` npm package, which contains the default `wasm-pack-template`'s compiled WebAssembly and JavaScript glue
    - Then it calls `hello-wasm-pack`'s `greet` function

## Install Dependencies

Run `npm install` within the `wasm-game-of-life/www` subdirectory

### Using our Local `wasm-game-of-life` Package in `www`

Rather than use the `hello-wasm-pack` package from npm, we want to use our local `wasm-game-of-life` package instead

- This will allow us to incrementally develop our Game of Life program

Open up `wasm-game-of-life/www/package.json` and next to `"devDependencies"`, add the `"dependencies"` field, including a `"wasm-game-of-life": "file:../pkg"` entry:

```json
{
  // ...
  "dependencies": {                     // Add this three lines block!
    "wasm-game-of-life": "file:../pkg"
  },
  "devDependencies": {
    //...
  }
}
```

Next, modify `wasm-game-of-life/www/index.js` to import `wasm-game-of-life` instead of the `hello-wasm-pack` package:

```jsx
import * as wasm from "wasm-game-of-life";

wasm.greet();
```

Since we declared a new dependency, we need to install it: `npm install`

## Serving Locally

Run `npm run start` from within the `wasm-game-of-life/www` directory
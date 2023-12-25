# Rendering to Canvas Directly from Memory

Generating (and allocating) a `String` in Rust and then having `wasm-bindgen` convert it to a valid JavaScript string makes unnecessary copies of the universe's cells

- JavaScript code already knows the width and height of the universe, and can read WebAssembly's linear memory that make up the cells directly
- We'll modify the `render` method to return a pointer to the start of the cells array

instead of rendering Unicode text, we'll switch to using the [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)

Inside `wasm-game-of-life/www/index.html`, let's replace the `<pre>` we added earlier with a `<canvas>` we will render into 

- (it too should be within the `<body>`, before the `<script>` that loads our JavaScript):

```html
<body>
  <canvas id="game-of-life-canvas"></canvas>
  <script src='./bootstrap.js'></script>
</body>
```

To get the necessary information from the Rust implementation, we'll need to add some more getter functions for a universe's width, height, and pointer to its cells array

```rust
#![allow(unused_variables)]
fn main() {
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn width(&self) -> u32 {
        self.width
    }

    pub fn height(&self) -> u32 {
        self.height
    }

    pub fn cells(&self) -> *const Cell {
        self.cells.as_ptr()
    }
}
}
```

Next, in `wasm-game-of-life/www/index.js`, let's also import `Cell` from `wasm-game-of-life`, and define some constants that we will use when rendering to the canvas:

```jsx
import { Universe, Cell } from "wasm-game-of-life";

const CELL_SIZE = 5; // px
const GRID_COLOR = "#CCCCCC";
const DEAD_COLOR = "#FFFFFF";
const ALIVE_COLOR = "#000000";
```

Now, let's rewrite the rest of this JavaScript code to no longer write to the `<pre>`'s `textContent` but instead draw to the `<canvas>`:

```jsx
// Construct the universe, and get its width and height.
const universe = Universe.new();
const width = universe.width();
const height = universe.height();

// Give the canvas room for all of our cells and a 1px border
// around each of them.
const canvas = document.getElementById("game-of-life-canvas");
canvas.height = (CELL_SIZE + 1) * height + 1;
canvas.width = (CELL_SIZE + 1) * width + 1;

const ctx = canvas.getContext('2d');

const renderLoop = () => {
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```

### Drawing to Canvas

To draw the grid between cells, we draw a set of equally-spaced horizontal lines, and a set of equally-spaced vertical lines:

```jsx
const drawGrid = () => {
  ctx.beginPath();
  ctx.strokeStyle = GRID_COLOR;

  // Vertical lines.
  for (let i = 0; i <= width; i++) {
    ctx.moveTo(i * (CELL_SIZE + 1) + 1, 0);
    ctx.lineTo(i * (CELL_SIZE + 1) + 1, (CELL_SIZE + 1) * height + 1);
  }

  // Horizontal lines.
  for (let j = 0; j <= height; j++) {
    ctx.moveTo(0,                           j * (CELL_SIZE + 1) + 1);
    ctx.lineTo((CELL_SIZE + 1) * width + 1, j * (CELL_SIZE + 1) + 1);
  }

  ctx.stroke();
};
```

### Accessing WebAssembly's linear memory

We can directly access WebAssembly's linear memory via `memory`, which is defined in the raw wasm module `wasm_game_of_life_bg`

- To draw the cells, we get a pointer to the universe's cells, construct a `Uint8Array` overlaying the cells buffer, iterate over each cell, and draw a white or black rectangle depending on whether the cell is dead or alive, respectively
- By working with pointers and overlays, we avoid copying the cells across the boundary on every tick

```jsx
// Import the WebAssembly memory at the top of the file.
import { memory } from "wasm-game-of-life/wasm_game_of_life_bg";

// ...

const getIndex = (row, column) => {
  return row * width + column;
};

const drawCells = () => {
  const cellsPtr = universe.cells();
  const cells = new Uint8Array(memory.buffer, cellsPtr, width * height);

  ctx.beginPath();

  for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
      const idx = getIndex(row, col);

      ctx.fillStyle = cells[idx] === Cell.Dead
        ? DEAD_COLOR
        : ALIVE_COLOR;

      ctx.fillRect(
        col * (CELL_SIZE + 1) + 1,
        row * (CELL_SIZE + 1) + 1,
        CELL_SIZE,
        CELL_SIZE
      );
    }
  }

  ctx.stroke();
};
```
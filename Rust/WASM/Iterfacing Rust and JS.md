# Iterfacing Rust and JS

JavaScript's garbage-collected heap — where `Object`s, `Array`s, and DOM nodes are allocated — is distinct from WebAssembly's linear memory space, where our Rust values live

- WebAssembly currently has no direct access to the garbage-collected heap
- JavaScript, on the other hand, can read and write to the WebAssembly linear memory space
    - But only as an `[ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)` of scalar values (`u8`, `i32`, `f64`, etc...)
    - WebAssembly functions also take and return scalar values

`wasm_bindgen` defines a common understanding of how to work with compound structures across this boundary

- Involves boxing Rust structures, and wrapping the pointer in a JavaScript class for usability, or indexing into a table of JavaScript objects from Rust
- think of `wasm_bindgen` as a tool for implementing the interface design you choose

### Design Choices

When designing an interface between WebAssembly and JavaScript, we want to optimize for the following properties:

1. **Minimizing copying into and out of the WebAssembly linear memory.** Unnecessary copies impose unnecessary overhead.
2. **Minimizing serializing and deserializing.** Similar to copies, serializing and deserializing also imposes overhead, and often imposes copying as well. If we can pass opaque handles to a data structure — instead of serializing it on one side, copying it into some known location in the WebAssembly linear memory, and deserializing on the other side — we can often reduce a lot of overhead. `wasm_bindgen` helps us define and work with opaque handles to JavaScript `Object`s or boxed Rust structures.

As a general rule of thumb, a good JavaScript↔WebAssembly interface design is often one where:

- Large, long-lived data structures are implemented as Rust types that live in the WebAssembly linear memory, and are exposed to JavaScript as opaque handles
- JavaScript calls exported WebAssembly functions that take these opaque handles, transform their data, perform heavy computations, query the data, and ultimately return a small, copy-able result
    - By only returning the small result of the computation, we avoid copying and/or serializing everything back and forth between the JavaScript garbage-collected heap and the WebAssembly linear memory

# Game of Life Example:

We don't want to:

- Copy the whole universe into and out of the WebAssembly linear memory on every tick
- Allocate objects for every cell in the universe, nor do we want to impose a cross-boundary call to read and write each cell

We can represent the universe as a flat array that lives in the WebAssembly linear memory, and has a byte for each cell. `0` is a dead cell and `1` is a live cell

************Example:************ What a 4x4 universe looks like in memory

![Untitled](Iterfacing%20Rust%20and%20JS/Untitled.png)

To find the array index of the cell at a given row and column in the universe, we can use this formula:

```rust
index(row, column, universe) = row * width(universe) + column
```

### Game of Life Implementation Rust:

```rust
#[wasm_bindgen]
#[repr(u8)]
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum Cell {
    Dead = 0,
    Alive = 1,
}
```

- `#[repr(u8)]`: Important so that each cell is represented as a single byte
- `Dead` variant is `0` and that the `Alive` variant is `1`, so that we can easily count a cell's live neighbors with addition

```rust
#[wasm_bindgen]
pub struct Universe {
    width: u32,
    height: u32,
    cells: Vec<Cell>,
}

impl Universe {
    fn get_index(&self, row: u32, column: u32) -> usize {
        (row * self.width + column) as usize
    }

		fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
		        let mut count = 0;
		        for delta_row in [self.height - 1, 0, 1].iter().cloned() {
		            for delta_col in [self.width - 1, 0, 1].iter().cloned() {
		                if delta_row == 0 && delta_col == 0 {
		                    continue;
		                }
		
		                let neighbor_row = (row + delta_row) % self.height;
		                let neighbor_col = (column + delta_col) % self.width;
		                let idx = self.get_index(neighbor_row, neighbor_col);
		                count += self.cells[idx] as u8;
		            }
		        }
		        count
		    }
		}
}
```

- The universe has a width and a height, and a vector of cells of length `width * height`.
- `get_index`: To access the cell at a given row and column, we translate the row and column into an index into the cells vector
- The `live_neighbor_count` method uses deltas and modulo to avoid special casing the edges of the universe with `if`s
    - When applying a delta of `-1`, we *add* `self.height - 1` and let the modulo do its thing, rather than attempting to subtract `1`
    - `row` and `column` can be `0`, and if we attempted to subtract `1` from them, there would be an unsigned integer underflow

Now we have everything we need to compute the next generation from the current one

Each of the Game's rules follows a straightforward translation into a condition on a `match` expression

We use `#[wasm_bindgen]` to expose JS to our WASM api

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    pub fn tick(&mut self) {
        let mut next = self.cells.clone();

        for row in 0..self.height {
            for col in 0..self.width {
                let idx = self.get_index(row, col);
                let cell = self.cells[idx];
                let live_neighbors = self.live_neighbor_count(row, col);

                let next_cell = match (cell, live_neighbors) {
                    // Rule 1: Any live cell with fewer than two live neighbours
                    // dies, as if caused by underpopulation.
                    (Cell::Alive, x) if x < 2 => Cell::Dead,
                    // Rule 2: Any live cell with two or three live neighbours
                    // lives on to the next generation.
                    (Cell::Alive, 2) | (Cell::Alive, 3) => Cell::Alive,
                    // Rule 3: Any live cell with more than three live
                    // neighbours dies, as if by overpopulation.
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // Rule 4: Any dead cell with exactly three live neighbours
                    // becomes a live cell, as if by reproduction.
                    (Cell::Dead, 3) => Cell::Alive,
                    // All other cells remain in the same state.
                    (otherwise, _) => otherwise,
                };

                next[idx] = next_cell;
            }
        }
        self.cells = next;
    }
    // ...
}
```

To make this human readable, let's implement a basic text renderer:

- The idea is to write the universe line by line as text:
    - For each cell that is alive, print the Unicode character `◼` ("black medium square")
    - For dead cells, we'll print `◻` (a "white medium square").
- Implementing the `[Display](https://doc.rust-lang.org/1.25.0/std/fmt/trait.Display.html)` trait from Rust's standard library: We can add a way to format a structure in a user-facing manner
    - Also automatically give us a `[to_string](https://doc.rust-lang.org/1.25.0/std/string/trait.ToString.html)` method

```rust
use std::fmt;

impl fmt::Display for Universe {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        for line in self.cells.as_slice().chunks(self.width as usize) {
            for &cell in line {
                let symbol = if cell == Cell::Dead { '◻' } else { '◼' };
                write!(f, "{}", symbol)?;
            }
            write!(f, "\n")?;
        }

        Ok(())
    }
}
```

Finally, we define a constructor that initializes the universe with an interesting pattern of live and dead cells, as well as a `render` method:

### Rendering with JavaScript

First, add a `<pre>` element to `wasm-game-of-life/www/index.html` to render the universe into, just above the `<script>` tag:

```html
<body>
  <pre id="game-of-life-canvas"></pre>
  <script src="./bootstrap.js"></script>
</body>
```

At the top of `wasm-game-of-life/www/index.js`, let's fix our import to bring in the `Universe` rather than the old `greet` function:

Get that `<pre>` element we just added and instantiate a new universe

```jsx
import { Universe } from "wasm-game-of-life";
const pre = document.getElementById("game-of-life-canvas");
const universe = Universe.new();
```

The JavaScript runs in a `requestAnimationFrame` loop

- On each iteration, it draws the current universe to the `<pre>`, and then calls `Universe::tick`
- To start the rendering process, all we have to do is make the initial call for the first iteration of the rendering loop:

```jsx
const renderLoop = () => {
  pre.textContent = universe.render();
  universe.tick();

  requestAnimationFrame(renderLoop);
};

requestAnimationFrame(renderLoop);
```
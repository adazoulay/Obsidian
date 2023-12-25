# Time Profiling

# Available Tools

## The `window.performance.now()` Timer

The `performance.now()` function returns a monotonic timestamp measured in milliseconds since the Web page was loaded

- Calling `performance.now` has little overhead, so we can create simple, granular measurements from it without distorting the performance of the rest of the system and inflicting bias upon our measurements

We can use it to time various operations, and we can access `window.performance.now()` via the `web-sys` crate

```rust
extern crate web_sys;

fn now() -> f64 {
    web_sys::window()
        .expect("should have a Window")
        .performance()
        .expect("should have a Performance")
        .now()
}
```

- [The `web_sys::window` function](https://rustwasm.github.io/wasm-bindgen/api/web_sys/fn.window.html)
- [The `web_sys::Window::performance` method](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Window.html#method.performance)
- [The `web_sys::Performance::now` method](https://rustwasm.github.io/wasm-bindgen/api/web_sys/struct.Performance.html#method.now)

## Developer Tools Profilers

All Web browsers' built-in developer tools include a profiler which display which functions are taking the most time with the usual kinds of visualizations like call trees and flame graphs

**Note:** If you build with debug symbols so that the "name" custom section is included in the wasm binary, then these profilers should display the Rust function names instead of something opaque like `wasm-function[123]`

- Note that these profilers *won't* show inlined functions, and since Rust and LLVM rely on inlining so heavily, the results might still end up a bit perplexing

## The `console.time` and `console.timeEnd` Functions

The `console.time` and `console.timeEnd` functions allow you to log the timing of named operations to the browser's developer tools console

- You call `console.time("some operation")` when the operation begins, and call `console.timeEnd("some operation")` when it finishes
- The string label naming the operation is optional

`console.time` and `console.timeEnd` logs will **also** show up in your browser's profiler's "timeline" or "waterfall" view

You can use these functions directly via the `web-sys` crate:

- `[web_sys::console::time_with_label("some operation")](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.time_with_label.html)`
- `[web_sys::console::time_end_with_label("some operation")](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.time_end_with_label.html)`

## Using ****`#[bench]` with Native Code**

The same way we can often leverage our operating system's native code debugging tools by writing `#[test]`s rather than debugging on the Web, we can leverage our operating system's native code profiling tools by writing `#[bench]` functions

Write your benchmarks in the `benches` subdirectory of your crate. Make sure that your `crate-type` includes `"rlib"` or else the bench binaries won't be able to link your main lib.

However! Make sure that you know the bottleneck is in the WebAssembly before investing much energy in native code profiling! Use your browser's profiler to confirm this, or else you risk wasting your time optimizing code that isn't hot.

# In Our Game of Life Example

## Creating a Frames Per Second Timer with the `window.performance.now` Function

This FPS timer will be useful as we investigate speeding up our Game of Life's rendering

We start by adding an `fps` object to `wasm-game-of-life/www/index.js`:

```jsx
const fps = new class {
  constructor() {
    this.fps = document.getElementById("fps");
    this.frames = [];
    this.lastFrameTimeStamp = performance.now();
  }

  render() {
    // Convert the delta time since the last frame render into a measure
    // of frames per second.
    const now = performance.now();
    const delta = now - this.lastFrameTimeStamp;
    this.lastFrameTimeStamp = now;
    const fps = 1 / delta * 1000;

    // Save only the latest 100 timings.
    this.frames.push(fps);
    if (this.frames.length > 100) {
      this.frames.shift();
    }

    // Find the max, min, and mean of our 100 latest timings.
    let min = Infinity;
    let max = -Infinity;
    let sum = 0;
    for (let i = 0; i < this.frames.length; i++) {
      sum += this.frames[i];
      min = Math.min(this.frames[i], min);
      max = Math.max(this.frames[i], max);
    }
    let mean = sum / this.frames.length;

    // Render the statistics.
    this.fps.textContent = `
Frames per Second:
         latest = ${Math.round(fps)}
avg of last 100 = ${Math.round(mean)}
min of last 100 = ${Math.round(min)}
max of last 100 = ${Math.round(max)}
`.trim();
  }
};
```

Next we call the `fps` `render` function on each iteration of `renderLoop`:

```jsx
const renderLoop = () => {
    fps.render(); //new

    universe.tick();
    drawGrid();
    drawCells();

    animationId = requestAnimationFrame(renderLoop);
};
```

And add the `fps` element to `wasm-game-of-life/www/index.html`, just above the `<canvas>`:

```html
<div id="fps"></div>
... With CS
#fps {
  white-space: pre;
  font-family: monospace;
}
```

## Time Each `Universe::tick` with `console.time` and `console.timeEnd`

To measure how long each invocation of `Universe::tick` takes, we can use `console.time` and `console.timeEnd` via the `web-sys` crate.

First, add `web-sys` as a dependency to `wasm-game-of-life/Cargo.toml`:

```toml

[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

Because there should be a corresponding `console.timeEnd` invocation for every `console.time` call, it is convenient to wrap them both up in an [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization) type:

```rust
extern crate web_sys;
use web_sys::console;

pub struct Timer<'a> {
    name: &'a str,
}

impl<'a> Timer<'a> {
    pub fn new(name: &'a str) -> Timer<'a> {
        console::time_with_label(name);
        Timer { name }
    }
}

impl<'a> Drop for Timer<'a> {
    fn drop(&mut self) {
        console::time_end_with_label(self.name);
    }
}
```

Then, we can time how long each `Universe::tick` takes by adding this snippet to the top of the method:

```rust
let _timer = Timer::new("Universe::tick");
```

In our case, at the top of `Universe::tick()`

**Note:** `console.time` and `console.timeEnd` pairs will show up in your browser's profiler's "timeline" or "waterfall" view

### Growing our Game of Life Universe

If we change the board size from 64x64 to 128x128 in `Universe::new()` we drop from a smooth 60 to a choppy 40-ish on my machine

- That's not just our JavaScript and WebAssembly, but also everything else the browser is doing, such as painting

If we look at what happens within a single animation frame, we see that the `CanvasRenderingContext2D.fillStyle` setter is very expensive!

**!!!** We might have expected something in the `tick` method to be the performance bottleneck, but it wasn't. Always let profiling guide your focus, since time may be spent in places you don't expect it to be.

In the `drawCells` function in `wasm-game-of-life/www/index.js`, the `fillStyle` property is set once for every cell in the universe, on every animation frame

```jsx
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);

    ctx.fillStyle = cells[idx] === DEAD
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
```

Now that we have discovered that setting `fillStyle` is so expensive, what can we do to avoid setting it so often?

We need to change `fillStyle` depending on whether a cell is alive or dead

- If we set `fillStyle = ALIVE_COLOR` and then draw every alive cell in one pass, and then set `fillStyle = DEAD_COLOR` and draw every dead cell in another pass, then we only end setting `fillStyle` twice, rather than once for every cell

```jsx
// Alive cells.
ctx.fillStyle = ALIVE_COLOR;
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);
    if (cells[idx] !== Cell.Alive) {
      continue;
    }

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}

// Dead cells.
ctx.fillStyle = DEAD_COLOR;
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);
    if (cells[idx] !== Cell.Dead) {
      continue;
    }

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}
```

******Explenation:****** The reason iterating over the cells twice gives better performance is not due to the time complexity of the algorithm but rather due to the reduced number of state changes in the rendering context. In the original version, the `fillStyle` property is set for every cell, causing a context switch in the rendering API. This switch is expensive and affects the performance significantly.

After saving these changes and refreshing [http://localhost:8080/](http://localhost:8080/), rendering is back to a smooth 60 frames per second.

If we take another profile, we can see that only about ten milliseconds are spent in each animation frame now.

## Making Time Run Faster

We can modify the `renderLoop` function in `wasm-game-of-life/www/index.js` to do this quite easily:

```jsx
for (let i = 0; i < 9; i++) {
  universe.tick();
}
```

On my machine, this brings us back down to only 35 frames per second

Now we know that time is being spent in `Universe::tick`, so let's add some `Timer`s to wrap various bits of it in `console.time` and `console.timeEnd` calls, and see where that leads us

**Hypothesis:** Allocating a new vector of cells and freeing the old vector on every tick is costly

**Note:** We are using scope `{}` to measure ticks since we implemented `Drop` on `Timer`

```rust
pub fn tick(&mut self) {
    let _timer = Timer::new("Universe::tick");

    let mut next = {
        let _timer = Timer::new("allocate next cells");
        self.cells.clone()
    };

    {
        let _timer = Timer::new("new generation");
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
    }

    let _timer = Timer::new("free old cells");
    self.cells = next;
}
```

Profiling output:

```
allocate new cells: 0.01416015625 ms
new generation: 1.103759765625 ms
Universe::tick: 1.2099609375 ms
```

Hypothesis is incorrect: The vast majority of time is spent actually calculating the next generation of cells

### Next Section Requires `nightly` compiler

Required for:

- [test feature gate](https://doc.rust-lang.org/unstable-book/library-features/test.html):  Used for the benchmarks
- [cargo benchcmp](https://github.com/BurntSushi/cargo-benchcmp): A small utility for comparing micro-benchmarks produced by `cargo bench`

Let's write a native code `#[bench]` doing the same thing that our WebAssembly is doing, but where we can use more mature profiling tools. Here is the new `wasm-game-of-life/benches/bench.rs`:

```rust
#![feature(test)]

fn main() {
extern crate test;
extern crate wasm_game_of_life;

#[bench]
fn universe_ticks(b: &mut test::Bencher) {
    let mut universe = wasm_game_of_life::Universe::new();

    b.iter(|| {
        universe.tick();
    });
}
```

We also have to comment out all the `#[wasm_bindgen]` annotations, and the `"cdylib"` bits from `Cargo.toml` or else building native code will fail and have link errors.

With all that in place, we can run `cargo bench | tee before.txt` to compile and run our benchmark!

- The `| tee before.txt` part will take the output from `cargo bench` and put in a file called `before.txt`

Then run profiler of choice, in this case `perf`

- `perf record -g target/release/deps/bench-8474091a05cfa2d9 --bench`

Loading up the profile with `perf report` shows that:

- 26.67% of time is being spent summing neighboring cells' values
- 23.41% of time is spent getting the neighbor's column index
- 15.42% of time is spent getting the neighbor's row index

the second and third are both costly `div` instructions. These `div`s implement the modulo indexing logic in `Universe::live_neighbor_count`

Recall the `live_neighbor_count` definition inside `wasm-game-of-life/src/lib.rs`

```rust
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
```

The reason we used modulo was to avoid cluttering up the code with `if` branches for the first or last row or column edge cases

- But we are paying the cost of a `div` instruction even for the most common case, when neither `row` nor `column` is on the edge of the universe and they don't need the modulo wrapping treatment

Instead, if we use `if`s for the edge cases and unroll this loop, the branches *should* be very well-predicted by the CPU's branch predictor

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;

    let north = if row == 0 {
        self.height - 1
    } else {
        row - 1
    };

    let south = if row == self.height - 1 {
        0
    } else {
        row + 1
    };

    let west = if column == 0 {
        self.width - 1
    } else {
        column - 1
    };

    let east = if column == self.width - 1 {
        0
    } else {
        column + 1
    };

    let nw = self.get_index(north, west);
    count += self.cells[nw] as u8;

    let n = self.get_index(north, column);
    count += self.cells[n] as u8;

    let ne = self.get_index(north, east);
    count += self.cells[ne] as u8;

    let w = self.get_index(row, west);
    count += self.cells[w] as u8;

    let e = self.get_index(row, east);
    count += self.cells[e] as u8;

    let sw = self.get_index(south, west);
    count += self.cells[sw] as u8;

    let s = self.get_index(south, column);
    count += self.cells[s] as u8;

    let se = self.get_index(south, east);
    count += self.cells[se] as u8;

    count
}
```

Wow! 7.61x speed up!
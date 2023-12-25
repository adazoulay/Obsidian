# Adding Interactivity

Let's add a button to toggle whether the game is playing or paused

To `wasm-game-of-life/www/index.html`, add the button right above the `<canvas>`

```html
<button id="play-pause"></button>
```

In the `wasm-game-of-life/www/index.js` JavaScript, we will make the following changes:

- Keep track of the identifier returned by the latest call to `requestAnimationFrame`, so that we can cancel the animation by calling `cancelAnimationFrame` with that identifier
- • When the play/pause button is clicked, check for whether we have the identifier for a queued animation frame.
    - If we do, then the game is currently playing, and we want to cancel the animation frame so that `renderLoop` isn't called again, effectively pausing the game
    - If we do not have an identifier for a queued animation frame, then we are currently paused, and we would like to call `requestAnimationFrame` to resume the game

Because the JavaScript is driving the Rust and WebAssembly, this is all we need to do, and we don't need to change the Rust sources.

- We introduce the `animationId` variable to keep track of the identifier returned by `requestAnimationFrame`
    - When there is no queued animation frame, we set this variable to `null`
- At any instant in time, we can tell whether the game is paused or not by inspecting the value of `animationId` through `isPaus`

```jsx
let animationId = null;

const renderLoop = () => {
  drawGrid();
  drawCells();
  universe.tick();
  animationId = requestAnimationFrame(renderLoop);
};

const isPaused = () => {
  return animationId === null;
};

const playPauseButton = document.getElementById("play-pause");

const play = () => {
  playPauseButton.textContent = "⏸";
  renderLoop();
};

const pause = () => {
  playPauseButton.textContent = "▶";
  cancelAnimationFrame(animationId);
  animationId = null;
};

playPauseButton.addEventListener("click", event => {
  if (isPaused()) {
    play();
  } else {
    pause();
  }
});
```

## Toggling a Cell’s State on `“click”` Events

To toggle a cell is to flip its state from alive to dead or from dead to alive. Add a `toggle` method to `Cell` in `wasm-game-of-life/src/lib.rs`

```rust
impl Cell {
    fn toggle(&mut self) {
        *self = match *self {
            Cell::Dead => Cell::Alive,
            Cell::Alive => Cell::Dead,
        };
    }
}
```

To toggle the state of a cell at given row and column, we translate the row and column pair into an index into the cells vector and call the toggle method on the cell at that index:

```rust
#[wasm_bindgen]
impl Universe {
    // ...
    pub fn toggle_cell(&mut self, row: u32, column: u32) {
        let idx = self.get_index(row, column);
        self.cells[idx].toggle();
    }
}
```

In `wasm-game-of-life/www/index.js`, we listen to click events on the `<canvas>` element, translate the click event's page-relative coordinates into canvas-relative coordinates, and then into a row and column, invoke the `toggle_cell` method, and finally redraw the scene

```jsx
canvas.addEventListener("click", event => {
  const boundingRect = canvas.getBoundingClientRect();

  const scaleX = canvas.width / boundingRect.width;
  const scaleY = canvas.height / boundingRect.height;

  const canvasLeft = (event.clientX - boundingRect.left) * scaleX;
  const canvasTop = (event.clientY - boundingRect.top) * scaleY;

  const row = Math.min(Math.floor(canvasTop / (CELL_SIZE + 1)), height - 1);
  const col = Math.min(Math.floor(canvasLeft / (CELL_SIZE + 1)), width - 1);

  universe.toggle_cell(row, col);

  drawGrid();
  drawCells();
});
```
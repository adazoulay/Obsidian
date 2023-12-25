# The Pipeline

A pipeline describes all the actions the gpu will perform when acting on a set of data

In this section, we will be creating a `RenderPipeline` specifically

****************Shaders:**************** mini-programs that you send to the gpu to perform operations on your data

- There are 3 main types of shader: vertex, fragment, and compute

**Vertex** is a point in 3d space (can also be 2d)

- These vertices are then bundled in groups of 2s to form lines and/or 3s to form triangles

![Untitled](The%20Pipeline/Untitled.png)

These triangles are stored as vertices which are the points that make up the corners of the triangles

We use a **vertex shader** to manipulate the vertices, in order to transform the shape to look the way we want it

The vertices are then converted into **fragments**

- Every pixel in the result image gets at least one fragment
- Each fragment has a color that will be copied to its corresponding pixel.
- The **fragment shader** decides what color the fragment will be

### WGSL

**[WebGPU Shading Language (opens new window)](https://www.w3.org/TR/WGSL/)**(WGSL) is the shader language for WebGPU

- focuses on getting it to easily convert into the shader language corresponding to the backend. Ex: SPIR-V for Vulkan, MSL for Metal…
- The conversion is done internally and we usually don't need to care about the details

## Writing the Shaders

In the same folder as `main.rs`, create a file `shader.wgsl`

### Vertex Shader

```rust
// Vertex shader

struct VertexOutput {
    @builtin(position) clip_position: vec4<f32>,
};

@vertex
fn vs_main(
    @builtin(vertex_index) in_vertex_index: u32,
) -> VertexOutput {
    var out: VertexOutput;
    let x = f32(1 - i32(in_vertex_index)) * 0.5;
    let y = f32(i32(in_vertex_index & 1u) * 2 - 1) * 0.5;
    out.clip_position = vec4<f32>(x, y, 0.0, 1.0);
    return out;
}
```

First, we declare `struct` to store the output of our vertex shader

- This consists of only one field currently which is our vertex's `clip_position`
- The `@builtin(position)` bit tells WGPU that this is the value we want to use as the vertex's **[clip coordinates (opens new window)](https://en.wikipedia.org/wiki/Clip_coordinates)**.
    
    Vector types such as `vec4` are generic. Currently, you must specify the type of value the vector will contain:
    Thus, a 3D vector using 32bit floats would be `vec3<f32>`.
    

The next part of the shader code is the `vs_main` function

- We are using `@vertex` to mark this function as a valid entry point for a vertex shader
- We expect a `u32` called `in_vertex_index` which gets its value from `@builtin(vertex_index)`
- We then declare a variable called `out` using our `VertexOutput` struct
- We create two other variables for the `x`, and `y`, of a triangle

Now we can save our `clip_position` to `out`. We then just return `out` and we're done with the vertex shader!

### Fragment Shader

```rust
// Fragment shader

@fragment
fn fs_main(in: VertexOutput) -> @location(0) vec4<f32> {
    return vec4<f32>(0.3, 0.2, 0.1, 1.0);
}
```

This sets the color of the current fragment to brown

The `@location(0)` bit tells WGPU to store the `vec4` value returned by this function in the first color target. We'll get into what this is later.

**Note:** about `@builtin(position)`, in the fragment shader this value is in **[framebuffer space (opens new window)](https://gpuweb.github.io/gpuweb/#coordinate-systems)**. This means that if your window is 800x600, the x and y of `clip_position` would be between 0-800 and 0-600 respectively with the y = 0 being the top of the screen

- This can be useful if you want to know pixel coordinates of a given fragment, but if you want the position coordinates you'll have to pass them in separately:
    
    ```rust
    struct VertexOutput {
    	@builtin(position) clip_position: vec4<f32>,
    	@location(0) vert_pos: vec3<f32>,
    }
    
    @vertex
    fn vs_main(
        @builtin(vertex_index) in_vertex_index: u32,
    ) -> VertexOutput {
        var out: VertexOutput;
        let x = f32(1 - i32(in_vertex_index)) * 0.5;
        let y = f32(i32(in_vertex_index & 1u) * 2 - 1) * 0.5;
        out.clip_position = vec4<f32>(x, y, 0.0, 1.0);
        out.vert_pos = out.clip_position.xyz;
        return out;
    }
    ```
    

## How do we use the Shaders?

This is the part where we finally make the thing in the title: the pipeline

First, let's modify `State` to include the following:

```rust
// lib.rs
struct State {
    surface: wgpu::Surface,
    device: wgpu::Device,
    queue: wgpu::Queue,
    config: wgpu::SurfaceConfiguration,
    size: winit::dpi::PhysicalSize<u32>,
    // NEW!
    render_pipeline: wgpu::RenderPipeline,
}
```

Now let's move to the `new()` method, and start making the pipeline
We'll have to load in those shaders we made earlier, as the `render_pipeline` requires those:

```rust
let shader = device.create_shader_module(wgpu::ShaderModuleDescriptor {
    label: Some("Shader"),
    source: wgpu::ShaderSource::Wgsl(include_str!("shader.wgsl").into()),
});
```

**Note:** You can also use `include_wgsl!` macro as a small shortcut to create the `ShaderModuleDescriptor`.

```rust
let shader = device.create_shader_module(wgpu::include_wgsl!("shader.wgsl"));
```

### Pipeline and PipelineLayout

One more thing, we need to create a `PipelineLayout`. We'll get more into this after we cover `Buffer`s

```rust
let render_pipeline_layout =
    device.create_pipeline_layout(&wgpu::PipelineLayoutDescriptor {
        label: Some("Render Pipeline Layout"),
        bind_group_layouts: &[],
        push_constant_ranges: &[],
    });
```

Finally, we have all we need to create the `render_pipeline`

```rust
let render_pipeline = device.create_render_pipeline(&wgpu::RenderPipelineDescriptor {
    label: Some("Render Pipeline"),
    layout: Some(&render_pipeline_layout),
    vertex: wgpu::VertexState {
        module: &shader,
        entry_point: "vs_main", // 1.
        buffers: &[], // 2.
    },
    fragment: Some(wgpu::FragmentState { // 3.
        module: &shader,
        entry_point: "fs_main",
        targets: &[Some(wgpu::ColorTargetState { // 4.
            format: config.format,
            blend: Some(wgpu::BlendState::REPLACE),
            write_mask: wgpu::ColorWrites::ALL,
        })],
    }),
    // continued ...
```

Several things to note:

1. Here you can specify which function inside the shader should be the `entry_point`. These are the functions we marked with `@vertex` and `@fragment`
2. The `buffers` field tells `wgpu` what type of vertices we want to pass to the vertex shader. 
    1. We're specifying the vertices in the vertex shader itself, so we'll leave this empty. We'll put something there in the next tutorial.
3. The `fragment` is technically optional, so you have to wrap it in `Some()`. 
    1. We need it if we want to store color data to the `surface`.
4. The `targets` field tells `wgpu` what color outputs it should set up. 
    1. Currently, we only need one for the `surface`. We use the `surface`'s format so that copying to it is easy, and we specify that the blending should just replace old pixel data with new data. We also tell `wgpu` to write to all colors: red, blue, green, and alpha. *We'll talk more about* `color_state` *when we talk about textures.*

```rust
primitive: wgpu::PrimitiveState {
        topology: wgpu::PrimitiveTopology::TriangleList, // 1.
        strip_index_format: None,
        front_face: wgpu::FrontFace::Ccw, // 2.
        cull_mode: Some(wgpu::Face::Back),
        // Setting this to anything other than Fill requires Features::NON_FILL_POLYGON_MODE
        polygon_mode: wgpu::PolygonMode::Fill,
        // Requires Features::DEPTH_CLIP_CONTROL
        unclipped_depth: false,
        // Requires Features::CONSERVATIVE_RASTERIZATION
        conservative: false,
    },
    // continued ...
```

1. The `primitive` field describes how to interpret our vertices when converting them into triangles
2. The `front_face` and `cull_mode` fields tell `wgpu` how to determine whether a given triangle is facing forward or not
    1. `FrontFace::Ccw` means that a triangle is facing forward if the vertices are arranged in a counter-clockwise direction
    2. Triangles that are not considered facing forward are culled (not included in the render) as specified by `CullMode::Back`

```rust
depth_stencil: None, // 1.
    multisample: wgpu::MultisampleState {
        count: 1, // 2.
        mask: !0, // 3.
        alpha_to_coverage_enabled: false, // 4.
    },
    multiview: None, // 5.
});
```

1. We're not using a depth/stencil buffer currently, so we leave `depth_stencil` as `None`. *This will change later*
2. `count` determines how many samples the pipeline will use. Multisampling is a complex topic, so we won't get into it here
3. `mask` specifies which samples should be active. In this case, we are using all of them.
4. `alpha_to_coverage_enabled` has to do with anti-aliasing. We're not covering anti-aliasing here, so we'll leave this as false now.
5. `multiview` indicates how many array layers the render attachments can have. We won't be rendering to array textures so we can set this to `None`

Now, all we have to do is add the `render_pipeline` to `State` and then we can use it!

```rust
// new()
Self {
    surface,
    device,
    queue,
    config,
    size,
    // NEW!
    render_pipeline,
}
```

## Using a Pipeline

If you run your program now, it'll take a little longer to start, but it will still show the blue screen we got in the last section

- That's because we created the `render_pipeline`, but we still need to modify the code in `render()` to actually use it

```rust
// render()

// ...
{
    // 1.
    let mut render_pass = encoder.begin_render_pass(&wgpu::RenderPassDescriptor {
        label: Some("Render Pass"),
        color_attachments: &[
            // This is what @location(0) in the fragment shader targets
            Some(wgpu::RenderPassColorAttachment {
                view: &view,
                resolve_target: None,
                ops: wgpu::Operations {
                    load: wgpu::LoadOp::Clear(
                        wgpu::Color {
                            r: 0.1,
                            g: 0.2,
                            b: 0.3,
                            a: 1.0,
                        }
                    ),
                    store: true,
                }
            })
        ],
        depth_stencil_attachment: None,
    });

    // NEW!
    render_pass.set_pipeline(&self.render_pipeline); // 2.
    render_pass.draw(0..3, 0..1); // 3.
}
// ...
```

We didn't change much, but let's talk about what we did change.

1. We renamed `_render_pass` to `render_pass` and made it mutable.
2. We set the pipeline on the `render_pass` using the one we just created
3. We tell `wgpu` to draw *something* with 3 vertices, and 1 instance. This is where `@builtin(vertex_index)` comes from
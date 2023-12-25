# Instancing

Our scene right now is very simple: we have one object centered at (0,0,0). 

What if we wanted more objects? 

This is where instancing comes in:

Instancing allows us to draw the same object multiple times with different properties

- position, orientation, size, color

There are multiple ways of doing instancing

1. One way would be to modify the uniform buffer to include these properties and then update it before we draw each instance of our object
    1. We don't want to use this method for performance reasons
    2. Updating the uniform buffer for each instance would require multiple buffer copies for each frame
    3. On top of that, our method to update the uniform buffer currently requires us to create a new buffer to store the updated data
2. `draw_indexed`

If we look at the parameters for the `draw_indexed` function **[in the wgpu docs (opens new window)](https://docs.rs/wgpu/latest/wgpu/struct.RenderPass.html#method.draw_indexed)**, we can see a solution to our problem

```rust
pub fn draw_indexed(
    &mut self,
    indices: Range<u32>,
    base_vertex: i32,
    instances: Range<u32> // <-- This right here
)
```

The `instances` parameter takes a `Range<u32>`

- This parameter tells the GPU how many copies, or instances, of the model we want to draw
- Currently, we are specifying `0..1`, which instructs the GPU to draw our model once, and then stop

The fact that `instances` is a `Range<u32>` may seem weird as using `1..2` for instances would still draw 1 instance of our object

- Seems like it would be simpler to just use a `u32` right?
- The reason it's a range is that sometimes we don't want to draw **all** of our objects

## The Instance Buffer

We'll create an instance buffer in a similar way to how we create a uniform buffer

First, we'll create a struct called `Instance`

```rust
struct Instance {
    position: cgmath::Vector3<f32>,
    rotation: cgmath::Quaternion<f32>,
}
```

Note: A `Quaternion` is a mathematical structure often used to represent rotation. The math behind them is beyond me (it involves imaginary numbers and 4D space) so I won't be covering them here.

Using these values directly in the shader would be a pain as quaternions don't have a WGSL analog

- we'll convert the `Instance` data into a matrix and store it into a struct called `InstanceRaw`

Now we need to add 2 fields to `State`: `instances`, and `instance_buffer`.

```rust
struct State {
    instances: Vec<Instance>,
    instance_buffer: wgpu::Buffer,
}
```

The `cgmath` crate uses traits to provide common mathematical methods across its structs such as `Vector3`, and these traits must be imported before these methods can be called

```rust
use cgmath::prelude::*;
```

We'll create the instances in `new()`. We'll use some constants to simplify things. We'll display our instances in 10 rows of 10, and they'll be spaced evenly apart.

```rust
const NUM_INSTANCES_PER_ROW: u32 = 10;
const INSTANCE_DISPLACEMENT: cgmath::Vector3<f32> = cgmath::Vector3::new(NUM_INSTANCES_PER_ROW as f32 * 0.5, 0.0, NUM_INSTANCES_PER_ROW as f32 * 0.5);
```

Now we can create the actual instances.

```rust
impl State {
    async fn new(window: Window) -> Self {
        // ...
        let instances = (0..NUM_INSTANCES_PER_ROW).flat_map(|z| {
            (0..NUM_INSTANCES_PER_ROW).map(move |x| {
                let position = cgmath::Vector3 { x: x as f32, y: 0.0, z: z as f32 } - INSTANCE_DISPLACEMENT;

                let rotation = if position.is_zero() {
                    // this is needed so an object at (0, 0, 0) won't get scaled to zero
                    // as Quaternions can effect scale if they're not created correctly
                    cgmath::Quaternion::from_axis_angle(cgmath::Vector3::unit_z(), cgmath::Deg(0.0))
                } else {
                    cgmath::Quaternion::from_axis_angle(position.normalize(), cgmath::Deg(45.0))
                };

                Instance {
                    position, rotation,
                }
            })
        }).collect::<Vec<_>>();
        // ...
    }
```

Now that we have our data, we can create the actual `instance_buffer`

```rust
let instance_data = instances.iter().map(Instance::to_raw).collect::<Vec<_>>();
let instance_buffer = device.create_buffer_init(
    &wgpu::util::BufferInitDescriptor {
        label: Some("Instance Buffer"),
        contents: bytemuck::cast_slice(&instance_data),
        usage: wgpu::BufferUsages::VERTEX,
    }
);
```

We're going to need to create a new `VertexBufferLayout` for `InstanceRaw`.

```rust
impl InstanceRaw {
    fn desc() -> wgpu::VertexBufferLayout<'static> {
        use std::mem;
        wgpu::VertexBufferLayout {
            array_stride: mem::size_of::<InstanceRaw>() as wgpu::BufferAddress,
            // We need to switch from using a step mode of Vertex to Instance
            // This means that our shaders will only change to use the next
            // instance when the shader starts processing a new instance
            step_mode: wgpu::VertexStepMode::Instance,
            attributes: &[
                wgpu::VertexAttribute {
                    offset: 0,
                    // While our vertex shader only uses locations 0, and 1 now, in later tutorials we'll
                    // be using 2, 3, and 4, for Vertex. We'll start at slot 5 not conflict with them later
                    shader_location: 5,
                    format: wgpu::VertexFormat::Float32x4,
                },
                // A mat4 takes up 4 vertex slots as it is technically 4 vec4s. We need to define a slot
                // for each vec4. We'll have to reassemble the mat4 in
                // the shader.
                wgpu::VertexAttribute {
                    offset: mem::size_of::<[f32; 4]>() as wgpu::BufferAddress,
                    shader_location: 6,
                    format: wgpu::VertexFormat::Float32x4,
                },
                wgpu::VertexAttribute {
                    offset: mem::size_of::<[f32; 8]>() as wgpu::BufferAddress,
                    shader_location: 7,
                    format: wgpu::VertexFormat::Float32x4,
                },
                wgpu::VertexAttribute {
                    offset: mem::size_of::<[f32; 12]>() as wgpu::BufferAddress,
                    shader_location: 8,
                    format: wgpu::VertexFormat::Float32x4,
                },
            ],
        }
    }
}
```

We need to add this descriptor to the render pipeline so that we can use it when we render.

```rust
let render_pipeline = device.create_render_pipeline(&wgpu::RenderPipelineDescriptor {
    // ...
    vertex: wgpu::VertexState {
        // ...
        // UPDATED!
        buffers: &[Vertex::desc(), InstanceRaw::desc()],
    },
    // ...
});
```

The last change we need to make is in the `render()` method. We need to bind our `instance_buffer` and we need to change the range we're using in `draw_indexed()` to include the number of instances.

```rust
render_pass.set_pipeline(&self.render_pipeline);
render_pass.set_bind_group(0, &self.diffuse_bind_group, &[]);
render_pass.set_bind_group(1, &self.camera_bind_group, &[]);
render_pass.set_vertex_buffer(0, self.vertex_buffer.slice(..));
// NEW!
render_pass.set_vertex_buffer(1, self.instance_buffer.slice(..));
render_pass.set_index_buffer(self.index_buffer.slice(..), wgpu::IndexFormat::Uint16);

// UPDATED!
render_pass.draw_indexed(0..self.num_indices, 0, 0..self.instances.len() as _);
```

We need to reference the parts of our new matrix in `shader.wgsl` so that we can use it for our instances. 

Add the following to the top of `shader.wgsl`.

```rust
struct InstanceInput {
    @location(5) model_matrix_0: vec4<f32>,
    @location(6) model_matrix_1: vec4<f32>,
    @location(7) model_matrix_2: vec4<f32>,
    @location(8) model_matrix_3: vec4<f32>,
};
```

We need to reassemble the matrix before we can use it.

We'll apply the `model_matrix` before we apply `camera_uniform.view_proj`. We do this because the `camera_uniform.view_proj` changes the coordinate system from `world space` to `camera space`

Our `model_matrix` is a `world space` transformation, so we don't want to be in `camera space` when using it

```rust
var out: VertexOutput;
out.tex_coords = model.tex_coords;
out.clip_position = camera.view_proj * model_matrix * vec4<f32>(model.position, 1.0);
return out;
```

With all that done we should see a forest of Trees!
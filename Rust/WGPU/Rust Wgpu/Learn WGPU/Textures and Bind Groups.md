# Textures and Bind Groups

Up to this point, we have been drawing super simple shapes

While we can make a game with just triangles, trying to draw highly detailed objects would massively limit what devices could even run our game

However, we can get around this problem with **textures**

### Textures

Textures are images overlaid on a triangle mesh to make it seem more detailed

There are multiple types of textures such as normal maps, bump maps, specular maps, and diffuse maps

We're going to talk about diffuse maps, or more simply, the color texture

## Loading an Image from a File

If we want to map an image to our mesh, we first need an image. Let's use this happy little tree:

![Untitled](Textures%20and%20Bind%20Groups/Untitled.png)

We'll use the **[image crate (opens new window)](https://docs.rs/image)**to load our tree. Let's add it to our dependencies:

```toml
[dependencies.image]
version = "0.24"
default-features = false
features = ["png", "jpeg"]
```

The jpeg decoder that `image` includes uses **[rayon (opens new window)](https://docs.rs/rayon)**to speed up the decoding with threads

WASM doesn't support threads currently so we need to disable this so that our code won't crash when we try to load a jpeg on the web

Decoding jpegs in WASM isn't very performant. If you want to speed up image loading in general in WASM you could opt to use the browser's built-in decoders instead of `image` when building with `wasm-bindgen`. This will involve creating an `<img>` tag in Rust to get the image, and then a `<canvas>` to get the pixel data, but I'll leave this as an exercise for the reader

In `State`'s `new()` method add the following just after configuring the `surface`:

```rust
surface.configure(&device, &config);
// NEW!

let diffuse_bytes = include_bytes!("happy-tree.png");
let diffuse_image = image::load_from_memory(diffuse_bytes).unwrap();
let diffuse_rgba = diffuse_image.to_rgba8();

use image::GenericImageView;
let dimensions = diffuse_image.dimensions();
```

Here we grab the bytes from our image file and load them into an image which is then converted into a `Vec` of rgba bytes

We also save the image's dimensions for when we create the actual `Texture`

Now, let's create the `Texture`

```rust
let texture_size = wgpu::Extent3d {
    width: dimensions.0,
    height: dimensions.1,
    depth_or_array_layers: 1,
};
let diffuse_texture = device.create_texture(
    &wgpu::TextureDescriptor {
        // All textures are stored as 3D, we represent our 2D texture
        // by setting depth to 1.
        size: texture_size,
        mip_level_count: 1, // We'll talk about this a little later
        sample_count: 1,
        dimension: wgpu::TextureDimension::D2,
        // Most images are stored using sRGB so we need to reflect that here.
        format: wgpu::TextureFormat::Rgba8UnormSrgb,
        // TEXTURE_BINDING tells wgpu that we want to use this texture in shaders
        // COPY_DST means that we want to copy data to this texture
        usage: wgpu::TextureUsages::TEXTURE_BINDING | wgpu::TextureUsages::COPY_DST,
        label: Some("diffuse_texture"),
        // This is the same as with the SurfaceConfig. It
        // specifies what texture formats can be used to
        // create TextureViews for this texture. The base
        // texture format (Rgba8UnormSrgb in this case) is
        // always supported. Note that using a different
        // texture format is not supported on the WebGL2
        // backend.
        view_formats: &[],
    }
);
```

## Getting data into a Texture

The `Texture` struct has no methods to interact with the data directly. However, we can use a method on the `queue` we created earlier called `write_texture` to load the texture in

```rust
queue.write_texture(
            // Tells wgpu where to copy the pixel data
            wgpu::ImageCopyTexture {
                texture: &diffuse_texture,
                mip_level: 0,
                origin: wgpu::Origin3d::ZERO,
                aspect: wgpu::TextureAspect::All,
            },
            // The actual pixel data
            &diffuse_rgba,
            // The layout of the texture
            wgpu::ImageDataLayout {
                offset: 0,
                bytes_per_row: Some(4 * dimensions.0),
                rows_per_image: Some(dimensions.1),
            },
            texture_size,
        );
```

## TextureViews and Samplers

Now that our texture has data in it, we need a way to use it

This is where a `TextureView` and a `Sampler` come in

- A `TextureView` offers us a *view* into our texture
- A `Sampler` controls how the `Texture` is *sampled*
    - Our program supplies a coordinate on the texture (known as a *texture coordinate*), and the sampler then returns the corresponding color based on the texture and some internal parameters

Let's define our `diffuse_texture_view` and `diffuse_sampler` now:

```rust
// We don't need to configure the texture view much, so let's
// let wgpu define it.
let diffuse_texture_view = diffuse_texture.create_view(&wgpu::TextureViewDescriptor::default());
let diffuse_sampler = device.create_sampler(&wgpu::SamplerDescriptor {
    address_mode_u: wgpu::AddressMode::ClampToEdge,
    address_mode_v: wgpu::AddressMode::ClampToEdge,
    address_mode_w: wgpu::AddressMode::ClampToEdge,
    mag_filter: wgpu::FilterMode::Linear,
    min_filter: wgpu::FilterMode::Nearest,
    mipmap_filter: wgpu::FilterMode::Nearest,
    ..Default::default()
});
```

The `address_mode_*` parameters determine what to do if the sampler gets a texture coordinate that's outside the texture itself. We have a few options to choose from:

- `ClampToEdge`: Any texture coordinates outside the texture will return the color of the nearest pixel on the edges of the texture.
- `Repeat`: The texture will repeat as texture coordinates exceed the texture's dimensions.
- `MirrorRepeat`: Similar to `Repeat`, but the image will flip when going over boundaries.

![Untitled](Textures%20and%20Bind%20Groups/Untitled%201.png)

The `mag_filter` and `min_filter` fields describe what to do when the sample footprint is smaller or larger than one texel

There are 2 options:

- `Linear`: Select two texels in each dimension and return a linear interpolation between their values.
- `Nearest`: Return the value of the texel nearest to the texture coordinates. This creates an image that's crisper from far away but pixelated up close. This can be desirable, however, if your textures are designed to be pixelated, like in pixel art games, or voxel games like Minecraft

Mipmaps are a complex topic and will require their own section in the future. For now, we can say that `mipmap_filter` functions similar to `(mag/min)_filter` as it tells the sampler how to blend between mipmaps

All these different resources are nice and all, but they don't do us much good if we can't plug them in anywhere

This is where `BindGroup`s and `PipelineLayout`s come in

# The BindGroup

A `BindGroup` describes a set of resources and how they can be accessed by a shader

We create a `BindGroup` using a `BindGroupLayout`

Let's make one of those first

```rust
let texture_bind_group_layout =
            device.create_bind_group_layout(&wgpu::BindGroupLayoutDescriptor {
                entries: &[
                    wgpu::BindGroupLayoutEntry {
                        binding: 0,
                        visibility: wgpu::ShaderStages::FRAGMENT,
                        ty: wgpu::BindingType::Texture {
                            multisampled: false,
                            view_dimension: wgpu::TextureViewDimension::D2,
                            sample_type: wgpu::TextureSampleType::Float { filterable: true },
                        },
                        count: None,
                    },
                    wgpu::BindGroupLayoutEntry {
                        binding: 1,
                        visibility: wgpu::ShaderStages::FRAGMENT,
                        // This should match the filterable field of the
                        // corresponding Texture entry above.
                        ty: wgpu::BindingType::Sampler(wgpu::SamplerBindingType::Filtering),
                        count: None,
                    },
                ],
                label: Some("texture_bind_group_layout"),
            });
```

Our `texture_bind_group_layout` has two entries: one for a sampled texture at binding 0, and one for a sampler at binding 1

- Both of these bindings are visible only to the fragment shader as specified by `FRAGMENT`
    - The possible values for this field are any bitwise combination of `NONE`, `VERTEX`, `FRAGMENT`, or `COMPUTE`

With `texture_bind_group_layout`, we can now create our `BindGroup`

```rust
let diffuse_bind_group = device.create_bind_group(
    &wgpu::BindGroupDescriptor {
        layout: &texture_bind_group_layout,
        entries: &[
            wgpu::BindGroupEntry {
                binding: 0,
                resource: wgpu::BindingResource::TextureView(&diffuse_texture_view),
            },
            wgpu::BindGroupEntry {
                binding: 1,
                resource: wgpu::BindingResource::Sampler(&diffuse_sampler),
            }
        ],
        label: Some("diffuse_bind_group"),
    }
);
```

`BindGroup` is a more specific declaration of the `BindGroupLayout`

- The reason they're separate is that it allows us to swap out `BindGroup`s on the fly, so long as they all share the same `BindGroupLayout`
- us to swap out `BindGroup`s on the fly, so long as they all share the same `BindGroupLayout`. Each texture and sampler we create will need to be added to a `BindGroup`

Now that we have our `diffuse_bind_group`, let's add it to our `State` struct

```rust
impl State {
    async fn new() -> Self {
        // ...
        Self {
            surface,
            device,
            queue,
            config,
            size,
            render_pipeline,
            vertex_buffer,
            index_buffer,
            num_indices,
            // NEW!
            diffuse_bind_group,
        }
    }
}
```

Now that we've got our `BindGroup`, we can use it in our `render()` function

## Pipeline Layout

Remember the `PipelineLayout` we created back in **[the pipeline section](https://sotrh.github.io/learn-wgpu/beginner/tutorial5-textures/learn-wgpu/beginner/tutorial3-pipeline#how-do-we-use-the-shaders)**? Now we finally get to use it!

The `PipelineLayout` contains a list of `BindGroupLayout`s that the pipeline can use

Modify `render_pipeline_layout` to use our `texture_bind_group_layout`

```rust
// ...
    let render_pipeline_layout = device.create_pipeline_layout(
        &wgpu::PipelineLayoutDescriptor {
            label: Some("Render Pipeline Layout"),
            bind_group_layouts: &[&texture_bind_group_layout], // NEW!
            push_constant_ranges: &[],
        }
    );
```

### A change to VERTICES

There are a few things we need to change about our `Vertex` definition

Up to now, we've been using a `color` attribute to set the color of our mesh

Now that we're using a texture, we want to replace our `color` with `tex_coords`

- These coordinates will then be passed to the `Sampler` to retrieve the appropriate color

Since our `tex_coords` are two dimensional, we'll change the field to take two floats instead of three

First, we'll change the `Vertex` struct:

```rust
#[repr(C)]
#[derive(Copy, Clone, Debug, bytemuck::Pod, bytemuck::Zeroable)]
struct Vertex {
    position: [f32; 3],
    tex_coords: [f32; 2], // NEW!
}
```

And then reflect these changes in the `VertexBufferLayout`:

```rust
impl Vertex {
    fn desc() -> wgpu::VertexBufferLayout<'static> {
        use std::mem;
        wgpu::VertexBufferLayout {
            array_stride: mem::size_of::<Vertex>() as wgpu::BufferAddress,
            step_mode: wgpu::VertexStepMode::Vertex,
            attributes: &[
                wgpu::VertexAttribute {
                    offset: 0,
                    shader_location: 0,
                    format: wgpu::VertexFormat::Float32x3,
                },
                wgpu::VertexAttribute {
                    offset: mem::size_of::<[f32; 3]>() as wgpu::BufferAddress,
                    shader_location: 1,
                    format: wgpu::VertexFormat::Float32x2, // NEW!
                },
            ]
        }
    }
}
```

Lastly, we need to change `VERTICES` itself. Replace the existing definition with the following:

```rust
// Changed
const VERTICES: &[Vertex] = &[
    Vertex { position: [-0.0868241, 0.49240386, 0.0], tex_coords: [0.4131759, 0.99240386], }, // A
    Vertex { position: [-0.49513406, 0.06958647, 0.0], tex_coords: [0.0048659444, 0.56958647], }, // B
    Vertex { position: [-0.21918549, -0.44939706, 0.0], tex_coords: [0.28081453, 0.05060294], }, // C
    Vertex { position: [0.35966998, -0.3473291, 0.0], tex_coords: [0.85967, 0.1526709], }, // D
    Vertex { position: [0.44147372, 0.2347359, 0.0], tex_coords: [0.9414737, 0.7347359], }, // E
];
```

### Shader Time

With our new `Vertex` structure in place, it's time to update our shaders

We'll first need to pass our `tex_coords` into the vertex shader and then use them over to our fragment shader to get the final color from the `Sampler`

Let's start with the vertex shader

```rust
// Vertex shader

struct VertexInput {
    @location(0) position: vec3<f32>,
    @location(1) tex_coords: vec2<f32>,
}

struct VertexOutput {
    @builtin(position) clip_position: vec4<f32>,
    @location(0) tex_coords: vec2<f32>,
}

@vertex
fn vs_main(
    model: VertexInput,
) -> VertexOutput {
    var out: VertexOutput;
    out.tex_coords = model.tex_coords;
    out.clip_position = vec4<f32>(model.position, 1.0);
    return out;
}
```

Now that we have our vertex shader outputting our `tex_coords`, we need to change the fragment shader to take them in

```rust
@group(0) @binding(0)
var t_diffuse: texture_2d<f32>;
@group(0)@binding(1)
var s_diffuse: sampler;

@fragment
fn fs_main(in: VertexOutput) -> @location(0) vec4<f32> {
    return textureSample(t_diffuse, s_diffuse, in.tex_coords);
}
```

The variables `t_diffuse` and `s_diffuse` are what's known as uniforms

- For now, all we need to know is that `group()` corresponds to the 1st parameter in `set_bind_group()`
- `binding()` relates to the `binding` specified when we created the `BindGroupLayout` and `BindGroup`

## Result:

Our tree is upside down!

This is because wgpu's world coordinates have the y-axis pointing up, while texture coordinates have the y-axis pointing down

- In other words, (0, 0) in texture coordinates corresponds to the top-left of the image, while (1, 1) is the bottom right

We can get our triangle right-side up by replacing the y coordinate `y` of each texture coordinate with `1 - y`:

```rust
const VERTICES: &[Vertex] = &[
    // Changed
    Vertex { position: [-0.0868241, 0.49240386, 0.0], tex_coords: [0.4131759, 0.00759614], }, // A
    Vertex { position: [-0.49513406, 0.06958647, 0.0], tex_coords: [0.0048659444, 0.43041354], }, // B
    Vertex { position: [-0.21918549, -0.44939706, 0.0], tex_coords: [0.28081453, 0.949397], }, // C
    Vertex { position: [0.35966998, -0.3473291, 0.0], tex_coords: [0.85967, 0.84732914], }, // D
    Vertex { position: [0.44147372, 0.2347359, 0.0], tex_coords: [0.9414737, 0.2652641], }, // E
];
```

## Cleaning Things Up

For convenience, let's pull our texture code into its own module. We'll first need to add the **[anyhow (opens new window)](https://docs.rs/anyhow/)**crate to our `Cargo.toml` file to simplify error handling;

Then, in a new file called `src/texture.rs`, add the following:

```rust
use image::GenericImageView;
use anyhow::*;

pub struct Texture {
    pub texture: wgpu::Texture,
    pub view: wgpu::TextureView,
    pub sampler: wgpu::Sampler,
}

impl Texture {
    pub fn from_bytes(
        device: &wgpu::Device,
        queue: &wgpu::Queue,
        bytes: &[u8], 
        label: &str
    ) -> Result<Self> {
        let img = image::load_from_memory(bytes)?;
        Self::from_image(device, queue, &img, Some(label))
    }

    pub fn from_image(
        device: &wgpu::Device,
        queue: &wgpu::Queue,
        img: &image::DynamicImage,
        label: Option<&str>
    ) -> Result<Self> {
        let rgba = img.to_rgba8();
        let dimensions = img.dimensions();

        let size = wgpu::Extent3d {
            width: dimensions.0,
            height: dimensions.1,
            depth_or_array_layers: 1,
        };
        let texture = device.create_texture(
            &wgpu::TextureDescriptor {
                label,
                size,
                mip_level_count: 1,
                sample_count: 1,
                dimension: wgpu::TextureDimension::D2,
                format: wgpu::TextureFormat::Rgba8UnormSrgb,
                usage: wgpu::TextureUsages::TEXTURE_BINDING | wgpu::TextureUsages::COPY_DST,
                view_formats: &[],
            }
        );

        queue.write_texture(
            wgpu::ImageCopyTexture {
                aspect: wgpu::TextureAspect::All,
                texture: &texture,
                mip_level: 0,
                origin: wgpu::Origin3d::ZERO,
            },
            &rgba,
            wgpu::ImageDataLayout {
                offset: 0,
                bytes_per_row: std::num::NonZeroU32::new(4 * dimensions.0),
                rows_per_image: std::num::NonZeroU32::new(dimensions.1),
            },
            size,
        );

        let view = texture.create_view(&wgpu::TextureViewDescriptor::default());
        let sampler = device.create_sampler(
            &wgpu::SamplerDescriptor {
                address_mode_u: wgpu::AddressMode::ClampToEdge,
                address_mode_v: wgpu::AddressMode::ClampToEdge,
                address_mode_w: wgpu::AddressMode::ClampToEdge,
                mag_filter: wgpu::FilterMode::Linear,
                min_filter: wgpu::FilterMode::Nearest,
                mipmap_filter: wgpu::FilterMode::Nearest,
                ..Default::default()
            }
        );
        
        Ok(Self { texture, view, sampler })
    }
}
```

We need to import `texture.rs` as a module, so somewhere at the top of `lib.rs` add the following

```rust
mod texture;
```

The texture creation code in `new()` now gets a lot simpler:

```rust
surface.configure(&device, &config);
let diffuse_bytes = include_bytes!("happy-tree.png"); // CHANGED!
let diffuse_texture = texture::Texture::from_bytes(&device, &queue, diffuse_bytes, "happy-tree.png").unwrap(); // CHANGED!

// Everything up until `let texture_bind_group_layout = ...` can now be removed.
```

We still need to store the bind group separately so that `Texture` doesn't need to know how the `BindGroup` is laid out

The creation of `diffuse_bind_group` slightly changes to use the `view` and `sampler` fields of `diffuse_texture`

```rust
let diffuse_bind_group = device.create_bind_group(
    &wgpu::BindGroupDescriptor {
        layout: &texture_bind_group_layout,
        entries: &[
            wgpu::BindGroupEntry {
                binding: 0,
                resource: wgpu::BindingResource::TextureView(&diffuse_texture.view), // CHANGED!
            },
            wgpu::BindGroupEntry {
                binding: 1,
                resource: wgpu::BindingResource::Sampler(&diffuse_texture.sampler), // CHANGED!
            }
        ],
        label: Some("diffuse_bind_group"),
    }
);
```

Finally, let's update our `State` field to use our shiny new `Texture` struct, as we'll need it in future tutorials.
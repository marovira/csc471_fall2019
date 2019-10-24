# The Rendering Pipeline

Divided into the following stages:

1. Application,
2. Geometry processing,
3. Rasterization, and
4. Pixel processing.

## Application

This is the program running on the CPU. Drives the pipeline. Notice that compute
shaders also fall into this category.

## Geometry processing

Divided into:

1. Vertex shaders,
2. clipping,
3. screen mapping.

The vertex shader is what we have seen. Initially it was used to compute colours
based on the rationale that there are less vertices than fragments. It is charge
of transforming from model coordinates to world coordinates (model matrix), to
view coordinates (view or camera matrix), and finally clip coordinates
(projection matrix). It is followed by 3 optional stages:

* Tessellation: can increase/decrease the number of triangles or primitives,
* Geometry: can generate more geometries are needed (there is an argument that
  the vertex shader is now useless and everything should be done here, but it is
  somewhat slow),
* Stream output: known as transform feedback in OpenGL. Allows the shader to
  write data back into the vertex buffer to perform computations (similar to the
  compute shader in a way).

Clipping is in charge of taking only the primitives that are in the viewing
frustrum (as determined by the projection matrix) and passing them forward. This
does include creating new vertices as needed to perform the clips.

Once clipping is done, they are transformed into screen (or window if we add the
z axis) coordinates. The z value is clamped to [0, 1] and then is passed on to
the raster stage.

## Rasterization

It's the one in charge of taking the final transformed geometry and converting
it into pixels. This is done by sampling and is (for the most part) outside of
our control and done internally in the GPU. It is divided into two stages:

* Triangle setup: sets up all the data structures and information required so we
  can then traverse the triangles.
* Triangle traversal: walk through every triangle and check to see if a pixel is
  covered by it. If it is, then mark it as covered for the next stage.

## Pixel Processing

This is where the final colours are computed before being displayed on the
screen. It is divided into:

1. Pixel shading: in charge of computing the colour of each fragment. This can
   be done using lighting equations, textures, etc. Notice that by this point
   all the data from the vertices has been interpolated.
2. Merging: this is the stage that performs the final merge of the colours
   computed by the fragment shaders. It is also in charge of using the z-buffer
   to check whether collisions occur and discard any fragments that are covered
   by others. In addition, this stage can also do blending by writing into the
   alpha channel, scissor testing (discard fragments outside of a rectangle on
   the screen), stencil testing (check against a buffer to see whether fragments
   should be discarded), amongst others.

Out of all the stages in the pipeline, the following are **programmable**:

1. Vertex shaders,
2. Tessellation and geometry,
3. Fragment shaders.

The following are **configurable** (that is to say, the programmer can change
their behaviour, but they are not programmable):

1. Merging.

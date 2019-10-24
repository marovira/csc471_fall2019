# Implementing Shading Models

In this class we will discuss the implementation (and optimization) details that
can (or should) be considered whenever implementing a shading algorithm. Before
we begin, it is important to note that whenever an operation (or sequence of
operations) is described as "slow" it usually refers to a bigger context where
shading is a part of an entire operation that needs to be performed to produce a
single frame. Furthermore, we will be assuming that we have the time constraint
of producing a framerate of 60FPS. With this in mind, let us examine the
implementation details of Phong (and Gouraud) shading.

## Frequency of Evaluations

The first thing to consider is *frequency of evaluations*. The question is the
following: for a specific operation (or sequence), how many times does this
sequence need to be performed?

* At program install: disabling/enabling of any OpenGL extensions, any
  optimizations made for hardware, etc.
* At program startup: load meshes, shaders, etc.; determine DPI, scaling, and
  similar quantities; system speed could also be used somehow; etc.

From here we can now move on to the area of computations that need to be
performed on the frames themselves. Operations that don't have to be performed
every frame should be bundled together and handled separately (even
asynchronously if at all possible). These operations can include: certain
physics updates, certain lighting simulations, amongst others. Keep in mind that
with OpenGL, only **one** thread may submit commands to the GPU. That isn't to
say that only one thread may do the work though. What we can do instead is to
split the work among threads and have each thread then submit commands to a
queue which a *single* thread executes.

Changes that need to be performed per-frame can then be further subdivided into
two categories: operations that are going to remain constant in throughout the
frame, and ones that aren't. Any variables that will remain constant throughout
the frame should:

1. Be computed (preferably) on the CPU,
2. Be sent to the shaders as uniforms (either variables or buffers).

Consider the case of the MVP matrix. All of those matrices would be constant in
a single frame, so they're going to be sent to the GPU as uniforms. Moreover,
the computation of the MVP matrix itself is always constant, so we compute it on
the CPU, along with things like:

* The inverse transpose of the model matrix,
* The model-view matrix,
* etc.

An interesting example that comes from the assignments is the usage of the
inverse of the view matrix to compute the view position. Inverting a matrix is a
relatively expensive operation, so it can be (probably should be) computed on
the CPU and the result sent as a uniform into the shader.

The next question comes in decided whether to use buffers or variables. If you
have several variables that belong together, bundle them as buffers. This allows
the driver to optimize memory allocation on the GPU and can increase
performance. As a general rule of thumb, it's good to strive to having no loose
uniform variables.

Data that needs to change per-frame and within the same frame must be handled in
the shaders and then forwarded down the pipeline as is relevant. The choice of
shader will determine the number of times its called.

Recall that the outputs of the geometry stage of the pipeline will get
interpolated before they reach the fragment shaders. Things that usually get
interpolated are:

* vertex positions,
* normals, and
* tangents.

This explains why Gouraud shading looks that way: the specular component is
interpolated linearly across the surface. Now an argument can be made that we
can compute the diffuse and ambient components and then forward them to the
fragment shader which will then compute the specular component. While this might
work, the overhead introduced by the extra outputs is not worth the effort.

An interesting side-effect of interpolation is that while the vertex shader
might produce unit normals, linearly interpolating will make them non-unit. This
is way normals are usually normalized in both shaders. Things like light
positions and view directions aren't interpolated, though they can be if
required. If they need to be interpolated, then they should be interpolated
*before* normalization to prevent artifacts.

Another optimization can be done by choosing the coordinate space. Since this
involves matrix multiplications (which while not expensive aren't cheap either),
we can optimize certain parts of the computation to remain in world space (if
say we have a large number of lights) and another in view space (if we wish to
produce higher detail in the view region for example). In fact, Phong shading
can be performed in either world or view space. The important part is to ensure
that both shaders agree on the space.

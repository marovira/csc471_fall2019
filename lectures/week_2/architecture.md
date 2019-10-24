# Architecture of the GPU

* Graphics acceleration began in software. First implementing simple things like
  blending neighbouring pixels, and then moving on to more complex things like
  the z-buffer.
* Eventually, these functions were moved on to hardware for even faster
  processing. This is the start of graphics specific hardware. The term GPU was
  coined by Sony in 1994 referring to the original Playstation, whereas NVidia
  popularized it in 1999 with the GeForce 256.
* The biggest issue that the GPU faces is *latency*. Because the GPU is further
  away from the CPU (and hence main memory), the waiting times for resources are
  much higher.

## Data-Parallel Architecture

* A CPU is a parallel architecture in that it can execute multiple tasks at
  once. However, each core ultimately operates in a linear fashion. Latency is
  solved in the CPU by using fast cache memory, along with tricks like branch
  prediction, instruction re-ordering, etc. The usage of different levels of
  cache memory comes at the cost of silicon for cores.
* The GPU goes the other way around. It contains a large number of cores, but a
  small amount of cache to store things. These cores are called *shader cores*
  and they can number in the thousands (current NVidia 2080 RTX Ti has 4352
  cores).
* The idea is that the GPU operates on streams of similar data, where the same
  operations need to be performed on multiple data points (SIMD). These are
  *independent* of each other, hence parallelization is trivial. Synchronization
  is possible, but comes at the cost of performance.
* To understand how the GPU works consider a card with a single core. It needs
  to operate on 6,000 pixels to compute colour. The core is equipped with
  registers to perform its task.  The shader that it's currently running
  contains a few operations, and then a texture look-up. Operations are trivial
  and no cost is incurred since they run on the registers. Textures are handled
  separately, so this incurs in a stall. Instead of stopping, the core then
  switches to the next pixel until the same thing happens, and so on. Eventually
  it returns to the first pixel to find the texture data is back. It then
  continues execution until it stalls again or it finishes.
* In reality, the GPU leverages SIMD to to execute the same instruction in
  *lock-step* on a fixed number of shader programs. Moving to modern GPU, our
  pixel shader invocation is known as a *thread*. Unlike its CPU counterpart, it
  consists of a bit of memory for the input values, and registers to execute
  things. Threads that execute the same shader are bundled together into *warps*
  (NVidia) or *wavefronts* (AMD). A warp is scheduled to execute on 8 to 64
  cores using SIMD. Each thread is mapped to a *SIMD lane*.
* Go back to our example with 6,000 threads to be executed. In NVidia, a warp is
  32 cores, so this results in 62.5 warps or 63 warps. The execution of the warp
  is similar to our previous example. They all execute instructions in
  *lock-step*, so if they stall, they all stall together. If a warp stalls, it
  is simply swapped for another warp and execution continues. Note that this can
  happen for simpler things than texture look-ups, since swapping warps is very
  low cost. This "hides" the latency problem we faced before. 
* Some factors influence how well this works. If a shader requires more data in
  the registers, then there can be less threads, and hence less warps. Warps
  that are currently on the GPU are said to be "in flight" and the number is
  referred to as *occupancy*. High occupancy is desirable, since it means that
  less warps are idle.
* Dynamic branching is also a problem. If all threads in a warp branch in the
  same direction, nothing happens. However, if some threads branch in different
  directions, then the GPU *must* compute both results and then throw away the
  result that isn't required. This can further reduce performance by leaving
  parts of a warp idle.

We will now re-visit the pipeline and see how it's designed at the hardware
level.

## Evolution of Graphics API

* The idea for programmable shading dates back to 1984 with Cook's *shade
  trees*. The RenderMan Shading Language was developed with this idea and is
  still used in film production along with the *Open Shading Language*.
* The first consumer-grade graphics hardware was introduced by 3dfx in 1996.
  Voodoo used a fixed function pipeline throughout. Effects were done with
  multi-pass renders.
* NVidia GeForce 256 was the first GPU. It wasn't programmable but it was
  configurable.
* 2001 NVidia introduced the first GPU to support programmable pipelines
  (GeForce 3).
* The shaders were written in assembly-like language and then converted by the
  drivers. DirectX 8.0 introduced the pixel shader, but it was really just to
  blend textures. It could not really be programmed, had only 12 instructions or
  less, and lacked functionality. Shaders did not support control flow, so
  conditionals were emulated by executing both sides of the if-statement and
  then selecting or interpolating the results.
* 2002 was when vertex and pixel shaders were fully programmable in DirectX 9.0.
  OpenGL handled this through extensions. DirectX also uses *shader models* to
  describe capabilities of hardware. 2002 marked the release of Shader model
  2.0.
* DirectX 9.0 also introduced HLSL with collaboration from NVidia, and around
  the same time OpenGL ARB (Architecture Review Board) introduced GLSL.
* Shader Model 3.0 was introduced in 2004 and added dynamic flow control, making
  shaders more powerful. At this point, consoles such as XBox 360 and
  Playstation 3 used SM 3.0. Nintendo's Wii was the last console to use a fixed
  function pipeline in 2006.

* In 2006, NVidia unveiled CUDA. Before this point, general purpose programming
  was done by tricking the GPU into computing data through textures and
  framebuffers. CUDA was the first API built on C that allowed the GPU to be
  used for general purposes (GPGPU). OpenCL was later released in 2009, which
  allowed the use of both the GPU and CPU for computations and was supported by
  all graphics cards.
* At the end of 2006, DirectX 10.0 was released, which introduced the geometry
  shader as well as stream output. It introduced a uniform programming model for
  all shaders, as well as integer data types. OpenGL 3.3 provided the same
  functionality.
* In 2009, DirectX 11 was released along with Shader Model 5.0. It introduced
  the tessellation and compute shaders. OpenGL introduced tessellation in 4.0
  and compute shaders in 4.3.
* OpenGL and DirectX evolve differently due to who controls them. DirectX is
  controlled by Microsoft, so they can work directly with hardware vendors to
  produce the API. OpenGL is controlled by Khronos, which is a group of various
  vendors including NVidia, AMD, Microsoft, Apple (passively), amongst others.
  While OpenGL does advance at a slower rate, new functionality is always
  available through *extensions*.
* In 2013, AMD and DICE created Mantle. The idea was to reduce the driver
  overhead and to efficiently handle CPU multi-core architectures. The ideas
  that Mantle introduced were picked up by Microsoft and released in 2015 in
  DirectX 12. While DirectX 12 does **not** introduce new functionality, it does
  provide a massive redesign of the API that more closely matches modern GPU
  architectures.
* Apple released its own version called Metal in 2014. One of the major
  improvements was the reduction of CPU usage, which in turn reduces power
  consumption.  The API is meant for both graphics and compute tasks.
* After AMD donated Mantle to Khronos, they released Vulkan in 2016. It shares
  the same compatibilities that OpenGL has, though removes GLSL in favour of a
  new intermediate language called SPIR-V. Shaders are now pre-compiled, making
  them more portable.
* As far as mobile devices are concerned, the norm has been OpenGL ES (Embedded
  Systems). While OpenGL and DirectX both evolve as the hardware evolves, OpenGL
  ES does not. Released in 2003, OpenGL ES was a stripped down version of OpenGL
  1.3 with the fixed function pipeline. In 2010 the first iPad released with
  OpenGL ES 1.1. In 2007 OpenGL ES 2.0 was released with contained programmable
  shading. It was based on OpenGL 2.0 but without the fixed function pipeline,
  so it was not backwards compatible with OpenGL ES 1.1. OpenGL ES 3.0 was
  released in 2012 with multiple render targets, texture compression, transform
  feedback, instancing, shader language improvements, amongst others. OpenGL ES
  3.1 adds compute shaders, and 3.2 adds geometry and tessellation shaders.
* WebGL was released in 2011, and was based on OpenGL ES 2.0, though more
  advanced features were available through extensions. WebGL 2.0 assumes OpenGL
  Es 3.0.
* It is important to note that while Vulkan, Metal, and DirectX 12 all have
  compute capabilities, they're not really designed to be full-compute APIs. In
  the case of Vulkan, there is support for interoperability with OpenCL.

## A Brief Note on GPGPUs

* Recall that 2006 NVidia released CUDA. What's significant about the GPU that
  implemented it (GeForce 8800 GTX) is that it was the first time a GPU ever had
  architecture specifically for general purpose programming.
* Instead of splitting resources (warps) between shaders (vertex, fragment,
  etc), CUDA architecture allowed *all* warps on the GPU to be used for
  computing a specific task. Furthermore, the GPU was built to comply with IEEE
  requirements for single-precision floating-point arithmetic along with
  arbitrary read and write access to memory as well as access to
  software-managed cache known as *shared memory*.
* The other big innovation was that CUDA was its own language (built on top of
  C) and as such was completely divorced from OpenGL/DirectX.

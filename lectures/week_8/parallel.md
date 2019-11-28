# Multiprocessing

* Traditional APIs have evolved to have fewer functions that do more. Metal,
  Vulkan, and DX12 go through a different route.
* For these APIs, the drivers are streamlined and minimal, offloading the
  complexity and responsibility of validating state to the calling application,
  as well as memory allocation and other functions. This gives us the advantage
  of being able to use multiple CPU processors to call the API.
* By 2003, the speed of processors flattened out. The solution was then to start
  cramming more cores per chip. This results in a higher performance per unit
  area, which is why GPUs are so efficient. The challenge is then creating
  programs that exploit concurrency.
* We can classify multiprocessor computers into two:
    * *Message-passing:* each processor has its own memory area, and messages
      are sent across processors to communicate results.
    * *Shared memory processors:* all processors share a logical address space
      of memory among themselves. Most popular systems use *symmetric
      multiprocessing* design, where all the processors are identical.
* For real-time graphics, we are going to talk about two main methods to use
  multiple processors:
    * *multiprocessor pipelining:* also called temporal parallelism.
    * *Parallel processing:* also called spatial parallelism.
  Both of these can be brought together with *task-based parallelism*.

## Multiprocessor Pipelining

So far we have seen pipelining in the context of the GPU, where each stage
forwards data to the next and the overall speed is determined by the slowest
stage. Each stage is run in parallel. We can also run a pipeline on the host,
called *multiprocessor pipelining* or *software pipelining*.

* We are going to focus on a type of software pipelining, but there are many
  more.
* Consider an application divided into three stages: app, cull, and draw. The
  app stage must go first and therefore drives the pipeline itself. The roles of
  these are:
    * App: can do things like collision detection, update viewpoint, etc.
    * Cull: frustrum culling, level of detail, state sorting, and generation of
      objects to be rendered.
    * Draw: issues all the graphics calls for the list of objects to be
      rendered.
* If we have one processor, do everything in one. Otherwise split the tasks
  across processors. How this split is performed depends on the tasks
  themselves. If 3 are available, execute each in its own core.
* The advantage of this technique is that throughput (i.e. the rendering speed)
  increases, the downside is that latency is greater. Latency, or temporal
  delay, is the time it takes from the polling of the user's actions to the
  final image.
* Note that this is *not* the same as the framerate. It describes how quickly
  the system can react to the inputs of the user, not how fast we can take to
  put the image on the screen (though it is involved). 
* Ignoring this delay, multiprocessing has more latency than parallel processing
  because of the use of the pipeline. In fact, latency increases with the number
  of stages in the pipeline (synchronization).

## Parallel Processing

* The major disadvantage of multiprocessor pipelining is that latency increases
  with the number of stages, which is a problem for certain applications.
* The idea with parallel processing is to try to run sections of the code
  concurrently. Suppose we have $n$ processors. We then split the work package
  into $n$ pieces, each processor taking care of one. When all processors are
  done, we may need to merge them, which is why the workload must by highly
  predictable. This is called static assignment.
* When this is not the case, we use dynamic assignment. The idea is to create
  work pools, and jobs are therefore assigned to the pools. The processors can
  then fetch one or more jobs from the queue when they have finished. It is
  important that only one CPU can fetch a particular job in order to keep the
  overhead of maintaining the queue low. Larger jobs mean that maintaining the
  queue is less difficult, but if they get too large they might hinder
  performance.
* The speedup in both types of parallelization is $n$.

## Task based parallelism

The idea is to break a job into tasks that can then be executed in parallel. The
tasks must then have the following properties:

* The task has a well-defined input and output.
* The task is independent and stateless when run and always completes.
* It is not so large a task that it often becomes the only process running.

## Graphics API Multiprocessing Support

* Historically, only one thread was allowed to access the driver at any time.
* There are two operations that the driver can perform in parallel: resource
  creation and render-related calls. For example, creating a texture can be done
  in parallel in the CPU. However, note that these can also be blocking tasks as
  they might trigger events on the GPU. Older APIs needed to be rewritten to
  support this concurrency.
* A key construct used is called the *command buffer*, which is an API of state
  change and draw calls. These can be created, stored, and replayed. They can
  also be combined to form larger command buffers.
* Only a single CPU processor communicates with the GPU via the driver so it can
  send a buffer for execution. However, every processor can create or
  concatenate stored command buffers in parallel.
* An advantage is that they can be stored and replayed, furthermore, they are
  not bound when created. For example, the view matrix changes, but this is
  stored in a uniform buffer. The command buffer does not hold the ubo, only a
  reference to it. Hence the ubo can be updated without reconstructing the
  command buffer.
* This system actually predates modern APIs. That said, certain semantics in
  earlier APIs did not allow the driver to parallelize various operations, which
  helped motivate the creation of Vulkan et al.

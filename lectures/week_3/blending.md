# Transparency and Alpha

There are two main ways in which semi-transparent objects can be rendered:

* *View-based:* The semi-transparent objects are rendered in such a way that the
  any objects behind can be seen.
* *Light-based:* the objects cause light to be attenuated or diverted, causing
  other objects in the scene to be rendered differently. Examples include
  frosted glass windows, refraction, attenuation due to thickness, etc.

For the time being we will focus on view-based effects.

A simple method for simulating transparency is called *screen-door
transparency*. The idea is to render the transparent triangle with a
pixel-aligned checkerboard pattern. In effect, we are rendering every other
pixel. The idea is that if the pixels are close enough (and small enough) then
it will properly simulate transparency. The main drawback is that exactly two
objects can be rendered in this way. Any more than that will not be shown
correctly. While it is possible to create a different kind of checkerboard
pattern that is not 50%, these tend to create detectable patterns. That being
said, their main advantage is simplicity. No additional hardware is needed, and
transparent objects can be created at any time and in any order.

A second option is to use *stochastic transparency* which uses subpixel
screen-door masks combined with stochastic sampling. This approach is
relatively simple, though it does produce noise in the final image. It is also
worth noting that this does require a large number of samples to create a
convincing image, which incurs in both performance and memory costs. That said,
this approach does not require any blending, and any form of antialiasing and
transparency are both executed by the same system.

The traditional way of performing transparencies is by blending the colour of
the object behind with the one in front. This technique is known as *alpha
blending*. The basic idea is the following: when each pixel is rendered, we
write an RGB value along with a fourth value called $\alpha$. This information
is then paired with the z-buffer to blend the colours together. If $\alpha$ is
1, then the colour in front is selected. If $\alpha$ is 0, then the remaining
colour is used. Otherwise, the colours are added together using their
corresponding $\alpha$ values to determine the final colour.

## Blending Order

The process of blending is usually performed with an operator called **over**,
which is defined as:

$$ c_0 = \alpha_sc_s + (1 - \alpha_s)c_d $$

Where $c_s$ is the colour of the transparent object (called *source*),
$\alpha_s$ is the object's alpha, $c_d$ is the pixel colour before blending
(called the *destination*) and $c_o$ is the resulting colour due to placing the
transparent object **over** the existing scene. In the case of the rendering
pipeline sending in $c_s$ and $\alpha_s$, the pixel's original colour $c_d$ gets
replaced by the result $c_o$. If the incoming $\alpha = 1$, then the object is
opaque and it is rendered normally.

This type of alpha blending works well to simulate things like a gauzy fabric.
It fails, however, to accurately simulate things like holding a piece of red
glass over a blue object. In this case, the blue object looks dark because the
light it emits does not make it through the red filter. 

Another operator that is commonly used is *additive blending*, where the pixel
values are summed together as follows:

$$ c_o = \alpha_sc_s + c_d$$

This blending mode works well to simulate effects such as lightning or sparks
that brighten the pixels behind them, but does not work well for general
transparency. Opaque objects do not appear filtered and semi-transparent objects
are over-saturated.

To correctly render transparencies, we first must render opaque objects with
blending off, and then render transparent objects with blending on, which
effectively places them **over** the opaque objects. While it is true that we
could render with blending on in both passes, the performance overhead is simply
not worth it. The main limitation with this technique is that the z-buffer can
only hold one object stored per-pixel. This means that if multiple transparent
objects cover an opaque object, only the first object to be rendered will be
seen. This issue can be solved by first rendering the opaque objects, and then
*sorting* the transparent objects in reverse order, at which point they can be
rendered in this way. The sorting can be done by distance in relation to the
view direction. While it works well, it is just an approximation and does have
its problems. For example, objects classified as ore distant bay be in front of
objects considered nearer. Objects that interpenetrate are impossible to resolve
on a per-mesh basis from all angles. Even concavities can present this problem.
That said, this technique is often used due to its simplicity and speed, as
well as needing no further memory or GPU support.

The **over** equation can also be modified so that blending front to back gives
the same result. This blending mode is called the **under** operator:

$$ c_o = \alpha_dd_c + (1 - \alpha_d)\alpha_sc_s $$
$$ a_o = \alpha_s(1 - \alpha_d) + \alpha_d = \alpha_s - \alpha_s\alpha_d +
\alpha_d $$

Notice that in this case we require the destination to maintain its alpha value,
whereas **over** does not. Other than this, **under** is equivalent to **over**,
but notice that the formula for computing alpha is order-independent. This
equation comes from considering the fragment's alphas as coverages. The general
idea is that if two areas are fragments are covering the same area, then the
output is the same as adding the two areas, and then subtracting the area where
they overlap.

## Order-Independent Transparency

The **under** operator is used by drawing all transparent objects to a separate
colour buffer, and then merging this colour buffer with the opaque scene using
**over**. Another use is for an *order-independent transparency (OIT)* called
depth-peeling. The general idea is to use two z-buffers and multiple passes.
First, render the scene (both opaque and transparent objects) and store the
depth data to the z-buffer. Next render only transparent objects and store their
depths in the second z-buffer. If the z-depth of an object matches the value in
the first z-buffer, we know it is the closes transparent object, and so we save
the RGBA value into a separate colour buffer. We then "peel" the layer by saving
the z-depth of whichever object is beyond the first z-depth and is closest (the
second-closest). The process continues peeling layers using **under**. We stop
after some number of passes and then blend the two images together.

This algorithm works well, but it has some drawbacks: first we don't know how
many passes will be sufficient. Second, we require multiple render passes to
fully render the image. This approach works well since the closest objects (the
one's users will see first) are rendered first and back objects are rendered
last. We can set a cut-off after a certain alpha level where the back objects
can be discarded. An alternative is to go from the back and move forwards,
though this suffers the disadvantage of rendering the closest objects *last*,
potentially cutting off objects in the process.

Another option is called the A-buffer. The idea is that each triangle creates a
coverage mask for each screen grid cell it fully or partially covers. Each pixel
stores a list of all relevant fragments. Opaque fragments cull fragments behind
them. Once the lists are done, the final fragment colour is computed by walking
through the list.

The idea of creating linked lists was made possible with functionality exposed
in DirectX 11 which includes unordered access views and atomic operations. The
main issue with the A-buffer is that, while only the required data is stored, we
have no way of knowing how much storage is required. Depending on the scene,
this storage can be costly (up to 200 semi-transparent objects).

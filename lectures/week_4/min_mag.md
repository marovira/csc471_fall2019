# Image Texturing

Assume that we have a texture of size $256 \times 256$ texels and we want to map
it onto a square. If the square is roughly the same equivalent size as the
texture, then the image will look fine. However, if the quad is much bigger, we
will have to expand the image via a process called *magnification*. Conversely,
if the quad is too small, we will need to shrink the image via *minification*.
We will now discuss the general techniques for both.

## Magnification

The two main techniques for enlarging a texture are the following:

* **Nearest Neighbour:** The idea is to use the value of the nearest neighbour
  to the fragment. This works as you might expect and results in very
  *pixelated* images, since we're basically repeating pixels.
* **Bilinear Interpolation:** This works as follows: for each pixel, find the 4
  neighbouring pixels and linearly interpolate the two dimensions to find the
  blended value for the pixel. The result is blurrier, and gets rid of a lot of
  jaggedness.

An alternative (though more expensive) is to use higher-order interpolations,
such as *bicubic interpolation* (or *cubic convolution*). This is not supported
by hardware, but it can be implemented in the shader.

Now explain how linear interpolation works. Note that this is called `GL_LINEAR`
in code, while nearest neighbour is `GL_NEAR`. A solution to blurriness is to
use *detail textures*. These are textures that are overlain onto the magnified
texture. Another option is to not use bilinear interpolation alone, but to clamp
the values past certain thresholds in order to retain a similar effect.

## Minification

We have the same approaches as magnification (and the same commands for OpenGL).
Note that bilinear interpolation fails when more than 4 texels cover a pixel. To
better solve this problem, we use mipmaps. The basic idea is to create a series
of textures by halving the texture using a $2\times 2$ sampler. This process
continues recursively until we reach a single texel. This results in a box
filter, which is the worst kind we can have. That said, there are better ways of
creating mipmaps by using Gaussian or other filters.

# Shadows

We will cover a variety of techniques for generating shadows, mostly by
providing the intuition behind the techniques. Some notation:

* Light source,
* occluder (object that casts the light),
* receiver (object that receives the light),
* umbra + penumbra = shadow. Umbra is dark and penumbra is the transition.

## Planar shadows

The idea is that the shadow can be constructed by projecting the occluder onto
the receiver. This can be done using similar triangles to determine the
projection matrix that we need to use. Once we have this, we then execute a
second render pass to render our objects as shadows. Notice that this assumes
that the light is always above, otherwise we get an *anti shadow*. 

Soft shadows occur when the light has area. We can approximate this by
subdividing the area into point lights and then rendering each one in turn,
accumulating the result into a texture and then blending everything together.
This can give good results if a high number of passes is used, but this comes
with a heavy performance hit. Furthermore, for $n$ lights, we can only create $n
+ 1$ colours. This technique is used to create the ground-truth against which
other techniques are compared.

Another solution is to use a Gaussian blur. Performance concerns aside, this
only works provided that the object does not have a part that is closer to the
receiver than the rest.

## Shadows on Curved Surfaces

Another option is to render the scene from the view point of the light. Anything
that the light sees is rendered white, and all else is black. The texture is
then projected onto the receiving plane. This has the downside that the
occluders and receivers **must** be distinct, and that under no circumstances
can the occluding objects shadow themselves. Also the receiver must always be
further away than the occluder.

## Shadow Maps

Same idea: render from the light but capture the depth buffer. Shadows are
created by comparing depth. If the object is further away than the corresponding
depth in the buffer, then it is in shadow. 

If a light is outside the scene (like the sun) then this works fine and we use
an orthographic projection to ensure that all objects are captured in the view
frustrum. If the light is inside the scene, then we need to use an
*omni-directional shadow map* by rendering a cube texture. Care must be taken to
ensure that we don't get any artifacts on the seams of the cube.

The accuracy of this approach is tied to the resolution of the depth buffer and
the numerical precision of the z-buffer. This results in *self-shadow aliasing*
or *shadow acne*. This has two main causes:

* The precision of the depth buffer itself,
* and the location of the samples in relation to the screen.

This results in triangles shadowing themselves. A way to fix this is to use an
offset bias. The idea is that when we compare the depth value with the fragment
value, we subtract from the receiver's distance. If we do this constantly, we
will get artifacts if the receiver isn't parallel with the light. What we can do
is use an offset that depends on the angle between the receiver and the light.
We can do this with OpenGL with `glPolygonOffset` to shift each polygon away
from the light. The downside of using offsets is that they can lead to *Peter
Panning* or *light leaks* which occur when the object appears to float slightly
above the underlying surface. This occurs because the area beneath the point of
contact is pushed too far and so does not receive a shadow.

To have soft shadows, we use PCH: we basically sample the 4 nearest neighbours
and linearly interpolate.

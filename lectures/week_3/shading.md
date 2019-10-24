# Shading

## Shading on Shaders

Your assignments asked you to implement Phong and Gouraud shading. The
equations as given in the assignments, showcase the flexibility of modern
programmable shaders. Built-in operations include:

* All arithmetic operations on shaders, including dot and cross products,
* Matrix multiplications, as well as transposing and inverting,
* Mathematical operations such as sine and cosine,
* Other convenience functions such as *reflection*.

Both Phong and Gouraud ultimately rely on light sources to compute the diffuse
and specular highlights. We will discuss primarily 2 types of light sources:

1. **Directional lights:** these are the simplest lights. Examples are: the sun (as
   viewed from Earth) or a flood-light 20ft away on a diorama. Their defining
   characteristic is that their direction and colour are constant throughout the
   scene (save for the colour which can be shadowed). In the case of
   implementation, the light direction is no longer computed with the light
   position (as there technically isn't one). We simply use the light direction
   as given.
2. **Punctual Lights:** These are lights that have a defined position in space.
   Not to be confused with "point lights", since these are a kind of punctual
   light. Unlike directional lights, we do require their position to compute the
   light direction vector. There are two kinds of punctual lights we will
   discuss: point/omni lights and spotlights.

### Point/Omni Lights

These are lights that shine uniformly in every direction. In fact, Phong and
Gouraud both rely on point lights for their equations. It is worth noting
however, that the lights shown in these models are "infinite" in so far as they
do not decrease in intensity the further away the model is from the light. If we
were to look at the spacing between light rays as we move further away from the
model, we would see that their spacing increases proportionally to the inverse
of the square distance. This implies that the density of the rays (and therefore
the intensity) decreases proportionally to $1/r^2$.  Hence the light colour
changes based on: $c(r) = c_0 (\frac{1}{r})^2$.

This formulation presents with some issues. First, consider the case when $r$ is
very small. As $r$ tends to 0, the value of that fraction tends to infinity,
until we reach a divide-by-zero singularity. We can fix this by adding a small
value of $\epsilon$ to the denominator, resulting in $c = c_0 \frac{1}{r^2 +
\epsilon}$. As an example of a value for $\epsilon$, unreal engine uses 1cm. An
alternative to this equation (used by CryEngine and Frostbite) is to clamp the
value of $r$ to some minimum value. This has the advantage that, unlike the
arbitrary choice of $\epsilon$, we can choose the minimum distance based on on a
physical interpretation: the radius of the physical object emitting the light.

The second issue is that at large distances, the light remains strong. While
this isn't generally a problem with visuals, it is a problem of performance.
Note that the intensity of the light will continue to trend to 0, but will never
reach it. As a result, it would be in our best interests for it to go to 0 at
some point. Ideally, what we would want would be to have a smooth transition,
meaning that both the intensity and its derivative should reach 0 at the same
distance. A solution is to apply a *windowing function*. An example used by both
Unreal and Frostbite is:

$$f(r) = (1 - (\frac{r}{r_{max})^4)^2 $$

It is worth noting that in this equation, the value is clamped prior to
squaring (if negative 0). Depending on the application we could require
different results. For example, *Just Cause 2* used a simpler form, replacing
the 4th power with a square in order to increase performance. Another option
would be to change the equation based on style or artistic reasons. *Tomb Raider
(2013)* used spline-editing to edit the values of the falloff curves.

### Spotlights

Unlike point lights, most light sources vary by both direction and distance.
This can be expressed byt using a directional falloff function which combines
with the distance falloff function to define the following equation:

$$ c = c_0 f(r) d(\mathbf{L})$$

An important type of effect is the *spotlight*, which projects light in a
circular cone. Since a spotlight's directional falloff function is symmetric
with respect to rotation around its direction vector $s$, we can express it as a
function of the angle $\theta_s$ between $s$ and the reversed light vector $-L$.
Note that we reverse it because we define $L$ at the surface pointing towards
the light, whereas here we need it coming from the light itself.

Most spotlights use $\theta_s$ in some way. They also have a *umbra angle*
$\theta_u$, which bounds the light such that $d(L) = 0 \forall \theta_s \geq
\theta_u$. This angle can also bue sued for culling in a similar manner to the
maximum falloff distance we saw before. It is also common for spotlights to have
a *penumbra angle* $\theta_p$ which defines an inner cone where the light is at
full intensity.

The falloff functions for spotlights tend to be somewhat similar. Consider the
following example used in Frostbite:

$$ t = \clamp(\frac{\cos\theta_s - \cos\theta_u}{\cos\theta_p - \cos\theta_u})$$
$$ d(L) = t^2$$

There are other types of punctual lights that may be used. The $d(L)$ function
is not limited to the examples shown above and can be made to be anything that
is required, including models from real-world light sources. The Illuminating
Engineering Society have defined a standard fiel format for such measurements.
IES profiles are available from many manufacturers and are used in the game
*Kilzone: Shadow Fall* as well as Unreal and Frostbite. The game *Tomb Raider
(2013)* has a type of punctual light that applies independent falloff functions
per axis. The curves can also be varied over time to produce a flickering torch.

## Other Light Types

The defining characteristic of directional and punctual lights is how the light
direction is computed. Given this, we can create different lights by changing
this function. For example, *Tomb Raider* also uses capsule lights that use a
line segment instead of a point. For each shaded pixel, the direction is the
closest point.

The light sources we have discussed here are *abstractions*. In reality, lights
have a size and shape, and they illuminate surfaces from different directions.
In rendering, these are called *area lights* and their use is increasing. There
are two categories for rendering techniques:

1. Those that simulate the softening of shadow edges that results from the area
   light being partially occluded,
2. and those that simulate the effect of the area light on a smooth surface.

The second kind is most noticeable for smooth, mirror-like surfaces, where the
light's shape and size can be clearly seen. The main reason for their increase
use is that approximations for them are becoming relatively inexpensive to
implement, along with increased GPU performance.

## Particles

In an *ideal gas* particles do not affect each other and thus their relative
positions are completely random and uncorrelated. While this is an abstraction,
it serves as a way to model air at normal atmospheric pressure. When light
interacts with an ideal gas, the waves that are scattered from different
molecules are random and changing. This results in the scattering being
incoherent, and so the waves are aggregated linearly. If we increase the packing
of the molecules into clusters, then the scattered waves from each cluster are i
phase with each other and so the waves aggregate quadratically. Intuitively, the
closer and larger these clusters become, the higher the intensity of the
scattered light. Thus this explains why fog and clouds scatter light so
strongly. 

When we discuss light scattering, the term *particles* will refer to both the
isolated molecules as well as the clusters. Main reason for this is that
scattering from a cluster whose diameter is smaller than the wavelength is an
amplified version of the scattering that occurs from a single molecule. This
type of scattering is called *Rayleigh scattering* n the case of atmospheric
particles and *Tyndall scattering* in the case of particles embedded in solids.
If we increase the width of the particles beyond a wavelength, the scattered
waves are no longer in sync throughout the particle, therefore changing the
characteristics of the scattering. In particular, this type of scattering
increasingly favours the forward direction and is called *Mie scattering*.

## Media

Another important case occurs when light moves through a *homogeneous medium*,
which is a volume filled with uniformly spaced *identical* molecules. The
spacing between the molecules does not need to be perfectly regular like a
crystal. Liquids (provided that they are pure, contain no other molecules, and
have no gaps or bubbles) are therefore equivalent to crystals.

In a homogeneous medium, the waves line up in such a way that they all interfere
destructively with each other in all directions except for the original
direction of propagation. Once all of the waves have been aggregated together,
we end up with the original wave, save for some differences in phase and even
amplitude. Note that due to destructive interference, the final wave presents no
scattering.

The ratio of the phase velocities of the original is called the index of 
refraction $n$. Some media is *absorptive* and therefore convert part of the
light energy to heat which causes the wave amplitude to decrease exponentially
with distance. The rate at which the wave's amplitude decreases is called the
attenuation index $k$. The complex index of refraction $n + ik$ defines how the
medium affects a given wavelength. While phase velocity does not affect the
appearance of light, *changes* in the velocity does. Absorption reduces the
intensity of light and can also change its colour.

Nonhomogeneous media can often be modeled as homogeneous media with embedded
scattering particles. Recall that in a homogeneous medium, the alignment of the
particles suppresses scattering through destructive interference, hence *any*
change in the alignment will break the pattern and allow the propagation of
scattering light waves. In fact, gases can be modelled in this fashion.

It is important to note that scattering and absorption are both
*scale-dependent*. In other words, while the air in a room or water in a glass
may not appear to scatter or absorb light in any meaningful way, the effect is
definitely noticeable in the ocean water or the atmosphere itself.

The appearance of a medium is dependent on a combination of scattering and
absorption. Specifically, the degree of cloudiness is determined by the degree
of scattering. Even higher degrees of scattering will lead to an opaque medium.
The colour of the medium  on the other hand, is determined by the wavelength
dependence of the absorption. Outside of very specific cases, particles in solid
and liquid media tend to be larger that the wavelength of light, and therefore
scatter all visible wavelengths equally. Lightness is therefore a combination of
both. A white medium is a combination of high scattering and low absorption.

## Surfaces

From an optical perspective, we can define a surface as the interface between
two volumes of different indices of refraction. In the typical case, on one side
we have air, with a refractive index of 1 (for the sake of simplicity), while
the refractive index of the other volume would depend on the material that it is
made of. Whenever light hits a surface, two components affect the behaviour of
light: the surface geometry, and the substances on either side of the surface. 

Let's consider the simplest possible case by taking a perfectly flat plane. The
index of refraction of the "outside" volume is then $n_1$ and the one for the
"inside" volume is $n_2$. Recall that scattering occurs whenever there is a
discontinuity in the index of refraction. Since the surface is itself a special
kind of discontinuity, the following things happen whenever a light wave hits a
surface:

1. At the surface, the scattered waves are either in phase, or 180 out of phase.
   What this ultimately means is that the scattered waves can only go in two
   directions: forwards into the surface, and backwards from it. These are the
   *transmitted wave* and *reflected wave* respectively.
2. The scattered waves must have the same frequency as the incident wave.
3. As light moves from one medium to another, the phase velocity (the speed
   at which the waves travel through a medium) is proportional to the relative
   index of refraction $n_1 / n_2$.

The way that light is refracted is known as *Snell's Law*. It is important to
note that while refraction is usually thought of as occurring with clear
materials such as glass or crystals, it also happens on opaque materials as
well. When light interacts with opaque materials, light is scattered and
absorbed within the materials medium.

The cases we have discussed present a discontinuity of the index of refraction
that is abrupt and occurs within a region of whose distance is less than a
single wavelength. If the change is more gradual, it causes light to bend, which
can be seen when the temperature of the air varies.

So far we discussed the case of a flat plane. However this does not exist in
reality. The important part to note is that if the irregularities are smaller
than the wavelength, then they have no effect. On the other hand, irregularities
that are considerably bigger will simply affect the tilt of the surface. Hence
its only irregularities that are of a specific size that affect the way the
surface looks through a process called *diffraction*.

In rendering, we typically use *geometric optics* which ignores things like
interference and diffraction. Additionally, we treat light as rays instead of
waves. At the point the light intersects, we treat the surface as a locally flat
plane.

Consider the case when the surface irregularities are smaller than the pixel,
called *microgeometry*. Then, even though a single point of will reflect light
in a specific direction, the pixel will cover multiple points that reflect light
in multiple different directions. The final colour of the pixel is then
determined by aggregating all of these reflections. As this can be
computationally expensive, we model the surface as having a random distribution
of microstructure normals. As a result, we model the surface as reflecting (and
refracting) light in a continuous spread of directions. The width of this spread
depends on the statistical variance of the microgeometry normal vectors. In
other words, the surface microscale *roughness*.

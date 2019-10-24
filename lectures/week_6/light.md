# Physics of Light

The way light interacts with matter forms the foundation for physically based
rendering. In order to understand these interactions, we need to have a basic
understanding of the nature of light.

In optics, light is modeled as an electromagnetic *transverse wave*, which is a
wave where the electric and magnetic fields oscillate perpendicularly to the
direction of its propagation. The oscillation of these two fields are coupled.
The magnetic and electric fields are perpendicular to each other and the ratio
of their lengths is fixed. This ratio is equal to the phase velocity, which we
will discuss later.

Consider the case of a simple light wave that consists of the simplest kind of
wave: a sine function. This wave has a single *wavelength*, denoted by
$\lambda$. Since the perceived colour of light is strongly related to its
wavelength, we call this type of light *monochromatic light*. Most lights
however are *monochromatic*. This simple light wave is also special in that it
is *linearly polarized*. This means that for a fixed point in space, the
electric and magnetic fields each move back and forth along a line. In general
we are more interested in *unpolarized* light, where the field oscillations are
spread equally over all directions perpendicular to the propagation axis.

If we track a point on the wave with a given phase (for example, the amplitude
peak) over time, we will see it move through space at a constant speed, which is
called the *phase velocity*. For light travelling in a vacuum, the phase
velocity is called $c$, commonly referred to as the speed of light.
In optics, it is often useful to talk about the size of features relative to the
wavelength $\lambda$. For visible light, the wavelength is usually between
400-700 nanometers. To put this into context, this is about half of the third of
the width of a single thread of spider silk, which is less than a fiftieth of
the width of human hair. As a result, the spider silk is $2\lambda - 3\lambda$
wide, while human hair is $100\lambda - 300\lambda$.

Light waves carry energy. The density of energy flow is then the product of the
magnitudes of the electric and magnetic fields which is (since the magnitudes
are proportional to each other) proportional to the squared magnitude of the
electric field. We will focus on the electric field since it affects matter more
strongly than the magnetic field. In rendering, we are concerned with the
*average* energy flow over time, which is proportional to the squared wave
amplitude. This average energy flow is the *irradiance* $E$. 

Light waves combine linearly. This means that the total wave is the sum of the
component waves. This leads to an apparent paradox since irradiance is
proportional to the square of the amplitudes. For example, would summing two
equal waves not lead to a $1 + 1 = 4$? And since irradiance measures energy
flow, would this not violate the conservation of energy? The answer is
"sometimes" and "no".

To illustrate, lets look at a simple case: the addition of $n$ monochromatic
waves, identical except for phase. The amplitude of each of the $n$ waves is
$a$. As mentioned earlier, $E_1$ of each wave is proportional to $a^2$, or in
other words $E_1 = ka^2$ for some $k$.
Suppose that all the waves line up with the same phase. Then $E_n = k(na)^2 =
n^2 E_1$. In this case, the waves reinforce each other, and so the combined wave
irradiance is $n^2$ times that of a single wave, which is $n$ times greater than
the sum of the irradiance values of a single wave. This situation is called
*constructive interference*. Now consider the case where each pair of waves is
in opposing phase, cancelling each other out. This is called *destructive
interference*. 

Both of these are special cases of *coherent addition*, where the peaks and
troughs of the waves align in some consistent way. Depending on the relative
phase relationships, coherent addition of $n$ identical waves can result in a
sum wave irradiance of anywhere between 0 and $n^2$ times that of the individual
wave. That said, most often waves are mutually *incoherent*, which means that
their phases are relatively random. In this scenario, the amplitude of the
combined wave is $\sqrt{n} a$ and the irradiance of individual waves adds up
linearly to $n$ times the irradiance of one wave, as one would expect.

It is important to note that while constructive and destructive interference
appear to violate the conservation of energy, this is in reality not the case.
As waves move through space, there are regions where waves interfere
constructively, gaining energy, while in others they interfere destructively
thereby loosing energy. Hence the sum total energy remains the same.

Light waves are emitted when electric charges in an object oscillate. Part of
the energy that caused the oscillations (heat, electrical energy, chemical
energy) is then converted to light energy, which is radiated away from the
object. In rendering, these are our light sources. After these waves are
emitted, they will continue to move through space until they encounter some
matter that they can interact with. The core phenomenon of the majority of
light-matter interactions is simple, and quite similar to the emission case.
When light encounters matter, the oscillating electrical field pushes and pulls
at the electrical field in the matter, causing them to oscillate in turn. This
oscillation therefore produces new light waves, which redirect some of the
incoming light wave in new directions. This reaction, called *scattering* is the
basis for a wide variety of optical phenomena.

A scattered light wave has the same frequency as the original wave. In the case
where the wave contains multiple frequencies of light, these interact with
matter separately. Incoming light energy at one frequency does not contribute to
the emitted light energy at a different frequency except for specific (and
relatively rare) cases such as fluorescence and phosphorescence which we will
not discuss in this class.

An isolated molecule scatters light in all directions, with some directional
variation in intensity. More light is scattered in directions close to the
original axis of propagation, both forward and backward. The molecule's
effectiveness as a scatterer (the chance that a light wave in its vicinity will
be scattered at all) varies strongly by wavelength. Short-wavelength light is
scattered much more effectively than longer-wavelength light.

In rendering, we are concerned with collections of many molecules. Light
interactions with such aggregates does not necessarily resemble interactions
with isolated molecules. Waves scattered from nearby molecules is often mutually
coherent, and thus exhibit interference, since they originate from the same
incoming wave.

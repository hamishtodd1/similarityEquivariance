# Similarity-equivariant neural networks
This is WIP fork of [Clifford Group Equivariant Networks (NeurIPS 2023 Oral)](https://arxiv.org/abs/2305.11141), focussed on rotation/translation/scale equivariance. This research was done at the Lasenby Lab in the Cambridge engineering department and is a joint work with https://github.com/YuxinYao620

## Scaling/rotation/translation with Geometric Algebra?
This is built on a base of 2D Conformal Geometric Algebra BUT we are forcibly restricting to a subset of that - ordinary CGA has lots of stuff that is *not* interesting for computer vision, such as circle inversions, vortex pairs, and hyperbolic translations, because they are not the sorts of transformations that occur with cameras.

2D CGA is Cl(3,1) and has 16 basis k-vectors:

$ 1, e_1, e_2, e_3, e_4, e_{12}, e_{13}, e_{14}, e_{23}, e_{24}, e_{34}, e_{123}, e_{124}, e_{134}, e_{234}, e_{1234}$

People sometimes say that CGA is the algebra of rotations, translations, and scalings. **This is diabolically incorrect**, again CGA has circle inversions and hyperbolic translations - yes, it does have rotations/translations/scalings but they make up literally 0% of its full set of transformations.

So. The system we use here to accomplish the thing CGA says it accomplishes has exactly 12 basis vectors:

$ 1, e_1, e_2, e_0, e_{12}, e_{10}, e_{20}, e_{34}, e_{120}, e_{134}, e_{234}, e_{1234}$

Where:

$e_0 := e_3+e_4$

$e_{10} := e_{13} + e_{14}$

$e_{20} := e_{23} + e_{24}$

$e_{120} := e_{123} + e_{124}$

If you have a heavily trained eye (get in touch!) you will spy the embedding of Projective GA in CGA, but it's been fused with some other stuff descending from $e_{34}$ - and indeed, that's the part that does scalings (and reflection-scalings, and rotation-scalings).

12 is a disturbingly unusual number of basis k-vectors. But note that it is a subgroup of CGA - in fact we can go further, it is a subalgebra. You can think of it as the set of all transformations that leave the plane at infinity where it is. Conceptually you could say that it is a system with enough "basis directions" to let you do these transformations, where $e_{34}$ is somehow a basis direction in "scale space".

## Why would you put this in a neural network?
The general justification for the success of convolutional NNs is that it shouldn't matter where, in an image, an object appears - so it's helpful (eg: makes your data go further) to "bake in" translation invariance. You do this by having your architecture be based on convolution kernels

The idea with scale invariance is that distance-from-the-camera causes the apparent size of objects to change (see eg Scale-equivariant steerable networks by Sosnovik et al). Therefore, it's reasonable to expect that scale equivariance can offer the same sort of improvement as translation equivariance.

Problem: how do you combine scale invariance with rotation and translation equivariance?

You could in principle do the same thing by representing the scale/rotation/translation with a 3x2 matrices. But matrices are too free - they allow for arbitrary scalings (getting taller while not getting wider) rather than just uniform scalings (you get taller and wider at the same rate). The algebra described above is scaling/translation/rotation *and that's mathematically guaranteed to be it*. Perhaps reflection (and transflection, and... scale-flection?) too; I want to make that optional.

The general use of this tool will be image classification and generation, especially for GIS which has rotation equivariance (other images are more sensetive to rotation). Note that while images are in some strong sense scale-equivariant (a photograph is of the same thing no matter how far away you are holding it from you!) the real world/laws of physics are usually *not* scale-invariant. Adults are, anatomically, *not* just scaled-up toddlers. For that reason, you would not expect this framework to improve a simulation of fluid/weather/traffic. There are some interesting exceptions to that: magnetostatics is scale invariant. On the other hand if you wanted to simulate magnetostatics, you should probably use the whole of CGA since magnetostatics is invariant to the whole of it (sphere inversions, hyperbolic translations...).

## Geometric aspects
Alright now's the time to name this thing, I'm calling it Similarity Geometric Algebra, because rotation/translation/scale/reflection and all their combos are "similarities".

In PGA people are quick to point out the beautiful fact that rotations are always rotations around points (in 2D) and a translation is a rotation around a point at infinity. I see this and I raise you: a uniform scaling is always towards a point; and a scaling toward a point at infinity is a translation.

It turns out that there are two kinds of points in 2D Similarity GA: rotation-points like e12, and scaling-points, like e34 - both those examples are points at the origin. One of them you can imagine with an arrow going around it, one with an arrow towards it. This turns out to perfectly realize Leo Dorst's idea of oriented geometry. Yes, they are dual, in the ordinary CGA sense.

What about the trivectors? They're lines (as are the 1-vectors). But there not reflection-lines, they're actually involved in "scale-reflection" transforms. For example, 1.25e1 + 0.75e134 = e1(1.25+0.75e34) is a scaling toward e34 (the part in brackets) followed by a reflection in e1. Alright, how about 1.25e1 + 2.5e0 + 0.75e134? That's also a scale-reflection, but it is a reflection in the line e1+2e0, followed by a scaling towards a different point. In general, when e134 is the trivector part, the scaling will be towards a point lying on the line e2, which is dual to e134. Consequently, if you have e134 on its own (though that makes no sense from a transformations point of view), it's sort of a scaling-line - it's a line that is made up of lots of little scaling-points.

## Projective transformation (will not be found here!)
In addition to distance from a camera, the *angle* from the camera of an object can change too, changing its appearance. This brings in a wider class of transformations called projective transformations. However, it seems to me that projective transformations involve a degree of "revealing something in an object which you couldn't originally see in the picture." Maybe that's fine for neural networks
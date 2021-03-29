---
title: "Stein Points: Narrowing Manifolds"
layout: full-width
categories: stein-points
desc: We can probably extend directional Stein points to SO(3), SE(3), and SPD matrices
---

<p class="subtitle">Directional Stein Points</p> 
{{site.desc}}
<!--more-->

## Stein Points over $$SO(3)$$

When defined via unit quaternions, points in $$SO(3)$$ form points on the hypersphere. So, if we describe our points in this quaternion co-ordinate system, we should be able to directly use dKSD to define Stein Points on $$SO(3)$$. For example, this [blog](https://marc-b-reynolds.github.io/quaternions/2017/11/10/AveRandomRot.html) gives a derivation of the volume form for $$SO(3)$$, which is the same as that for the hypersphere.

## Stein Points over $$SE(3)$$

The $$SE(3)$$ manifold is formed via the cartesian product of $$\mathbb{R}^3$$ and $$SO(3)$$ i.e. concatenate the vectors $$[x,y,z]$$ and the components of a unit quaternion. Most methods in Riemannian methods (e.g. exp and log maps) smoothly carry over to such cartesian-product-manifolds. For example, the exp map for $$SE(3)$$ is the direct product of the exp maps for $$\mathbb{R}^3$$ and $$SO(3)$$. Can we do the same with the KSD? Can we construct the KSD for SE(3) by somehow forming the cartesian product of the Euclidean KSD and the directional KSD (for $$SO(3)$$)?

Firstly, for this approach to make sense, we need to be able to formulate an RKHS for $$SE(3)$$ by combining the RKHS for $$\mathbb{R}^3$$ and $$SO(3)$$. Fortunately, it seems like the direct-product of two RKHS is also an RKHS.

## Stein Points over SPD Matrices

**Q: Does the SPD manifold have zero boundary?**

Recall, that the derivation of the directional KSD relied on the fact that the hyperspherical manifold has no boundary. Specifically, Xu and Matsuda (2020) use the following fact
$$
\int_{\mathcal{S}^{d-1}} d\omega = \int_{\delta S^{d-1}} \omega = 0
$$
SPD matrices form the interior of a pointed convex cone. Its boundary is the set of singular positive semidefinite matrices (for example, as mentioned [here](http://indico.ictp.it/event/a08167/session/124/contribution/85/material/0/0.pdf)). We know that singular matrices form a zero measure subset of the space of all matrices. Can we use this fact to show that the above integral goes to $$0$$? If so, we might be able to use the same formula for the dKSD in this case as well -- just with a different coordinate system and volume element.

**Q: What is the volume element for SPD matrices?**

[Equation 13 in this paper](https://hal.archives-ouvertes.fr/hal-01248573/document) gives an expression for the volume element for SPD matrices. It requires the determinant of the matrix, so it might be computationally expensive. 

**Q: Why not just work with the Cholesky decomposition directly?**

Generally, we just work with SPD matrices in a Cholesky factorization to ensure constraints. In this case, it is not clear if we can do that. What would the volume element be in that case? Would the boundary of the manifold (of arbitrary square matrices) vanish as we need?


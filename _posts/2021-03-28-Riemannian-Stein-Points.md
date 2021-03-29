---
title: "Riemannian Stein Points"
layout: full-width
categories: stein-points
desc: Background on Stein Points
---

<p class="subtitle">Extending Stein points to Riemannian manifolds</p> 
<!--more-->


The Stein Points (SP) algorithm is a deterministic algorithm that can draw samples from a distribution, when one has access to the score function (gradient of the log of the probability density) of a probability distribution. Since it only depends on the score function, SP is especially useful when we only have access to the normalized probability density -- for instance, we might have a likelihood but not the partition function needed to normalize it. Compared to the most popular method used to sample in this setting (i.e. MCMC sampling), SP avoids the problem of positive auto-correlation between samples by inducing a repulsive force the sample generated during an iteration, and all previously generated samples. SP is closely related to Stein Variational Gradient Descent (SVGD), which can also be used to sample using score functions, and also enforces repulsion between samples. 

### Stein Operators

The SP algorithm generates new samples by minimizing the kernel Stein discrepency (KSD) between the generated samples and the target distribution. The central object of the KSD is the Stein operator. Let us consider our domain $$\mathcal{X}$$ to be a smooth manifold. Let $$T_p$$ be a linear differential operator acting on $$\mathcal{X}$$; note that $$T_p$$ is defined with respect to our target distribution $$P$$. The operator $$T_p$$ is called a *Stein operator* with a corresponding set of functions $$\mathcal{F}$$ called its *Stein set* if it satisfies the following property

$$
\begin{align*}
\int T_p[f]~dP = 0~~\forall f \in \mathcal{F}
\end{align*}
$$

This equation tells us that the Stein operator $$T_p$$ is a special kind of operator such that when you apply it to any function $$f \in \mathcal{F}$$, it results in another function $$T_p[f]$$ that as expectation value $$0$$ under the distribution $$P$$. Note that this is not the case for *all* possible functions $$f$$, but only functions that are within the Stein class of the operator $$T_p$$.

 For now, let's assume that this Stein class covers all the functions we are interested in, and is thus not a restriction at all. In this case, what does the Stein operator buy us? When computing the discrepency between a set of samples (its empirical distribution) and a target distribution, we have to sometimes deal with some pesky expectation values that can be intractable to compute. Using the Stein operator, we can simply zero out these expectations and simplify calculations.

The Maximum Mean Discrepency (MMD) between set of samples $$\{x_i\}_{i=1}^n$$ and a probability distribution $$P$$ is calculated as follows

$$
\begin{align*}
MMD(\{x_i\}_{i=1}^n, P) &= \left|\left| \frac{1}{n}\sum_{i=1}^n k(x_i, \cdot) - \int k(x,\cdot)dP(x)\right|\right|_{\mathcal{K}}
\end{align*}
$$

where $$k$$ is a reproducing kernel, and $$\mathcal{K}$$ is its reproducing kernel Hilbert space (RKHS). The MMD is thus just the difference between the empirical estimate of the kernel mean embedding of our samples $$\{x_i\}$$ and the popularion kernel mean embedding for the distribution $$P$$.  If we expand the above equation further, we get the following

$$
\begin{align*}
\text{MMD}(\{x\}_i, P) = \mathbb{E}_{x \sim P,~x' \sim P}\left[k(x,x')\right] - \frac{2}{n} \mathbb{E}_{x \sim P} \sum_{i=1}^n k (x,x_i) + \frac{1}{n^2} \sum_{i,j=1}^n k(x_i, x_j)
\end{align*}
$$

### Kernel Stein Discrepency

Now, remember our task: we're given (the score function of ) a probability distribution $$P$$ and want to generate some samples from it. Once we pick a kernel $$k$$, the last term in the above sum is easy enough to calculate. But the first two terms are not so easy: we need to compute expectations over the distribution $$P$$. Here's what might make our lives easier: say we can somehow pick a kernel $$k_0$$ such that $$\mathbb{E}_{x \sim P}(k_0(x,x')) = 0~~\forall x \in \mathcal{X}$$. With this kernel, the first two terms in the above sum become $$0$$ and we're just left with the last term, which we know how to compute! It turns out that we can construct such a kernel $$k_0$$ using a Stein operator
$$
\begin{align*}
k_0 (x,x') = <T_p k (x,\cdot)~,~T_p k (\cdot, x') >
\end{align*}
$$
This new kernel $$k_0$$ is called a *Stein Reproducing Kernel*. This kernel is defined with respect to a particular Stein operator, which in turn is defined with respect to the target distribution $$P$$.  Using this kernel, the MMD reduces to the following discrepency also called the *kernel Stein discrepency* (KSD)

$$
\begin{align*}
KSD(\{x_i\}_{i=1}^n~,~P) = \sqrt{\frac{1}{n^2}\sum_{i,j=1}^n k_0(x_i,x_j)}
\end{align*}
$$

### The Stein Points Algorithm

The SP algorithm simply finds the set of samples $$\{x_i\}$$ that minimize this KSD. Here's one way to think about what's going on: when we pick a reproducing kernel $$k$$, we're defining a map from our domain $$\mathcal{X}$$ to its RKHS $$\mathcal{K}$$. If we were to compute discrepency between distributions in this RKHS, we'd run into problems of having to evaluate the expectations values we saw earlier. So, we take our target distribution $$P$$, and come up with a Stein operator $$T_p$$ with respect to it. This Stein operator takes elements in our original RKHS $$\mathcal{K}$$ and maps them to a different hilbert space where these pesky expectation values become $$0$$ and the discrepency has a simple formula that only depends on elements in our sample set. 

The SP paper proposes two algorithms to minimize the KSD. The first is a *herding* algorithm, which is equivalent to the Frank-Wolfe or conditional-gradient algorithm. The second one is a *greedy* approach, which essentially adds regularization term to the objective of the herding algorithm

__Herding__

The herding approach to SP is quite simple. If we want to generate $$n$$ samples, we just repeat the following optimization $$n$$ times

$$
\begin{align*}
x_n \in \text{argmin}_{x \in \mathcal{X}} \sum_{i=1}^{n-1} k_0(x_i,x)
\end{align*}
$$

We can look at the above update scheme as a particular instance of kernel herding. In the kernel herding algorithm, we try to find new samples that are as close as possible to a target kernel mean embedding while simultaneously being far away from all the samples that have been generated thus far. This second repulsive force is identical to the objective being optimized at every step of the SP herding approach. In the case of SP, we've basically used our Stein operator to map the target mean embedding to $$0$$ in a different RKHS. So we're trying to herd new points close to this $$0$$ mean embedding while enforcing repulsion to promote diversity in our generated samples.

__Greedy__

The greedy SP algorithm just adds a regularization term to the herding objective at every iteration

$$
\begin{align*}
x_n \in \text{argmin}_{x \in \mathcal{X}}~~\frac{1}{2} k_0(x,x) +  \sum_{i=1}^{n-1} k_0(x_i,x)
\end{align*}
$$

### The Langevin Stein Operator

We now have the recipe for the SP algorithm -- all that's left is to figure how to compute the reproducing Stein kernel $$k_0$$. Recall that this depends on two things. First, we need a base kernel (e.g. the RBF kernel). Second, we need to pick a Stein operator for our target distribution $$P$$, with probability density $$p$$. The most common Stein operator (also used in SVGD) is the *Langevin Stein operator*

$$
\begin{align*}
T_p f = \frac{\texttt{div}(pf)}{p}
\end{align*}
$$

where $$\text{div}$$ is the divergence operator. This Stein operator leads to the following Stein repoducing kernel

$$
\begin{align*}
k_0(x,x') =~~&\texttt{div}_x \left(\nabla_{x'} k(x,x')\right) \\
& + \nabla_x k(x,x')\cdot \nabla_{x'} \log p(x')\\
& + \nabla_{x'} k(x,x') \cdot \nabla \log p(x)\\
& + k(x,x')\nabla_x \log(p(x)) \cdot \nabla_{x'} \log p(x')
\end{align*}
$$

Note that $$k_0$$ depends on $$p$$ only through its score function $$\nabla_x \log p(x)$$. 
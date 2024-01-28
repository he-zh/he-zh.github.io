---
layout: post
title:  Notes for learning bounds
date:   2023-12-25
description: a short overview on techniques to prove learning bounds
tags: learning-theory note
categories: basics
related_posts: true
giscus_comments: false
toc:
  beginning: true


# toc:
#   - name: Overview
#   - name: Bound the estimation error of ERM
#     subsections:
#       - name: Covering numbers
#       - name: Rademacher complexity
#       - name: VC dimension
#   - name: Bound the approximation error
#     subsections:
#       - name: Non-uniform bound
#       - name: Structural risk minimization
#   - name: Is ERM enough?
#   - name: Stability based bound
#     subsections:
#       - name: Regularized loss minimization

---





# Overview

In the heart of machine learning lies a fundamental question: 
**can we trust the predictions of our models beyond the data we trained them on?** 
This is where the analysis of generalization error comes into play.
This note delves into various techniques that offer guarantees on  how well our models generalize.
We'll introduce concepts like VC dimension, covering numbers, Rademacher complexity, non-uniform learning, and stability bound.


Central to this analysis lies the concept of excess error, a measure of how much a model's performance deviates from its ideal, true performance. But this excess error isn't a monolithic entity; it's a captivating dance between two key players: approximation error and estimation error.

$$\underbrace{L_D(\hat h_S) - L_{Bayes}}_\text{excess error} =  \underbrace{L_D(\hat h_S) - L_D(h^*)}_\text{estimation error}
+  \underbrace{L_D(h^*)  - L_{Bayes}}_\text{approximation error}, 
$$

where $$\hat h_S \in \arg\min_{h\in\mathcal{H}} L_S(h)$$, $$h^* = \arg\inf_{h\in\mathcal{H}} L_D(h)$$, $$L_S(\cdot)$$ is the empirical risk and $$L_D(\cdot)$$ is the expected risk.

- **Excess Error:** $$L_D(\hat h_S) - L_{Bayes}$$ is the difference between the error of a model and the Bayes error (Bayes risk or irreducible error) on a given task. 
- **Bayes Error:** $$L_{Bayes}$$ represents the lowest achievable error rate for any classifier on that task.
- The excess error in Empirical Risk Minimization (ERM) can be effectively bounded by dissecting it into two primary components: estimation error and approximation error.
- **Estimation Error:** $$L_D(\hat h_S) - L_D(h^*)$$ arises from using our algorithm $$\hat h_S$$ instead of selecting the best predictor $$h^*$$ within the hypothesis space $$\mathcal{H}$$. As the sample size m approaches infinity, the estimation error ideally tends toward zero.
- **Approximation Error:** $$L_D(h^*)  - L_{Bayes}$$ is incurred by choosing the optimal predictor within $$\mathcal{H}$$ rather than utilizing the optimal classifier (Bayes classifier) from any hypothesis.


# Uniform convergence
Bounding the estimation error of Empirical Risk Minimization (ERM) involves controlling the generalization gap between the empirical risk and the expected risk.
The estimation error could be written as 

$$L_D(\hat h_S) - L_D(h^*) \le L_D(\hat h_S) - L_S(\hat h_S) + L_S(h^*) - L_D(h^*) $$

and we can probabilistically bound Generalization gap for $$h^*$$ and $$\hat h_S$$.


**Upper bound $$L_S(h^*) - L_D(h^*)$$:** Hoeffiding's inequality

Let $$L_S(h^*)$$ be an average of iid random variables and $$L_D(h^*)$$ be its expectation, we have 

$$L_S(h^*) - L_D(h^*) = \frac{1}{m}\sum_{i=1}^{m} \ell(h,z_i) - \mathbb{E}_{z\sim D} \ell (h,z),$$

If we further assume that $$\ell(h,z)\in [a, b]$$ for all $$h, z$$, we could apply <a href="https://en.wikipedia.org/wiki/Hoeffding%27s_inequality">Hoeffding's inequality</a> to get

$$\mathrm{Pr}(L_S(h^*) - L_D(h^*) \le (b-a) \sqrt{\frac{\log 1/\delta}{2m}} ) \ge 1-\delta $$

- For the assumption that $$\ell(h,z)\in [a, b]$$
	- bounded loss naturally satisfies
		- 0-1 loss, ramp loss
	- unbounded loss might be depending on $$H$$ and $$D$$
		- square loss, logistic loss, hinge loss...


**Upper bound $$L_D(\hat h_S) - L_S(\hat h_S)$$:** uniform convergence

We can't apply <a href="https://en.wikipedia.org/wiki/Hoeffding%27s_inequality">Hoeffding's inequality</a> directly on the generalization gap of $$\hat h_S$$ as $$\ell(\hat h_S, z_i)$$ aren’t independent. The choice of $$\hat h_S$$ depends on all of $$S$$, i.e. on all of the other $$z_j$$ , as well as the ones we’re evaluating on.

**Uniform convergence.**
The basic idea is that if we know that $$L_D(h) - L_S(h)$$ is small for all $$h\in\mathcal{H}$$, then it’ll be small for $$\hat h_S$$ 

$$ L_D(\hat h_S) - L_S(\hat h_S)  \le \sup_{h\in\mathcal{H}} L_D(h) - L_S(h) ,$$

and bound the $$\sup_{h\in\mathcal{H}} L_D(h) - L_S(h)$$ instead.

Here are a few commonly used techniques:
- Finite $$\vert \mathcal{H}\vert <\infty$$
	- we could use the union bound
		- but $$\vert \mathcal{H}\vert$$ might be really large which makes the bound vacuous
- Infinite $$\vert \mathcal{H}\vert =\infty$$
	- covering numbers
	- Rademacher complexity
	- VC dimension

<hr>
## Finite hypothesis class $$|\mathcal{H}|<\infty$$
With probability at least $$1-\delta$$, we have

$$\begin{align*}L_D(\hat h_S) - L_D(h^*) 
&\le  \sup_{h\in\mathcal{H}}[ L_D(h) - L_S(h)] + L_S(h^*) - L_D(h^*) \\
&\le (b-a) \sqrt{\frac{2}{m}\log \frac{|H|+1}{\delta}}
\end{align*}$$

{% details Proof for the finite hypothesis class bound%}
We start by noting 

$$\sup_{h\in\mathcal{H}} L_D(h) - L_S(h) \le \varepsilon \leftrightarrow  \forall h\in\mathcal{H}. \ L_D(h) - L_S(h) \le \varepsilon$$

Use union bound $$P(A+B) \le P(A) + P(B)$$ we get

$$ \mathrm{Pr}_{S\sim D^m}(\exists  h\in\mathcal{H}. \ L_S(h) - L_D(h) > \varepsilon) \le \sum_{h\in\mathcal{H}} P_{S\sim D^m} (L_S(h) - L_D(h) > \varepsilon) $$

Let
$$\mathrm{Pr}_{S\sim D^m} (L_S(h) - L_D(h) > \varepsilon) \le \frac{\delta}{|H|+1}$$ 
for 
$$\forall h \in\mathcal{H}$$, where $$\varepsilon=(b-a) \sqrt{\frac{1}{{2m}}\log \frac{|H|+1}{\delta}}$$ then 

$$\begin{align*} \mathrm{Pr}_{S\sim D^m}(\exists h\in\mathcal{H}. \ L_S(h) - L_D(h) > \varepsilon) \le \frac{|\mathcal{H}|\delta}{|\mathcal{H}|+1}\\
\mathrm{Pr}_{S\sim D^m}(L_S(h^*) - L_D(h^*) > \varepsilon) \le \frac{\delta}{|\mathcal{H}|+1}\end{align*}$$

Thus, we have 

$$\mathrm{Pr}_{S\sim D^m}\left((\sup_{h\in\mathcal{H}} L_S(h) - L_D(h) \le \varepsilon) \bigcap (L_S(h^*) - L_D(h^*) \le \varepsilon)\right) \ge 1- \delta,$$

which means with probability at least $$1-\delta$$, the following inequality holds 

$$ \sup_{h\in\mathcal{H}}[ L_S(h) - L_D(h)] + L_S(h^*) - L_D(h^*) \le 2\varepsilon $$
{% enddetails %}
<br>
<hr>

## Infinite hypothesis class $$|\mathcal{H}|=\infty$$

### Covering numbers
We can rewrite the uniform bound like this 

$$\begin{align*}
&\sup_{h\in\mathcal{H}} L_D(h) - L_S(h) \\
&= \sup_{h\in\mathcal{H}} L_D(h) - L_D(t) + L_D(t) - L_S(t) + L_S(t)- L_S(h) \\
&\le \sup_{h\in\mathcal{H}} [L_D(h) - L_D(t)] + \sup_{t\in\mathcal{T}} [L_D(t) - L_D(t)] + \sup_{h\in\mathcal{H}} [L_S(t)- L_S(h)] \end{align*}$$

where $$\mathcal{T}$$ is a finite $$ρ$$-cover set of $$\mathcal{H}$$.

**Definition ($$\rho$$-cover set).**
A $$ρ$$-cover of a set $$\mathcal{H}$$ is a set $$\mathcal{T}⊆\mathcal{H}$$ such that, for all $$h∈\mathcal{H}$$, there is a $$t ∈ \mathcal{T}$$ with $$dist(t, h) ≤ ρ$$.
The smallest size of the $$\rho$$-cover set of $$\mathcal{H}$$ is $$N(\mathcal{H}, \rho)$$.

**Bounds with $$\rho$$-covering**

- To bound the first and the third term, we need to assume <a href="https://en.wikipedia.org/wiki/Lipschitz_continuity">Lipschitz continuity</a> of loss that $$\vert \ell(h,z) - \ell(t,z)\vert \le K \Vert {h(x)-t(x)}\Vert$$. According to the assumption of $$ρ$$-cover, we also have $$\Vert{h-t}\Vert \le \rho$$. Thus, we can bound the differences respectively.
- To bound the second term, as $$\mathcal{T}$$ is finite set, we could apply <a href="https://en.wikipedia.org/wiki/Hoeffding%27s_inequality">Hoeffding's inequality</a> as the finite hypothesis class case. The only problem is how to decide the size of $$\mathcal{T}$$. 

{% details **Example:** logistic regression with bounded $$X$$ and bounded $$\mathcal{H}$$%}
- Input space $$Z = X \times Y, X = \mathbb{R}^d, \Vert x\Vert \le C, Y=\{-1,1\}$$
- Hypothesis class $$\mathcal{H}=\{x\to w\cdot x : w\in \mathbb{R}^d, \Vert w\Vert \le B\}$$
- Logistic loss $$\ell_{\rm{log}} (h,z) =l_y(h(x)) = \log(1+\exp(-y h(x)))$$, which is 1-Lipschitz. And we also have $$\vert \ell_{\rm{log}} (h,z)\vert \le |1+h(x)\vert \le 1+BC$$ 
According to Lipschitz continuity, we have $$\vert \ell(h,z) - \ell(t,z)\vert  \le \Vert{w\cdot x - t\cdot x}\Vert \le \Vert w-t\Vert \Vert x\Vert = \rho C$$. Thus, 

$$\begin{align*} \sup_{h\in\mathcal{H}} [L_D(h) - L_D(t)] \le \rho C \\
\sup_{h\in\mathcal{H}} [L_S(h) - L_S(t)] \le \rho C \end{align*}$$

Besides, with probability at least $$1-\delta$$, we have 

$$\sup_{h\in\mathcal{T}} L_D(t) - L_S(t) \le (1+BC) \sqrt{\frac{1}{2m}\log\frac{N(\mathcal{H},\rho)}{\delta}} \le \frac{1+BC}{\sqrt{2m}} \left(\sqrt{\log\frac{1}{\delta}} + \sqrt{\log N(\mathcal{H},\rho)}\right)$$

**The overall bound on estimation error of logistic regression:** from the above, we know that 

$$\begin{align*}
&\mathrm{Pr}\left(\sup_{h\in\mathcal{H}} [L_D(h) - L_S(h)] > 2\rho C + \frac{1+BC}{\sqrt{2m}} (\sqrt{\log\frac{1}{\delta}} + \sqrt{\log N(\mathcal{H},\rho)}) \right) 
< \delta \\
&\mathrm{Pr}\left(L_S(h^*) - L_D(h^*) > \frac{1+BC}{\sqrt{2m}} \sqrt{\log\frac{1}{\delta}} \right) < \delta
\end{align*}$$

Thus, with probability at least $$1-\delta$$ the following holds 

$$\begin{align*}
L_D(\hat h_S) - L_D(h^*) &\le \sup_{h\in\mathcal{H}} [L_D(h) - L_S(h)] + L_S(h^*) - L_D(h^*)\\ 
&\le 2\rho C + \frac{1+BC}{\sqrt{2m}} (2\sqrt{\log\frac{2}{\delta}} + \sqrt{\log N(\mathcal{H},\rho)})
\end{align*}$$ 

where $$\rho$$ can be further optimized to get rid of $$\rho$$ and obtain a tighter bound.
{% enddetails %}
<br>


<hr>

### Rademacher complexity

- Rademacher complexity is typically applied on a function class of models that are used for **classification**, with the goal of measuring their ability to classify points drawn from a probability space under arbitrary labellings. 
- When the function class is rich enough, it contains functions that can appropriately adapt for each arrangement of labels, simulated by the random draw of $$\sigma_i$$ under the expectation, so that this quantity in the sum is maximized.
- measures richness of a class of real-valued functions with respect to a probability distribution.

**Definition (Rademacher complexity).**
The Rademacher complexity of a set $$V ⊆ \mathbb{R}^m$$ is given by 
$$\mathrm{Rad}(V) = \mathbb{E}_{\sigma\sim\rm{Unif}(\pm1)^m} \sup_{v\in V} \frac{\sigma \cdot v}{m}$$

**Relation between the mean worst-case generalization gap and Rademacher complexity**

$$\mathbb{E}_{S\sim D^m} \sup_{h\in\mathcal{H}} L_D(h) - L_S(h) = 2 \mathbb{E}_{S\sim D^m} \mathrm{Rad}((\ell \circ \mathcal{H})\vert_S)$$

where $$(\ell \circ \mathcal{H})\vert_S = \{(\ell(h,z_1), \dots, \ell(h,z_m)): h\in \mathcal{H}\}$$

{% details Proof for the equation%}

$$\begin{align*}\mathbb{E}_{S\sim D^m} \sup_{h\in\mathcal{H}} L_D(h) - L_S(h) 
&=\mathbb{E}_{S\sim D^m} \sup_{h\in\mathcal{H}} [\mathbb{E}_{S'\sim D^m}[L_{S'}(h)] - L_S(h)]\\
&\le \mathbb{E}_{S,S'\sim D^m} \sup_{h\in\mathcal{H}} [L_{S'}(h) - L_S(h)] \\
&= \mathbb{E}_{S,S'\sim D^m} \sup_{h\in\mathcal{H}} \frac 1m \sum_{i=1}^m [\ell(h, z_i) - \ell(h, z_i')]\end{align*}
$$

Let $$\sigma_i \in [-1,1]$$ for $$i \in [m]$$, $$P(\sigma_i=1)= P(\sigma_i=-1)= \frac12$$, and $$\vec \sigma = (\sigma_1, \dots, \sigma_m)$$. Let $$(u_i, u_i')=\begin{cases}(z_i, z_i'), &\text{if } \sigma_i=1 \\ (z_i', z_i), &\text{if } \sigma_i=-1 \end{cases}$$. So, for any value of $$S, S',$$ and $$\vec \sigma$$, defining $$U=(u_1, \dots, u_m)$$ and $$U'=(u_1', \dots, u_m')$$ accordingly, we have $$\ell(h, z_i) - \ell(h, z_i')=\sigma_i (\ell(h, u_i) - \ell(h, u_i'))$$. Thus,

$$\begin{align*}
\mathbb{E}_{S,S} \sup_{h\in\mathcal{H}} \frac 1m \sum_{i=1}^m [\ell(h, z_i) - \ell(h, z_i')]
&=\mathbb{E}_{S, S'} \mathbb{E}_{\vec \sigma} \mathbb{E}_{U,U'} [\sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m \sigma_i (\ell(h, u_i) - \ell(h, u_i')) \vert S,S',\vec \sigma]\\
&=\mathbb{E}_{U, U'} \mathbb{E}_{\vec \sigma} \mathbb{E}_{S,S'} [\sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m \sigma_i (\ell(h, u_i) - \ell(h, u_i')) \vert U,U',\vec \sigma]\\
&=\mathbb{E}_{U, U'} \mathbb{E}_{\vec \sigma}[\sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m \sigma_i (\ell(h, u_i) - \ell(h, u_i'))]\\
&=\mathbb{E}_{S, S'} \mathbb{E}_{\vec \sigma}[\sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m \sigma_i (\ell(h, z_i) - \ell(h, z_i'))]\\
&\le \mathbb{E}_{S} \mathbb{E}_{\vec \sigma} \sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m \sigma_i \ell(h, z_i)+\mathbb{E}_{S'} \mathbb{E}_{\vec \sigma} \sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m -\sigma_i \ell(h, z_i')\\
&= 2 \mathbb{E}_{S} \mathbb{E}_{\vec \sigma} \sup_{h\in\mathcal{H}}\frac{1}{m} \sum_{i=1}^m \sigma_i \ell(h, z_i)\\
&= 2 \mathbb{E}_{S\sim D^m} \mathrm{Rad}((\ell \circ \mathcal{H})|_S) \end{align*}$$

{% enddetails %}
<br>



**Compute $$\rm{Rad}((\ell \circ \mathcal{H})\vert_S)$$**

**Talagrand's contraction lemma.** Let $$\phi : \mathbb{R}^m → \mathbb{R}^m$$ be given by $$\phi(t) = (φ_1(t_1), \dots, φ_m(t_m))$$, where each $$φ_i$$ is $$ρ$$-Lipschitz (<a href="https://en.wikipedia.org/wiki/Lipschitz_continuity">Lipschitz continuity</a>). Then

$$\mathrm{Rad}(\phi \circ V) = \mathrm{Rad}(\{\phi(v): v\in V\}) \le \rho \mathrm{Rad}(V)$$

Thus, if $\ell$ is $$ρ$$-Lipschitz, we have 

$$\rm{Rad}((\ell \circ \mathcal{H})\vert_S) \le \rho \rm{Rad}(\mathcal{H}\vert_S) $$

{% details **Example:** logistic regression with bounded $$\mathcal{H}$$%}

- Input space $$Z = X \times Y, X = \mathbb{R}^d, Y=\{-1,1\}$$
- Hypothesis class $$\mathcal{H}=\{x\to w\cdot x : w\in \mathbb{R}^d, \Vert w\Vert \le B\}$$
- Logistic loss $$\ell_{\rm{log}} (h,z) =l_y(h(x)) = \log(1+\exp(-y h(x)))$$, which is 1-Lipschitz 

$$\begin{align*} 
\mathrm{Rad} ((l_y\circ \mathcal{H})\vert_S) 
&\le \mathrm{Rad} (\mathcal{H}|_S)\\
&= \mathbb{E}_\sigma \sup_{\Vert w\Vert \le B} \frac{1}{m} \sum_{i=1}^m \sigma_i \left\langle w, x_i\right\rangle  \\
&= \frac{1}{m} \mathbb{E}_\sigma \sup_{\Vert w\Vert  \le B} \left\langle w, \sum_{i=1}^m \sigma_i x_i\right\rangle \\
&\le \frac{B}{m} \mathbb{E}_\sigma \Vert \sum_{i=1}^m \sigma_i x_i\Vert  \\
&\le \frac{B}{m} \sqrt{\mathbb{E}_\sigma \Vert \sum_{i=1}^m \sigma_i x_i\Vert^2} \\
&= \frac{B}{m} \sqrt{\mathbb{E}_\sigma \sum_{i,j} \left\langle \sigma_i x_i, \sigma_j x_j \right\rangle }\\
&= \frac{B}{m} \sqrt{\mathbb{E}_\sigma \sum_{i} \Vert x_i\Vert^2 + \sum_{i\neq j} \mathbb{E}_\sigma \sigma_i \sigma_j \left\langle x_i, x_j \right\rangle }\\
&= \frac{B}{m}  \sqrt{\sum_{i=1}^m \Vert x_i\Vert^2}
\end{align*}$$

$$\mathbb{E}_S \mathrm{Rad} ((l_y\circ \mathcal{H})\vert_S) \le \frac{B }{\sqrt{m}} \mathbb{E}_S \sqrt{\frac{1}{m} \sum_{i=1}^m \Vert x_i\Vert^2} \le \frac{B }{\sqrt{m}}  \sqrt{\mathbb{E}_x \Vert x\Vert^2} $$
{% enddetails %}
<br>

{% details **Special case:** 0-1 loss of binary classification%}

- Input space $$Z = X \times Y, X = \mathbb{R}^d, Y=\{-1,1\}$$
- Hypothesis class  $$\mathcal{H}=\{x\to h(x) : h(x)\in \{-1, 1\}\}$$
- 0-1 loss $$\ell_{\rm{0-1}} (h,z) =l_y(h(x)) = \mathbb{1}_{h(x)\neq y}$$
	- 0-1 loss is not a function on $$\mathbb{R}$$, so applying Talagrand’s lemma is a little weird. For computing the loss, we can just extend the function $$l_y$$ to $$\mathbb{R}$$ in any way at all, and the loss will be exactly the same.
	- So we could pick a $$\frac12-$$Lipschitz function. According to Talagrand's lemma, we have $$\mathrm{Rad}(\ell_{0-1} \circ \mathcal{H}_{\pm1}) \le \frac12 \mathrm{Rad}(\mathcal{H}_{\pm1})$$

$$\mathcal{H}\vert_S \subseteq \{-1,1\}^m$$ is finite, so we could bound the Rademacher complexity of this finite set based on its size. We have 

$$\mathrm{Rad}(\mathcal{H}_{\pm1}\vert_S) \le \sqrt{\frac{2}{m} \log \vert\mathcal{H}_{\pm1}\vert_S\vert}$$ 

>Growth function \eqref{eq:growth-function} provides the bound of $$\vert \mathcal{H}_{\pm1}\vert_S\vert$$.

*Extension: for binary classifiers to {0, 1}, $$\mathrm{Rad}(\mathcal{H}_{0,1}\vert_S)=\mathrm{Rad}(\frac12 (\mathcal{H}_{\pm1} + 1)\vert_S)=\frac12\mathrm{Rad}(\mathcal{H}_{\pm1}\vert_S)$$.*


We prove this by utilizing the following lemma and properties of sub-Gaussian distribution.
**Rademacher complexity of an arbitrary finite set.** If $$V=\{v: v\in\mathbb{R}^m\}$$ is finite, $$\Vert v\Vert  \le B$$ for all $$v \in V$$, then

$$\mathrm{Rad} (V)\le \frac{B}{m} \sqrt{2\log \vert V\vert}$$

*Proof.* We have  $$\mathrm{Rad}(V) = \mathbb{E}_{\sigma\sim\rm{Unif}(\pm1)^m} \sup_{v\in V} \sum_{i=1}^m \frac{\sigma_i v_i}{m}$$. According to [[Sub-Gaussian distribution#Sum]], $$\sigma_i \in SG(\frac{1-(-1)}{2})=SG(1)$$ and $$\frac{\sigma_i v_i}{m} \in SG(|v_i|/m)$$. As each $$v_i\sigma_i$$ is independent of each other, we have $$\sum_{i=1}^m \frac{\sigma_i v_i}{m} \in SG(\sqrt{\sum_i v_i^2} /m)=SG(\frac{\Vert v\Vert }{m}) \subseteq SG(\frac{B}{m})$$. 
According to <a href="https://en.wikipedia.org/wiki/Sub-Gaussian_distribution"> Sub-Gaussian distribution </a>, $$\sum_{i=1}^m \frac{\sigma_i v_i}{m}$$ is zero-mean random variables that satisfy

$$\mathbb{E}_\sigma \left[ \sup_{v_i\in V} \sum_{i=1}^m \frac{\sigma_i v_i}{m} \right] \le \frac{B}{m} \sqrt{2 \log(\vert V\vert)}.$$

Thus, $$\mathrm{Rad}(\mathcal{H}_{\pm1}\vert_S) \le \sqrt{\frac{2}{m} \log \vert\mathcal{H}_{\pm1}\vert_S\vert}$$
{% enddetails %}
<br>


**High probability bounds with Rademacher complexity** -- McDiarmid's inequality

We need to convert the average-case bound into a high probability bound using a concentration inequality - <a href="https://en.wikipedia.org/wiki/McDiarmid%27s_inequality">McDiarmid's inequality</a>, which is a generalized version of <a href="https://en.wikipedia.org/wiki/Hoeffding%27s_inequality">Hoeffding's inequality</a>. It could be used to bound the deviation between the sampled value and the expected value of certain functions (i.e., $$\sup_h [L_D(h)-L_S(h)]$$) when they are evaluated on independent random variables. 

**McDiarmid's inequality for the worst-case generalization gap.** Assume loss is bounded that $$\ell(h,z) \in [a,b]$$ for all $$h$$ and $$z$$. Let $$S=(z_1,\dots,z_i, \dots z_m)$$ and $$S^{(i)}=(z_1, \dots, z_i', \dots, z_m)$$. Then, 

$$\begin{align*}
\sup_{h\in\mathcal{H}} L_D(h)-L_S(h) &= \sup_{h\in\mathcal{H}}L_D(h) - L_{S^{(i)}}(h) + L_{S^{(i)}}(h) - L_S(h) \\
&\le \sup_{h\in\mathcal{H}} L_D(h) - L_{S^{(i)}}(h) + \sup_{h\in\mathcal{H}}L_{S^{(i)}}(h) - L_S(h) \\
&\le \sup_{h\in\mathcal{H}} L_D(h) - L_{S^{(i)}}(h) + \frac{1}{m}(b-a)
\end{align*}$$

We could substitute $$S$$ and $$S^{(i)}$$ without affecting the result, then the absolute value of difference is also bounded that $$\vert \sup_{h\in\mathcal{H}} [L_D(h)-L_S(h)] - \sup_{h\in\mathcal{H}} [L_D(h) - L_{S^{(i)}}(h)]\vert \le \frac{1}{m}(b-a)$$. Given the bounded differences property, we could apply McDiarmid's inequality directly. Thus, with probability at least $$1-\delta$$, we have

$$\begin{align*}\sup_{h\in\mathcal{H}} L_D(h)-L_S(h) 
&\le \mathbb{E}_{S\sim D^m} \sup_{h\in\mathcal{H}} L_D(h)-L_{S}(h) + (b-a)\sqrt{\frac{1}{2m}\log\frac{1}{\delta}}\\
&\le 2 \mathbb{E}_{S\sim D^m} \mathrm{Rad}((\ell \circ\mathcal{H})\vert_S) + (b-a)\sqrt{\frac{1}{2m}\log\frac{1}{\delta}}
\end{align*}$$


**The overall bound on estimation error**

$$\begin{align*}
&\mathrm{Pr}\left(\sup_{h\in\mathcal{H}} [L_D(h) - L_S(h)] > 2 \mathbb{E}_{S\sim D^m} \mathrm{Rad}((\ell \circ\mathcal{H})\vert_S) + (b-a)\sqrt{\frac{1}{2m}\log\frac{1}{\delta}} \right) 
< \delta \\
&\mathrm{Pr}\left(L_S(h^*) - L_D(h^*) > (b-a) \sqrt{\frac{1}{2m}\log\frac{1}{\delta}} \right) < \delta
\end{align*}$$


With probability at least $$1-\delta$$, the estimation error satisfy 

$$ \begin{align*}
L_D(\hat h_S) - L_D(h^*) &\le \sup_{h\in\mathcal{H}} [L_D(h)-L_S(h)]  + L_S(h^*) - L_D(h^*) \\
&\le 2 \mathbb{E}_{S\sim D^m} \mathrm{Rad}((\ell \circ\mathcal{H})|_S) + (b-a)\sqrt{\frac{2}{m}\log\frac{2}{\delta}}
\end{align*}$$


---


### VC dimension
- VC dimension is a measure of the size of a class of sets. The notion can be extended to classes of **binary** functions. 
- It is defined as the cardinality of the largest set of points that the algorithm can shatter, which means the algorithm can always learn a perfect classifier for any labeling of at least one configuration of those data points. 

The binary classification case in Rademacher complexity section have shown that for hypothesis $$\mathcal{H}_{\pm1}=\{x\to h(x) : h(x)\in \{-1, 1\}\}$$, we have $$\mathrm{Rad}(\mathcal{H}_{\pm1}\vert_S) \le \sqrt{\frac{2}{m} \log \vert\mathcal{H}_{\pm1}\vert_S\vert}$$. If we use zero-one loss, then $$\mathrm{Rad}(\ell_{0-1} \circ \mathcal{H}_{\pm1}) \le \frac12 \mathrm{Rad}(\mathcal{H}_{\pm1})$$. However, $$\mathrm{Rad}(\mathcal{H}_{\pm1}\vert_S)$$ depends on particular distribution $$S$$, which is not straightforward when taking expectation on it. So we need to upper bound it a little bit to make it look nicer *(not $$\vert \mathcal{H}\vert_S\vert \le \vert\mathcal{H}\vert$$, that would be too loose!)*

**Growth function.** The growth function $$Γ_\mathcal{H}(m)$$ of a hypothesis class $$\mathcal{H}$$ is given by 

\begin{equation}\label{eq:growth-function}\Gamma_\mathcal{H}(m)=\sup_{x_1, \dots, x_m\in \mathcal{X}}\vert \mathcal{H}\vert_{(x_1, \dots, x_m)}\vert \end{equation}

By definition, $$\vert\mathcal{H}\vert_S\vert \le Γ_\mathcal{H}(m)$$ for any $$S_x$$ of size $$m$$; thus for binary classifiers with zero-one loss, we immediately know that the expected worst-case generalization gap 

$$\mathbb{E}_{S\sim D^m} \sup_{h\in\mathcal{H}} L_D(h)-L_{S}(h) \le 2 \mathbb{E}_{S\sim D^m} \mathrm{Rad}((\ell_{0,1} \circ \mathcal{H})|_S) \le \sqrt{\frac{2}{m} \log \Gamma_\mathcal{H}(m)}$$


**Shatter.** A hypothesis class $$\mathcal{H}$$ is said to shatter a set $$S_x ⊆ \mathcal{X}$$ if it can achieves all possible labellings of $$S_x$$, i.e. $$\vert \mathcal{H}\vert_{S_x}\vert = 2^m$$.

**VC dimension.** The VC dimension of $$\mathcal{H}$$ is the size of the largest set $$\mathcal{H}$$ can shatter: 

$$\mathrm{VCdim}(\mathcal{H})=\max\{m\ge0: \Gamma_\mathcal{H}(m) = 2^m\}$$

If $$\mathcal{H}$$ can shatter unboundedly large sets, we say its VC dimension is infinite. We can bound the growth function in terms of the VC dimension: $$\Gamma_\mathcal{H}(m)= \mathcal{O}(m^{\mathrm{VCdim}(\mathcal{H})})$$.

**Compute VC dimension**

<!-- **Example** -->

- $$\mathrm{VCdim}(\{x \to \mathbb{1}(x\ge a) :a∈\mathbb{R}\})=1$$.
- $$\mathrm{VCdim}(\{x\to \mathrm{sgn}(w\cdot x) : w\in \mathbb{R}^d\}) = d$$.
- $$\mathrm{VCdim}(\{x\to w\cdot x + b : w\in \mathbb{R}^d, b\in \mathbb{R} \}) = d+1$$.

**Bound the growth function**

- $$m \le \mathrm{VCdim}(\mathcal{H})$$ case: $$\Gamma_\mathcal{H}(m) = 2^m$$
- $$m > \mathrm{VCdim}(\mathcal{H})$$ case: $$\Gamma_\mathcal{H}(m)\le (\frac{em}{\mathrm{VCdim}(\mathcal{H})})^{\mathrm{VCdim}(\mathcal{H})}$$. In this case, 

$$\mathbb{E}_{S\sim D^m} \sup_{h\in\mathcal{H}} L_D(h)-L_{S}(h) \le \sqrt{\frac{2\mathrm{VCdim}(\mathcal{H})}{m} (1+\log m-\log \mathrm{VCdim}(\mathcal{H}))}$$

When $$\mathrm{VCdim}(\mathcal{H}) ≥ 3$$, we can replace $$\log m + 1 − \log \mathrm{VCdim}(\mathcal{H})$$ with simply $$\log m$$ above.

**The overall bound on estimation error**

If $$m > \mathrm{VCdim}(\mathcal{H})$$ and we use zero-one loss

$$\begin{align*}
&\mathrm{Pr}\left(\sup_{h\in\mathcal{H}} [L_D(h) - L_S(h)] > \sqrt{\frac{2\mathrm{VCdim}(\mathcal{H})}{m} (1+\log m-\log \mathrm{VCdim}(\mathcal{H}))} + \sqrt{\frac{1}{2m}\log\frac{1}{\delta}} \right) 
< \delta \\
&\mathrm{Pr}\left(L_S(h^*) - L_D(h^*) > \sqrt{\frac{1}{2m}\log\frac{1}{\delta}} \right) < \delta
\end{align*}$$


With probability at least $$1-\delta$$, the estimation error satisfy

$$ \begin{align*}
L_D(\hat h_S) - L_D(h^*) &\le \sup_{h\in\mathcal{H}} [L_D(h)-L_S(h)]  + L_S(h^*) - L_D(h^*) \\
&\le \sqrt{\frac{2\mathrm{VCdim}(\mathcal{H})}{m} (1+\log m-\log \mathrm{VCdim}(\mathcal{H}))}  + \sqrt{\frac{2}{m}\log\frac{2}{\delta}}
\end{align*}$$

---
#### Lower bound
if we only know upper bounds, we never really know how tight they are, and so we can never really know if one algorithm is better than another.

**No free lunch**

Let $$\mathcal{H}$$ be a hypothesis set of binary classifiers over $$\mathcal{X}$$. Let $$m ≤ \mathrm{VCdim}(\mathcal{H})/2.$$ Then 

$$\inf_{\mathcal{A}}\sup_{D\ realizable\ by\ \mathcal{H}}\mathrm{Pr}_{S\sim D^m}\left(L_D(\mathcal{A}(S)) \ge \frac18 \right) \ge \frac17$$

where the infimum over $$\mathcal{A}$$ is over all learning algorithms which return hypotheses in $$\mathcal{H}$$.
This theorem implies Any $$\mathcal{H}$$ with $$\mathrm{VCdim}(\mathcal{H}) = ∞$$ is not PAC learnable.
- a “no free lunch” theorem, in that there is no algorithm that always works (in the sense of PAC learning): every algorithm fails on at least one distribution.

**Lower bound based on VC dimension**

Let $$\mathcal{H}$$ be a hypothesis set of binary classifiers over $$\mathcal{X}$$. For any $$m\ge 1$$, 

$$\inf_{\mathcal{A}}\sup_{D\ realizable\ by\ \mathcal{H}}\mathrm{Pr}_{S\sim D^m}\left(L_D(\mathcal{A}(S)) \ge \frac{\mathrm{VCdim}(\mathcal{H})-1}{32m} \right) \ge \frac{1}{100}$$

where $$L_D$$ uses zero-one loss, and the infimum over $$\mathcal{A}$$ is over all learning algorithms returning hypotheses in $$\mathcal{H}$$.

---
# Beyond uniform convergence
if we don’t know what the optimal predictor looks like, we can just try a bunch of different H, which induces the
concept of non-uniform bound and structural risk minimization

## Non-uniform learning

Let $$\mathcal{H}=\mathcal{H}_1\bigcup\mathcal{H}_2\bigcup \cdots$$, a set of weights $$w_k ≥ 0$$ and $$\sum_{k}w_k \le 1$$. Assume that each $$\mathcal{H}_k$$ has uniform convergence: 

$$\mathrm{Pr}_{S\sim D^m}(\sup_{h\in\mathcal{H}_k} \ L_S(h) - L_D(h) \le \varepsilon_k(m, \delta)) \ge 1-\delta$$

where for all $$k$$ and $$\delta\in(0,1)$$, $$\lim_{m\to \infty} \varepsilon_k(m, \delta) =0$$.  
Then for any $$D$$, with probability at least $$1 − δ$$ over the choice of $$S ∼ D^m$$ , we have 

$$∀h∈\mathcal{H}. \ L_D(h)≤L_S(h)+ \min_{k:h∈\mathcal{H}_k}\varepsilon_k(m, w_k \delta)$$


{% details Proof for non-uniform bound%}
Given a set of weights $$w_k ≥ 0$$ on $$\mathcal{H}_k$$, for each $$k$$ we have

$$\mathrm{Pr}_{S\sim D^m}(\exists h \in\mathcal{H}_k. \ L_S(h) - L_D(h) > \varepsilon_k(m, w_k \delta)) < w_k \delta$$

Then, 

$$\begin{align*}\mathrm{Pr}_{S\sim D^m}\left( \bigcap_k (\forall h \in\mathcal{H}_k. \ L_S(h) - L_D(h)) \le \varepsilon_k(m, w_k \delta)\right) \ge 1 - \sum_k w_k \delta\\
\Rightarrow \mathrm{Pr}_{S\sim D^m}\left(∀h∈\mathcal{H}. \ L_D(h)≤L_S(h)+ \min_{k:h∈\mathcal{H}_k}\varepsilon_k(m, w_k \delta) \right)  \ge 1-\delta\end{align*}$$
{% enddetails %}
<br>

### Structural risk minimization (SRM)

SRM is then the algorithm that minimizes this upper bound on $$L_D(h)$$

**Definition 1.** Let $$\mathcal{H}=\bigcup_{k:w_k>0}\mathcal{H}_k$$, a set of weights $$w_k ≥ 0$$ and $$\sum_{k}w_k \le 1$$. Given the uniform convergence bound on the decomposition of $$\mathcal{H}$$, structural risk minimization is given by:

$$\mathrm{SRM}_{\mathcal{H}}(S) \in \arg\min_{h\in\mathcal{H}} L_S(h)+ \varepsilon_{k_h}(m, w_{k_h} \delta) \ \text{ where }k_h\in\min_{k:h∈\mathcal{H}_k}ε_k(m,δw_k)$$

where $$k: h\in\mathcal{H}_k$$ refers to the index of the hypothesis classes where $$h$$ lives in (there is at least one $$\mathcal{H}_k$$ that contains $$h$$).

**Definition 2.** Let $$\mathcal{H}=\bigcup_{k:w_k>0}\mathcal{H}_k$$, a set of weights $$w_k ≥ 0$$ and $$\sum_{k}w_k \le 1$$. If we could eliminate the dependence on $$\delta$$ of $$\varepsilon$$, the structural risk minimization can also be defined by:

$$\mathrm{SRM}_{\mathcal{H}}(S) \in \arg\min_{h\in\mathcal{H}} L_S(h)+ \varepsilon_{k_h}(m, w_{k_h}) \ \text{ where }k_h\in\min_{k:h∈\mathcal{H}_k}ε_k(m,w_k)$$

This bound don't need to commit to a certain $$\delta$$ during training, so it is better to use in pratice.

**Estimation error bound of SRM**

Let $$\hat h_S=\mathrm{SRM}_{\mathcal{H}}(S)$$ for simplification and $$h^*$$ be any fixed function in $$\mathcal{H}$$, we have

$$\begin{align*}
\mathrm{Pr}\left(L_D(\hat h_S)  > L_S(h^*) + \varepsilon_{k_{h^*}}(m, \frac{1}{2} w_{k_{h^*}} \delta) \right)  < \mathrm{Pr}\left(L_D(\hat h_S)  > L_S(\hat h_S) + \varepsilon_{k_{\hat h_S}}(m, \frac{1}{2} w_{k_{\hat h_S}} \delta) \right) 
< \frac12\delta \\
\mathrm{Pr}\left(L_S(h^*) - L_D(h^*) > (b-a) \sqrt{\frac{1}{2m}\log\frac{2}{\delta}} \right) < \frac12\delta
\end{align*}$$

Then, with probability at least $$1-\delta$$ over the choice of $$S \sim D^m$$, we have

$$ \begin{align*}
L_D(\mathrm{SRM}_{\mathcal{H}}(S)) - L_D(h^*) 
&\le \varepsilon_{k_{h^*}}(m, \frac{1}{2} w_{k_{h^*}} \delta) + (b-a) \sqrt{\frac{1}{2m}\log\frac{2}{\delta}}
\end{align*}$$

{% details  Example: specific bound using Rademacher complexity%}

Here we introduce a specific bound using Rademacher complexity. Let the Rademacher complexity of $$k$$th hypothesis $$R_k = \mathbb{E}_{S\sim D^m} \mathrm{Rad} (\mathcal{H}_k\vert_S)$$. According to the high probability bounds with Rademacher complexity, for each $$k$$, we have 

$$\begin{align*}
\mathrm{Pr}_{S\sim D^m}\left(\sup_{h\in\mathcal{H}_k} \ L_S(h) - L_D(h) \le 2R_k + (b-a)\sqrt{\frac{1}{2m}\log\frac{1}{\delta}} \right) \ge 1-\delta \end{align*}$$

Then, let $$k_h=\arg\min_{k:h\in\mathcal{H}_k} 2R_{k_h} + (b-a)\sqrt{\frac{1}{2m}\log\frac{1}{w_{k_h}\delta}}$$, we have 

$$\begin{align*}
\mathrm{Pr}_{S\sim D^m}\left(\sup_{h\in\mathcal{H}} \ L_S(h) - L_D(h) \le 2R_{k_h} + (b-a)\sqrt{\frac{1}{2m}\log\frac{1}{w_{k_h}\delta}} \right) \ge 1-\delta
\end{align*}$$

To separate $$w_k$$ and $$\delta$$, we can choose $$w_k=\frac{6}{\pi^2k^2}$$ such that with probability at least $$1-\delta$$

$$\begin{align*}
\sup_{h\in\mathcal{H}} \ L_S(h) - L_D(h) &\le 2R_{k_h} + (b-a)\sqrt{\frac{1}{2m}\log\frac{\pi^2{k_h}^2}{6\delta}} \\
&\le 2R_{k_h} + (b-a)\sqrt{\frac{1}{2m}(2\log{k_h} + \log\frac{\pi^2}{6} + \log\frac{1}{\delta})}\\
&\le 2R_{k_h} + (b-a)\sqrt{\frac{1}{2m}(2\log{k_h} + \log\frac{\sqrt{e}}{\delta})}\\
&\le 2R_{k_h} + (b-a)\sqrt{\frac{1}{m}\log{k_h}} + (b-a)\sqrt{\frac{1}{2m}\log\frac{\sqrt{e}}{\delta}}
\end{align*}$$

Then the SRM solution is given by$$\mathrm{SRM}_{\mathcal{H}}(S) \in \arg\min_{h\in\mathcal{H}} L_S(h)+ 2R_{k_h} + (b-a)\sqrt{\frac{1}{m}\log{k_h}}$$


Let $$\hat h_S=\mathrm{SRM}_{\mathcal{H}}(S)$$ , then with probability at least $$1-\delta$$ 

$$\begin{align*} 
L_D(\hat h_S) &\le L_S(\hat h_S) + 2R_{k_{\hat h_S}} + (b-a)\sqrt{\frac{1}{m}\log{k_{\hat h_S}}} + (b-a)\sqrt{\frac{1}{2m}\log\frac{\sqrt{e}}{\delta}}\\
&\le L_S(h^*) + 2R_{k_{h^*}} + (b-a)\sqrt{\frac{1}{m}\log{k_{h^*}}} + (b-a)\sqrt{\frac{1}{2m}\log\frac{\sqrt{e}}{\delta}}\\
&\le L_D(h^*) + 2R_{k_{h^*}} + (b-a)\sqrt{\frac{1}{m}\log{k_{h^*}}} + (b-a)\sqrt{\frac{1}{2m}\log\frac{2\sqrt{e}}{\delta}}+(b-a) \sqrt{\frac{1}{2m}\log\frac{2}{\delta}} \\
&\le L_D(h^*) + 2\mathbb{E}_{S\sim D^m} \mathrm{Rad} (\mathcal{H}_{k_{h^*}}|_S) + (b-a)\sqrt{\frac{1}{m}\log{k_{h^*}}} + (b-a)\sqrt{\frac{2}{m}\log\frac{3}{\delta}})
\end{align*}$$

where $$h^*$$ is any function in hypothesis $$\mathcal{H}$$.
{% enddetails %}
<br>

<hr>

## Algorithmic stability
- Sometimes $$\mathcal{H}$$ is too big for uniform convergence, but $$\mathcal{A}$$ still learns well.  Stability based bound is algorithm-specific that it can be insightful for models prone to non-uniform behavior, like deep networks.
- Stability-based bounds measure how much the model's performance changes when the training data is slightly perturbed. 

**Randomness of algorithm**

- When we’re dealing with specific algorithms, though, it’s important to note that these algorithms might be randomized: for instance, stochastic gradient descent sees points in a random order, and might start at a random location.
- We’ll always assume that this randomness is independent of $$S$$ (other than its size): for instance, this will be the random seed that determines that pattern in which we access the $$z_i$$ . If $$\mathcal{A}$$ is deterministic, it’s just a point-mass distribution.


**Notation.** for changing single sample points in set $$S$$ : If $$S = (z_1, \dots, z_{i-1}, z_i, z_{i+1}, \ldots, z_m)$$, then $$S^{(i\leftarrow z')}=(z_1, \dots, z_{i-1}, z_i', z_{i+1}, \ldots, z_m)$$.

**Proposition.** For any distribution $$\mathcal{D}$$ and learning algorithm $$\mathcal{A}$$, 

$$\mathbb{E}_{S\sim D^m, \mathcal{A}} [L_D(\mathcal{A}(S)) - L_S(\mathcal{A}(S))] 
= \mathbb{E}_{\substack{S\sim D^m, z_i'\sim D\\ \mathcal{A}, i\sim\text{Unif}([m])}} [\ell(\mathcal{A}(S^{(i\leftarrow z')}), z_i) - \ell(\mathcal{A}(S), z_i)] $$


<!-- ### Definitions -->

**Definition (average stability).**  $$\mathcal{A}$$ is $$ε(m)$$-on-average-replace-one stable if for all $$D$$, 

$$\mathbb{E}_{\substack{S\sim D^m, z_i'\sim D\\ \mathcal{A}, i\sim\text{Unif}([m])}} [\ell(\mathcal{A}(S^{(i\leftarrow z')}), z_i) - \ell(\mathcal{A}(S), z_i)] \le \varepsilon (m)$$

Thus, an $$ε(m)$$-on-average-replace-one stable algorithm will have small average-case generalization gap $$\mathbb{E}_{S\sim D^m, \mathcal{A}} [L_D(\mathcal{A}(S)) - L_S(\mathcal{A}(S))]$$.


**Definition (uniform stability).**  $$\mathcal{A}$$ is $$β(m)$$-uniformly stable if for all $$m ≥ 1$$ and all $$i\in\{1,\dots,m\}$$ 

$$\sup_{\substack{S\sim D^m \\ z, z'\sim D}} [\mathbb{E}_{\mathcal{A}} \ell(\mathcal{A}(S^{(i\leftarrow z')}), z) - \mathbb{E}_{\mathcal{A}}\ell(\mathcal{A}(S), z)] \le \beta (m)$$

That is, changing one point in any training set gives you a hypothesis that looks almost the same for any test point.

**Upper bound $$\mathbb{E}_{\mathcal{A}} [L_D(\mathcal{A} (S))- L_S(\mathcal{A} (S))]$$**

<!-- We’ll apply <a href="https://en.wikipedia.org/wiki/McDiarmid%27s_inequality">McDiarmid's inequality</a> to bound the worst case generalization gap. -->

**Theorem (uniform-stability).** Suppose that $$\ell(h, z) ∈ [a, b]$$ almost surely. Let $$\mathcal{A}$$ be $$β(m)$$-uniformly stable. Then, with probability at least $$1 − δ$$ over the choice of training points $$S ∼ D^m$$, 

$$\mathbb{E}_{\mathcal{A}} [L_D(\mathcal{A} (S))- L_S(\mathcal{A} (S))] \le \beta(m)+(2m\beta(m)+b-a) \sqrt{\frac{1}{2m} \log\frac{1}{\delta}}$$

The best case for this bound is when $$β(m) = \mathcal{O}(1/m)$$, in which case you get a $$\mathcal{O}(1/\sqrt{m})$$ rate.

{% details Proof for the theorem%}
Let $$f(S)=\mathbb{E}_{\mathcal{A}} [L_D(\mathcal{A} (S))- L_S(\mathcal{A} (S))]$$. If $$\mathcal{A}$$ is $$\beta(m)$$-uniformly stable, then $$\mathbb{E}_S f(S)\le \beta(m)$$.
Assume $$\ell \in [a,b]$$, let $$\hat h = \mathcal{A}(S), \hat h^i=\mathcal{A}(S^{(i \leftarrow z')})$$. 

We have 

$$\begin{align*} 
\vert \mathbb{E}_{\mathcal{A}} [L_D(\hat h^i)- L_D(\hat h)] \vert
&= \vert \mathbb{E}_{\mathcal{A}, z\sim D} \ell(\hat h^i, z) - \mathbb{E}_{\mathcal{A}}\ell(\hat h, z) \vert \\
&\le \mathbb{E}_{z\sim D}\vert \mathbb{E}_{\mathcal{A}} \ell(\hat h^i, z) - \mathbb{E}_{\mathcal{A}}\ell(\hat h, z) \vert \\
&\le \beta(m)\\
\end{align*}$$

Also, we have 

$$\begin{align*} 
\vert\mathbb{E}_{\mathcal{A}} [L_S(\hat h)- L_{S^{(i\leftarrow z')}}(\hat h^i)]\vert
&= \frac{1}{m} \left\vert \mathbb{E}_{\mathcal{A}}\sum_{j=1}^m (\ell(\hat h, z_j) - \ell(\hat h^i, z_j))\right\vert\\
&\le \frac{1}{m}  \sum_{j\neq i} \left\vert \mathbb{E}_{\mathcal{A}} \ell(\hat h, z_j)  - \ell(\hat h^i, z_j) \right\vert + \frac{1}{m} \left\vert \mathbb{E}_{\mathcal{A}} \ell(\hat h, z_i)  - \ell(\hat h^i, z_i)  \right\vert \\
&\le \frac{m-1}{m} \beta(m) + \frac{b-a}{m} \\
&\le \beta(m) + \frac{b-a}{m} 
\end{align*}$$


Combining the above two inequalities, we can show the bounded difference of $$f(S)$$ that 

$$\begin{align*}\vert f(S) - f(S^{(i \leftarrow z')})\vert
&= \left\vert\mathbb{E}_{\mathcal{A}} [L_D(\hat h)- L_S(\hat h) - L_D(\hat h^i) +L_{S^{(i \leftarrow z')}}(\hat h^i)] \right\vert \\
&\le \left\vert\mathbb{E}_{\mathcal{A}} [L_D(\hat h)- L_D(\hat h^i)] \right\vert + \left\vert \mathbb{E}_{\mathcal{A}}[ L_S(\hat h) - L_{S^{(i \leftarrow z')}}(\hat h^i)]\right\vert \\
&\le 2\beta(m) + \frac{b-a}{m} 
\end{align*}$$

According to <a href="https://en.wikipedia.org/wiki/McDiarmid%27s_inequality">McDiarmid's inequality</a>, with probability at least $$1-\delta$$ 

$$\begin{align*} 
\mathbb{E}_{\mathcal{A}} [L_D(\mathcal{A} (S))- L_S(\mathcal{A} (S))]
&\le \mathbb{E}_{S} \mathbb{E}_{\mathcal{A}} [L_D(\mathcal{A} (S))- L_S(\mathcal{A} (S))] + \sqrt{\frac12 m(2\beta(m) + \frac{b-a}{m})^2 \log \frac{1}{\delta}}\\
&\le \beta(m) + (2m\beta(m) + b-a)\sqrt{\frac{1}{2m} \log \frac{1}{\delta}}
\end{align*}$$
{% enddetails %}
<br>

### Regularized loss minimization

Major challenges for using ERM to explain deep learning:
- **Computational Hurdle:** ERM minimizes the empirical loss over the training data. However, for complex deep learning models with millions of parameters, this optimization problem becomes computationally NP-hard.
- **Intractability of Uniform Convergence:** Uniform convergence ERM bounds might not be enough for generalization. In practice, deep learning models often exhibit non-uniform convergence, where their performance can vary significantly across different data points.
- **Regularized Loss Minimization (RLM):** In reality, we often observe that regularized loss minimization (RLM), which optimizes a trade-off between model complexity and loss, leads to better generalization performance even if it doesn't strictly minimize the empirical risk. 

Therefore, while ERM remains a theoretical cornerstone for understanding generalization, regularized loss minimization emerges as the preferred approach in practice. By incorporating regularization terms that penalize model complexity, we can achieve better generalization performance and obtain more interpretable models.


Regularized loss minimization (RLM) adds a regularization term (or regularizer) to a loss function:

$$\arg\min_{h\in \mathcal{H}} L_S(h) + λR(h)$$

It is closely connected to constrained ERM:

$$\arg\min_{h\in \mathcal{H}: R(h) \le B} L_S(h)$$

they are Lagrange dual to each other: for any λ, there is some B such that the solutions agree, and vice versa. 

But RLM is typically easier computationally, and it’s usually easier to choose a good λ than to choose a good B.

Suppose the loss function $$h → \ell(h, z)$$ is convex for each $$z$$, and regularizer $$R(h)$$ is 1-strongly convex. Then $$f_S(h) = L_S(h) + λR(h)$$, the sum of a convex function and a λ-strongly convex function, is λ-strongly convex. Let $$\mathcal{A}(S)$$ denote $$\arg\min_{h\in \mathcal{H}} f_S(h)$$; since $$f_S$$ is strongly convex, it has a unique minimizer (so we can leave out the $$\mathbb{E}_\mathcal{A}$$ term).

**Upper bound $$L_D (\mathcal{A}(S))-L_D(h^*)$$**

Assume the regularizer is $$R(h)$$ is nonnegative, and $$R(h^*) \le \frac12 B^2$$ for any $$h^*$$. If $$L_S(h)$$ is convex, $$R(h)$$ is 1-strongly convex and $$l(h,z)$$ is $$\rho$$-Lipschitz, we have 

$$\mathbb{E}_S L_D(\mathcal{A}(S)) \le L_D(h^*)+ \frac12 \lambda B^2+ \frac{4\rho^2}{\lambda m}$$

For any fixed $$h^*$$, it holds with probability at least $$1 − \delta$$ that

$$L_D (\mathcal{A}(S)) \le  L_D(h^*) + \frac \lambda 2 B^2 + \frac{4\rho^2}{\lambda m} + \frac{4\rho^2}{\lambda m} \sqrt{\frac m 2 \log \frac1 \delta}$$

`Proof sketch.`
- Given the convex and Lipschitz assumption, we can show RLM is $$\frac{4\rho^2}{\lambda m}$$-uniformly stable 
- then bound $$\mathbb{E}_S L_D(\mathcal{A}(S)) - L_S(\mathcal{A}(S))$$ using uniformly stability
- also, $$L_S(\mathcal{A}(S)) \le L_S(h^*)+R(h)$$
- Then $$\mathbb{E}_S L_D(\mathcal{A}(S)) \le L_D(h^*)+ \frac12 \lambda B^2+ \frac{4\rho^2}{\lambda m}$$
- Use McDiarmid's inequality to derive the high probability bound$$L_D (\mathcal{A}(S)) \le \mathbb{E}_S L_D(\mathcal{A}(S)) + \frac{4\rho^2}{\lambda m} \sqrt{\frac m 2 \log \frac1 \delta}$$


{% details Full proof%}
We first show that the function $$f_S(h)=L_S(h) + λR(h)$$ is  $$\frac{4\rho^2}{\lambda m}$$-uniformly stable if $$L_S(h)$$ is convex, $$R(h)$$ is 1-strongly convex and $$l(h,z)$$ is $$\rho$$-Lipschitz. Let $$S'=S^{(i \leftarrow z')}$$, then

$$\begin{align*} 
f_S(h)-f_S(g)&=f_S(h)-f_{S'}(h)+f_{S'}(h) - f_{S'}(g)+f_{S'}(g)-f_{S}(g)\\
&=f_{S'}(h) - f_{S'}(g)+ \frac{1}{m} (\ell (h,z_i) - \ell(h, z_i')) + \frac{1}{m} (\ell (g,z_i') - \ell(g, z_i))\\
&=f_{S'}(h) - f_{S'}(g)+ \frac{1}{m} (\ell (h,z_i) -  \ell(g, z_i)) + \frac{1}{m} (\ell (g,z_i') - \ell(h, z_i'))
\end{align*}$$

Let $$\hat h = \arg\min_h f_S(h)$$ and $$\hat h^i = \arg\min_h f_S'(h)$$. Then,

$$\begin{align*} 
f_S(\hat h^i) - f_S(\hat h) &\le  \frac{1}{m} (\ell (\hat h^i,z_i) -  \ell(\hat h, z_i)) + \frac{1}{m} (\ell (\hat h,z_i') - \ell(\hat h^i, z_i'))
\end{align*}$$

As $$f_S(h)$$ is $$\lambda$$-strongly convex and $$\nabla f_S(\hat h)=0$$, then 

$$f_S(\hat h^i) - f_S(\hat h) \ge \langle \nabla f_S(\hat h), \hat h^i-\hat h \rangle +\frac12 \lambda \Vert \hat h^i-\hat h\Vert^2 = \frac12 \lambda \Vert\hat h^i-\hat h\Vert^2$$

Thus, 

$$\begin{align*} 
\frac12 \lambda \Vert \hat h^i-\hat h\Vert ^2 &\le \frac{1}{m} (\ell (\hat h^i,z_i) -  \ell(\hat h, z_i)) + \frac{1}{m} (\ell (\hat h,z_i') - \ell(\hat h^i, z_i'))\le \frac{2\rho}{m} \Vert \hat h^i-\hat h\Vert \\
&\Rightarrow \Vert \hat h^i-\hat h\Vert  \le \frac{4\rho}{\lambda m} \\
&\Rightarrow \vert\ell (\hat h,z_i') - \ell(\hat h^i, z_i')\vert \le \frac{4\rho^2}{\lambda m}
\end{align*}$$

$$\mathcal{A}$$ is  $$\frac{4\rho^2}{\lambda m}$$-uniformly stable.

Assume $$R(h)$$ is non-negative, then 

$$L_S(\mathcal{A}(S)) \le L_S(\mathcal{A}(S))+\lambda R(\mathcal{A}(S)) \le L_S(h^*)+\lambda R(h^*)$$

for all 
$$h^* \in \mathcal{H}$$. 
Then $$\mathbb{E}_S L_S(\mathcal{A}(S)) \le L_D(h^*)+\lambda R(h^*)$$.

$$\mathbb{E}_S L_D (\mathcal{A}(S)) \le \mathbb{E}_S L_S(\mathcal{A}(S)) + \frac{4\rho^2}{\lambda m}\le L_D(h^*) + \frac \lambda 2 B^2 + \frac{4\rho^2}{\lambda m}$$ 

where we don’t actually need to apply uniform-stability theorem directly.
{% enddetails %}
<br>


---
layout: post
title:  a note on neural data compression
date:   2034-01-28
description: a short introduction to neural data compression
tags: information-theory note
categories: basics
related_posts: true
giscus_comments: false
toc:
  beginning: true

---

# Lossless Compression
1. build a probabilistic model of the data, 
2. feed its probabilities into a so-called entropy coding scheme which converts data into compact bit-strings


## Entropy coding

### basic concepts
- We call a sequence of outcomes of a discrete random variable a message, and entropy coding is a way to achieve lossless compression of a message.
- Entropy coding techniques, aim to minimize the average number of bits required to represent the data, resulting in efficient compression
- Symbol coding can be a part of entropy coding algorithms, such as Huffman coding, where variable-length codes are assigned to symbols based on their frequency of occurrence, thereby reducing the message’s overall length.

### entropy and information
- Entropy, by definition, is the **expected value of surprise**,
$$H [X] = E_{x∼P} [− \log_2 P (x)]$$
- the expected message length $$\vert C(x)\vert$$ of an optimal prefix-free code C is bounded as
$$H[X] \le E[\vert C(X)\vert] \le H[X] + 1$$.
- We usually to approximate it with a distribution Q, and estimate Q by minimizing the cross-entropy between P and Q, defined as, $$H[P, Q] = E_{x∼P} [− \log_2 Q(x)]$$.
- The cross-entropy is always at least as large as the entropy, and the gap between the two is called relative entropy or Kullback-Leibler (KL) divergence, defined as
$$d_{KL}[P \vert Q] = E_{x∼P} [− \log_2 Q(x) + \log_2 P (x)]$$.

### Coding techniques
#### Huffman coding 
- belongs to symbol coding
- is a specific type of entropy coding that is used to compress data by assigning variable-length codes to different symbols based on their frequency of occurrence.
- Huffman coding assigns a codeword with length at most $$⌈−log_2 P(x)⌉$$ for each symbol x, and can be shown to be optimal among prefix codes

#### Arithmetic coding
- also known as range coding, is an example of a streaming code
- Streaming codes differ from symbol codes in that they assign codewords to entire messages and individual symbols do not have unique codewords
- The basic idea of arithmetic coding is to associate each symbol with a subinterval of the interval $$[0,1]$$ such that the subinterval’s length equals the symbol’s probability.
- If $$x^n$$ is a message of length n, and $$C(x^n)$$ the codeword assigned by arithmetic coding using an assumed probability distribution Q, then $$\vert C(x^n)\vert < \lceil − log_2 Q(x^n)\rceil + 1$$
- The advantage of arithmetic coding over Huffman coding is that the excess bits needed for encoding the message are amortized over all symbols, making it more efficient for long messages.


### Continuous models for discrete data
- Lossless compression generally operates on discrete data.
- it is infeasible to losslessly encode any real number with a finite number of bits
- However, many generative models in machine learning assume continuous data, and model it with a density function. It turns out such continuous models are still useful for lossless compression.
- Suppose we have a density model q over $$R^D$$, and $$x ∈ \{0,...,255\}^D$$ is an RGB image following the ground truth image distribution P.$$Q(x) :=\int_{[−.5,.5)^D} q(x + u) du$$
- Assume we add uniform noise $u ∈ [−0.5, 0.5)^D$ to the data and that the resulting noisy/dequantized data,$$y = x + u$$, has density $$p$$. Then it is not difficult to show that
$$− \int p(y) \log_2 q(y) dy \ge - \sum_x P (x) \log_2 Q(x).$$


### Lossless compression models
#### Autoregressive models
- we can write any probability distribution as a product of conditional distributions using the chain rule of probabilities
$$p(x) = \prod_i p(x_i \vert x_{<i}).$$
- $$x_i$$ is the i-th entry of the vector $x$ and $$x_{<i}$$ corresponds to all previous entries in an arbitrary order. The autoregressive factorization does not make any assumptions about the data distribution yet still al- lows us to easily incorporate useful assumptions.
- LSTMs, RIDE, PixelRNN, or PixelCNN++

#### Latent variable models
- Latent variable models represent the data distribution using the sum rule of probability $$p(x) = \int p(x, z) dz = \int p(x \vert z)p(z) dz$$ where z is a vector of latent variables (also called “latents”), and the joint distribution p(x, z) factorizes into a prior $$p(z)$$ and a likelihood $$p(x \vert z)$$.
- hidden Markov models, variational autoencoders (VAEs)
- unlike in a fully-observed (e.g., autoregressive) model where the data probability $$p(x)$$ can be readily evaluated, doing so is no longer straightforward since it is defined through an often intractable integral


# Lossy Compression
- compression with imperfect data reconstruction.
- When working with continuous data, such as an analogue signal, lossy compression is necessary, as it is impossible to losslessly store real values with a finite number of bits.

## Background
- lossy compression involves encoding the given data into a discrete representation, which is communicated losslessly to the receiver, using as few bits as possible.
- The quality of lossy reconstructions is measured by a fidelity criterion, or a distortion function between pairs of original data samples and reconstructions.


## Rate-distortion theory
- we formalize a lossy compression algorithm (or a lossy codec) as a 3-tuple consisting of an encoder, a decoder, and an entropy code, denoted by $$c = (e,d,γ)$$.

$$X \xrightarrow{\text{encoder } e} W \xrightarrow{\text{losslessly transimit with } \gamma} W \xrightarrow{\text{decoder }d} \hat X$$

- The encoder $$e : X → W$$ maps each data point $$x ∈ X$$ to a representation w in a countable set W
- The decoder $$d : W → \hat X$$ maps each representation w to a reconstruction point $$\hat x ∈ \hat X$$.
- The encoder and decoder further agree to transmit the symbols of W losslessly with an entropy code $$γ$$
- $$l(x) := \vert γ(e(x))\vert$$ to denote the code length assigned to the encoding of x.
- Suppose we are given a distortion function $$ρ : X × \hat X → [0, ∞)$$ that measures the error caused by representing x with the lossy reconstruction $$\hat x$$.
- Given the above, lossy compression is then generally concerned with minimizing the average distortion 

$$D = E_{x∼P_{data}} [ρ(x, \hat x)]$$ 

where $$\hat x:= d(e(x))$$, and simultaneously, the (bit-)rate 

$$R = E_{x∼P_{data}} [l(x)]$$ 

that is, the average number of bits needed to encode the data.
- Any lossy compression algorithm implements a noisy channel that receives a data input X and outputs a reconstruction $$\hat X$$, described by a conditional distribution $$Q_{\hat X\vert X}$$
- The mutual information, $$I[X,\hat X]$$ , is then defined as the KL divergence between the joint distribution $$P_{data} \cdot Q_{\hat X\vert X}$$ and the product of its two marginal distributions $$P_{data} \cdot Q_{\hat X}$$. $$I[X,\hat X]$$ is a fundamental measure of the amount of information needed to transmit $$\hat X$$ given X (in bits per sample). **Mutual information measures the reduction in uncertainty about one random variable due to the knowledge of another.**  Mutual information is symmetric.

$$I[X,\hat X] = \mathbb{E}_{\begin{subarray}{c} x\sim P_X \\ \hat x \sim Q_{\hat X}\end{subarray}} 
[\log_2 \frac{p(x, \hat x)}{p(x)p(\hat x)}]
= \mathbb{E}_{x\sim P_X } [-\log_2 {p(x)} ] + \mathbb{E}_{x \sim Q_{X\vert \hat X}} [ \log_2 p(x\vert \hat x)] = H(P_X)-H(Q_{X\vert \hat X})$$

- for a given distortion threshold D, the **lowest achievable bit-rate** is characterized by the information rate-distortion (R-D) function 

$$R_I(D) = \inf_{Q_{\hat X\vert X}: \mathbb{E}[\rho(X,\hat X)]\le D} I[X,\hat X]$$

- The rate distortion function $$R(D)$$ for an i.i.d. source X with distribution p(x) and bounded distortion function $$\rho(X,\hat X)$$ is equal to the associated information rate distortion function $$R_I(D)$$. $$R(D)=R_I(D)$$ 
- The (information) R-D function is a fundamental quantity that depends only on the data distribution and choice of distortion function, and generalizes the Shannon entropy H from lossless compression to lossy compression. The rate-distortion (R-D) function is a concept from information theory that characterizes the trade-off between the rate at which information is transmitted and the distortion incurred in the process.
- The rate distortion function R(D) is a nonincreasing convex function of D.
- We can relax the constrained optimization problem to an unconstrained one, by introducing the rate-distortion Lagrangian
 
$$L(λ, c) = R(c) + λD(c) = E[l(X)] + λE[ρ(X, \hat X)]$$

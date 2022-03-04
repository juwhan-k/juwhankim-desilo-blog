---
layout: post
title: "CKKS encoding (Part I)"
subtitle: "Polynomial coefficients and the folding pattern of the quotient ring"
categories: ["engineering explained"]
tags: [CKKS, encoding]
date: 2022-02-25 08:40:00 +0900
---

## Foreword

I find, in many cases engineers have hard time understanding the core concepts of cryptographic algorithms. With the advent of the modern crypto, such as [Homomorphic Encryption](https://en.wikipedia.org/wiki/Homomorphic_encryption) (HE), the tendency is even more exacerbated.

I think the problem stems from the fact, the modern crypto utilizes some intense and abstract constructions of number theory, that are often cumbersom to understand. For us, engineers including me and colleagues, that trend poses a problem, because we're often subject to the task of _implementing them_.

Hence, I decided to explain the things that I understand about one of the most import HE algorithm out there, published in [Homomorphic Encryption for Arithmetic of Approximate Numbers](https://eprint.iacr.org/2016/421.pdf) (usually cited as CKKS), in engineering terms.

For the purpose, I will try my best to avoid using concepts and terms from number theory, and explain how they can be implemeted in a computer programming language _python_. Although, no where elegant in representation, I hope my effort will provide _some_ reference understanding toward the somewhat mystified inner works of HE, more specifically CKKS.

## Polynomials.

In CKKS, multiple texts are encoded in _coefficients_ of a polynomial. That is, if the messages array is $\mathbf{m} = \\{ m_0, m_1, \ldots, m_{N-1} \\}$, the array gets encoded into coefficients of

$$
P[X] = c_0 + c_1 X + c_2 X^2 + \ldots + c_{N-1} X^{N-1}
$$

In other words, encoding sends $\mathbf{m}$ to $\mathbf{c} = \\{ c_0, c_1, \ldots, c_{N-1} \\}$. 

We necessiate the conversion (encoding), for the reason of operating in polynomials gives the encryptin scheme stronger security. Crudly speaking, multiplying 2 polynomials ends up mixing all the coefficients across --- polynomial multiplication is convolution of coefficients. Also, we require that the coefficients are integers,  because we want the arithmetic to be _exact_.

In the course of key generation, encryption, decryption, and homomorphic operations, we apply an operation repeatedly. That is polynomial multiplication. A problematic outcome of the operation is --- each multiplication bumps up the highest order of the polynomial! In other words, the resultant of the multiplication is a polynomial of roughly twice the length of the multiplicands.

For example,

$$P1 = -3 + 3X -4X^2$$, and
$$P2 = 1 -2X^2$$, then
$$P1 \times P2 = -3 + 3X + 2X^2 -6X^3 + 8X^4$$

## Quotient ring of polynomials.

In the above example, observe that the highest degree term of $P1$ and $P2$ is $X^2$, while $P1 \times P2$ has $X^4$. We want the length of the polynomial bounded in successive multiplications. For the purpose, we use the quotient ring of polynomials, instead of direct polynomials. The quotient ring is a remainder of a polynomial when divided by another polynomial. For CKKS, we us the form
$$
\mathbb{Z}[X]/\Phi_M(X)
$$, where $\mathbb{Z}[X]$ is a polynomial with integer coefficients, and $\Phi_M(X)$ is the $M$th cyclotomic polynomial, that is
$$
\Phi_M(X) = X^{M/2} + 1 = X^N + 1
$$, and $M = 2^K$ ($K \gt 2$).

A [cyclotomic polynomial](https://en.wikipedia.org/wiki/Cyclotomic_polynomial) is simply a irreducible polynomial with integer coefficients.

Again, let's look at a concrete example. Let $P[X] = -2+X^2−3X^3−2X^4−3X^5−2X^6$, and $\Phi_8(X) = 1 + X^4$. You can think of $P[X]$ as a product of two polynomials in $\mathbb{Z}[X]/\Phi_8(X)$. Polynomials in $\mathbb{Z}[X]/\Phi_8(X)$ has $X^3$ as the highest degree term, such that their product yields $X^6$ as the highest. Performing the long (Euclidean) division of $P[X]$ and $\Phi_8(X)$ gives
$$
(-2+X^2−3X^3−2X^4−3X^5−2X^6) - (-2X^2(1 + X^4)) = -2 + 3X^2 − 3X^3 − 2X^4 − 3X^5
$$
$$
-2 + 3X^2 − 3X^3 − 2X^4 − 3X^5 - (-3X(1 + X^4)) = -2 + 3X + 3X^2 − 3X^3 − 2X^4
$$
$$
-2 + 3X + 3X^2 − 3X^3 − 2X^4 - (-2(1 + X^4)) = 3X + 3X^2 -3X^3
$$
, in successive order. Hence, $P[X]/\Phi_8(X) = 3X + 3X^2 -3X^3$.

## The folding pattern.

You might have already noticed, but the above procedure yields a special pattern. If we divide the coefficient array $\mathbf{c} = \\{-2, 0, 1, -3, -2, -3, -2 \\}$, into two halves as
$$
\mathbf{c}_1 = \\{-2, 0, 1, -3\\}
$$, and
$$
\mathbf{c}_2 = \\{-2, -3, -2, 0\\}
$$
by assigning the element at the center to $\mathbf{c}_1$, and padding $\mathbf{c}_2$ with an extra $0$.
We can calculate the coefficients of $P[X]/\Phi_8(X)$, $\mathbf{c}_3$ as
$$
\mathbf{c}_3 = \mathbf{c}_1 - \mathbf{c}_2 = \\{0, 3, 3, -3\\}
$$.

Indeed, $P[X]/\Phi_M(X)$ when $M = 2^K$, in terms of division always follows this ___folding___ pattern such that
$$
\mathbf{c}_1 =\\{c^1_0, c^1_1,\ldots, c^1\_{N-1} \\}
$$ of $P_1[X]$,
$$
\mathbf{c}_2 =\\{c^2_0, c^2_1,\ldots, c^2\_{N-1} \\}
$$ of $P_2[X]$, and
$$
\mathbf{c}_3 =\\{c^3_0, c^3_1,\ldots, c^3\_{2N-2} \\}
$$ 
of $P_3[X] = P_1[X] \times P_2[X]$, then $\mathbf{c}_3$ can be cut into halves as
$$
\mathbf{c}_3^0 = \\{c^3_0, c^3_1, \ldots, c^3\_{N-1}\\}
$$, 
$$
\mathbf{c}_3^1 = \\{c^3_N, c^3\_{N+1}, \ldots, c^3\_{2N-2}, 0\\}
$$, and
$$
\mathbf{c}_3^2 = \\{c^3_0 - c^3_N, c^3_1 - c^3\_{N+1}, \ldots, c^3\_{N-2} - c^3\_{2N-2}, c^3\_{N-1}\\}
$$ which is the coefficient array of $P[X]/\Phi_M(X)$.

Note that the superscript of the coefficient terms $c$ denotes the polynomial index, _not_ exponentiation of the term.

Graphically,
<img src="/assets/images/posts/coefficient_folding.png" width="50%" >

This folding pattern is extremely important, because when related to the FFT the pattern results in a very useful property. In fact, when $M = 2^K$, FFT of $P[X]$ is the same as FFT of $P[X]/\Phi_M(X)$, as long as the highest order term of $P[X]$ is less than or equal to $2N-2$. In FFT lingo, $P[X]$ and $P[X]/\Phi_M(X)$ are analytically equivalent. Also, the coefficient at $N-1$ is sometimes denoted as the _Nyquist frequency term_.

## What's next?

In this posting, we have briefly examined polynomials of specific forms, that are used in CKKS. When divided by a $M$th cyclotomic polynomial $\Phi_M(X)$, the coefficients follow a specific pattern of calculation, namely ___folding___.

In the proceeding article, equipped with the knowledge of the ___folding___ pattern, we will develop FFT-like encoding/decoding algorithms. There, it will become evident how the pattern can be exploited to accelerate the encoding/decoding procedure.
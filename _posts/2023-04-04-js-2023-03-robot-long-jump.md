---
layout: post
title:  "Solution to the Jane Street Puzzle of March 2023: Robot Long Jump"
date:   2023-04-04 08:00:00 +0000
image1: /assets/images/js-2023-03-1.png
image2: /assets/images/js-2023-03-2.png
---

<!-- Intro -->
In this post, we will solve the Jane Street puzzle of [March 2023](https://www.janestreet.com/puzzles/robot-long-jump-index/). This is a probability and game theory brainteaser, whose solution involves using dynamic programming, calculus and differential equations. We will also use a Python script to do some symbolic computations, which can be found in my [GitHub](https://github.com/miguelbper/jane-street-puzzles/blob/main/2023-03-robot-long-jump.py).


# Statement of the problem
In this puzzle, two robots will compete in a long jump match. 

Let's start by focusing on a single robot. In each **attempt**,
the robot is at $0$ on the line of real numbers and faces towards $1$. The robot will then execute a sequence of **advances**, where an advance works as follows.

The robot **draws** a number $y \in [0,1]$ uniformly at random, and moves to $x + y$, where $x$ is the position of the robot before the random draw. If $x + y \geq 1$, the attempt gets a **score** of $0$. If $x + y < 1$, the robot has the option to **wait** or to **jump**.
- If the robot decides to wait, nothing else happens this advance, and we move to the next advance (where the robot will now start at position $x + y$).
- If the robot decides to jump, another random number $z \in [0, 1]$ is drawn (uniformly). In this case, the attempt terminates and receives a score of $x + y + z$. 

In the competition, there will be two robots. Every round, each robot has a single attempt to score. If one of the robots scores higher than the other, that robot wins. If the robots tie in score, we do another round. The robots are programmed to use an optimal strategy and each robot does the attempt without knowing the other robot's score. Question: what is the probability that an attempt scores $0$?


# Outline of the solution
The strategy of a robot consists of a rule for when the robot will decide to jump or to wait (these are the only decisions that a robot can make in this game). Suppose that $0 < x < y < 1$. Let's suppose that a robot is playing optimaly.
- If the robot decides to jump when at $x$, then the robot would also decide to jump when at $y$.
- If a robot decides to wait when at $y$, then the robot would also decide to wait when at $x$.

Therefore, there exists a unique number $\lambda_0$ such that the robot decides to wait when at $x < \lambda_0$, and the robot decides to jump when at $x > \lambda_0$. Since the game is symmetric, both robots will choose the same $\lambda_0$.

We wish to compute the probability that an attempt scores $0$. For this, we need to compute $\lambda_0$. To compute this probability, we will need to find an expression for 

$$
    p(\lambda, \mu) \coloneqq P(R_1 \text{ wins} \mid R_1 \text{ uses strategy } \lambda \text{ and } R_2 \text{ uses strategy } \mu).
$$

Here, we are labelling the robots $R_1$ and $R_2$. Notice that this is the analogue of the payoff matrix for our puzzle (where each robot has a continuous range of strategies available). But, to compute $p(\lambda, \mu)$, we need to compute the probability distribution of the score that a robot can achieve (notice that for a fixed value of $\lambda$, the score is a random variable). The answer to the original question can then be given by replacing $\lambda = \lambda_0$ in the probability distribution of the score.


# Computing the probability distribution of the score
In this section, we will focus on a single robot, and we will assume that the number $\lambda$ is fixed. Define

$$
    \begin{align*}
        S_\lambda &\coloneqq \text{score of the robot while using strategy $\lambda$ (this is a random variable),} \\
        F_\lambda &\coloneqq \text{cumulative distribution function of $S_\lambda$,} \\
        f_\lambda &\coloneqq \text{probability density function of $S_\lambda$.}
    \end{align*}
$$

Our goal in this section is to compute $F_\lambda(s) \coloneqq P(S_\lambda \leq s)$ for every $s$. For this, define

$$
    \begin{align*}
        p_{\lambda,s}(x)        &\coloneqq P(S_\lambda \leq s \mid \text{robot starts at $x$}), \\
        p_{\lambda,s}(x \mid y) &\coloneqq P(S_\lambda \leq s \mid \text{robot starts at $x$, the random draw is $y$}), \\
        f_U                     &\coloneqq \text{probability density function of the uniform distribution}.
    \end{align*}
$$

Notice that $F_\lambda(s) = p_{\lambda,s}(0)$. Our strategy will be to use the law of total probability to deduce an integral equation for $p_{\lambda,s}$. Differentiating this integral equation, we will obtain a differential equation for $p_{\lambda,s}$. We will then solve the ODE and obtain $p_{\lambda,s}(x)$ for every $x$, in particular also $p_{\lambda,s}(0) = F_\lambda(s)$. This reasoning can be considered dynamic programming, in the sense that we are expressing the solution to a subproblem ($p_{\lambda,s}(x)$) in terms of the solution to "neighbouring" subproblems ($p_{\lambda,s}(x + y)$).

The previously mentioned integral equation is obtained by the following computation:

$$
    \begin{align*}
        p_{λ,s}(x)
        &= \int_0^1 p_{λ,s}(x|y) f_U(y) dy \\
        &= \int_0^1 p_{λ,s}(x|y) dy \\
        &= \int_{0}^{λ-x} p_{λ,s}(x|y) dy + \int_{λ-x}^{1-x} p_{λ,s}(x|y) dy + \int_{1-x}^{1} p_{λ,s}(x|y) dy \\
        &= \int_{0}^{λ-x} p_{λ,s}(x+y) dy + \int_{λ-x}^{1-x} \mathrm{len}([0,s] \cap [x+y, x+y+1]) dy + \int_{1-x}^{1} 1 dy \\
        &= \int_{x}^{λ} p_{λ,s}(u) du + \int_{λ}^{1} \mathrm{len}([0,s] ∩ [u, u+1]) du + x.
    \end{align*}
$$

Here, in the first equality we used the law of total probability, in the second equality we used the fact that $f_U(y) = 1$ (since the distribution is uniform), in the third equality we used the fact that integrals are additive, in the fourth equality we used the rules of how the game is scored, and in the fifth equality we used the change of variables $u = x + y$. Here, $\mathrm{len}([0,s] ∩ [u, u+1])$ is the length of the interval $[0,s] ∩ [u, u+1]$. Denoting $h_{λ,s} \coloneqq ∫_{λ}^{1} \mathrm{len}([0,s] ∩ [u, u+1]) du$, we have shown that

$$
    p_{λ,s}(x) = \int_{x}^{λ} p_{λ,s}(u) du + h_{λ,s} + x,
$$

which is the desired integral equation. This implies that

$$
    \begin{align*}
        \frac{d}{dx} p_{λ,s}(x) &= - p_{λ,s}(x) + 1,  \\
        p_{λ,s}(\lambda)        &= h_{λ,s} + \lambda. 
    \end{align*}
$$

This is a 1st order, linear, constant coefficient ODE. We are also given
one boundary condition. The solution of the boundary value problem is

$$
    p_{λ,s}(x) = 1 + (h_{λ,s} + λ - 1) \exp(λ - x).
$$

Therefore,

$$
    \begin{align}
        F_λ(s) &= 1 + (h_{λ,s} + λ - 1) \exp(λ), \\
        f_λ(s) &= \frac{d}{ds} F_\lambda(s) = \exp(\lambda) \frac{d}{ds} h_{\lambda,s}.
    \end{align}
$$

It remains to compute $h_{λ,s}$. Using the definition of $h_{λ,s}$, it is possible to show that

$$
    h_{λ,s} = 
    \begin{cases}
        0                                                 & \text{if } 0 \leq s < \lambda, \\
        (s - l)^2 / 2                                     & \text{if } \lambda \leq s < 1, \\
        (1 - l) (2s - l - 1) / 2                          & \text{if } 1 \leq s < 1 + \lambda, \\
        (s - 1 - l)(2 - s) + (2 - s)^2/2 + (1 - l)(s - 1) & \text{if } 1 + \lambda \leq s \leq 2.
    \end{cases}
$$

This gives us the desired closed form solutions for the probability distribution of $S_\lambda$.

It will be helpful to do all these computations symbolically (using Python and `sympy`), since the computations in the upcoming sections can be quite cumbersome. The following Python code computes $h_{λ,s}$, $F_\lambda(s)$, and $f_\lambda(s)$:

{% highlight python %}
from sympy import exp, diff, integrate, nsolve
from sympy.abc import s, l, m

# h_{λ,s}
h0 = 0                                                    # 0 <= s < λ
h1 = (s - l)**2 / 2                                       # λ <= s < 1
h2 = (1 - l) * (2*s - l - 1) / 2                          # 1 <= s < 1 + λ
h3 = (s - 1 - l)*(2 - s) + (2 - s)**2/2 + (1 - l)*(s - 1) # 1 + λ <= s <= 2

# Cumulative distribution function
F0 = 1 + (h0 + l - 1) * exp(l)
F1 = 1 + (h1 + l - 1) * exp(l)
F2 = 1 + (h2 + l - 1) * exp(l)
F3 = 1 + (h3 + l - 1) * exp(l)

# Probability density function
f0 = diff(F0, s)
f1 = diff(F1, s)
f2 = diff(F2, s)
f3 = diff(F3, s)
{% endhighlight %}

# Computing the probability that a robot wins

In this section, we will imagine that the robots $R_1$ and $R_2$ are about to compete in a match, where $R_1$ is using a threshold of $\lambda$ to decide when to jump and $R_2$ is using a threshold of $\mu$. Our goal is to compute

$$
    p(\lambda, \mu) \coloneqq P(R_1 \text{ wins} \mid R_1 \text{ uses strategy } \lambda \text{ and } R_2 \text{ uses strategy } \mu).
$$

By the law of total probability,

$$
    \begin{align*}
        p(λ, μ)
        
        =&\; p(λ, μ | S_λ=0 \text{ and } S_μ=0) P(S_λ=0 \text{ and } S_μ=0) \\
        &  + p(λ, μ | S_λ=0 \text{ and } S_μ>0) P(S_λ=0 \text{ and } S_μ>0) \\
        &  + p(λ, μ | S_λ>0 \text{ and } S_μ=0) P(S_λ>0 \text{ and } S_μ=0) \\
        &  + p(λ, μ | S_λ>0 \text{ and } S_μ>0) P(S_λ>0 \text{ and } S_μ>0) \\
        
        =&\; p(λ, μ) P(S_λ=0) P(S_μ=0) \\
        &  +       0 \cdot P(S_λ=0) P(S_μ>0) \\
        &  +       1 \cdot P(S_λ>0) P(S_μ=0) \\
        &  + P(R1 \text{ wins and } S_λ>0 \text{ and } S_μ>0) \\

        =&\; p(λ, μ) P(S_λ=0) P(S_μ=0) + P(S_λ>0) P(S_μ=0) + P(S_λ > S_μ > 0).
    \end{align*}
$$

Solving for $p(\lambda, \mu)$, we obtain that

$$
    \begin{align*}
        p(λ, μ) 
        &= \frac{P(S_λ>0) P(S_μ=0) + P(S_λ > S_μ > 0)}{1 - P(S_λ=0) P(S_μ=0)} \\
        &= \frac{(1 - F_λ(0)) F_μ(0) + P(S_λ > S_μ > 0)}{1 - F_λ(0) F_μ(0)}.

    \end{align*}
$$

We need to compute $q(\lambda, \mu) \coloneqq P(S_λ > S_μ > 0)$. It will be enough to know $p(λ, μ)$ and $q(λ, μ)$ in the case $λ < μ$. This can be computed by integrating the joint probability density function of $(S_\lambda, S_\mu) = (x, y)$ in the region $0 < y < x$ (which implies $\lambda \leq x \leq 2$ and $\mu \leq y < x$). The scores of both robots are independent of one another, so the joint probability density is $f_{\lambda,\mu}(x,y) \coloneqq f_λ(x) f_μ(y)$.

$$
    \begin{align*}
        q(\lambda, \mu)
        &= ∫_{λ}^{2} ∫_{μ}^{x} f_λ(x) f_μ(y) dy dx \\
        &= ∫_{λ}^{2} f_λ(x) ∫_{μ}^{x} f_μ(y) dy dx \\
        &= ∫_{λ}^{2} f_λ(x) (F_μ(x) - F_μ(μ)) dx \\
        &= ∫_{  λ}^{  μ} (\ldots) dx 
         + ∫_{  μ}^{  1} (\ldots) dx 
         + ∫_{  1}^{λ+1} (\ldots) dx 
         + ∫_{λ+1}^{μ+1} (\ldots) dx 
         + ∫_{μ+1}^{  2} (\ldots) dx.
    \end{align*}
$$

Here, in the first equality we are using the definition of the joint probability density, in the second equality we are using the fact that $f_\lambda(x)$ is constant with respect to $y$, in the third equality we are using the fundamental theorem of calculus, and in the fourth equality we split the integral into five regions using additivity. We need to do this because the functions $f_\lambda$ and $F_\mu$ have a different expression for each region.

The following Python code finishes the computations of $q(\lambda, \mu)$ and $p(λ, μ)$:

{% highlight python %}
I0 = integrate(f1 * (F0.subs({l:m}) - F0.subs({l:m, s:m})), (s,   l,   m))
I1 = integrate(f1 * (F1.subs({l:m}) - F0.subs({l:m, s:m})), (s,   m,   1))
I2 = integrate(f2 * (F2.subs({l:m}) - F0.subs({l:m, s:m})), (s,   1, l+1))
I3 = integrate(f3 * (F2.subs({l:m}) - F0.subs({l:m, s:m})), (s, l+1, m+1))
I4 = integrate(f3 * (F3.subs({l:m}) - F0.subs({l:m, s:m})), (s, m+1,   2))
q = I0 + I1 + I2 + I3 + I4
p = ((1 - F0)*(F0.subs({l:m})) + q) / (1 - F0 * F0.subs({l:m})) 
{% endhighlight %}

We only computed $p(\lambda, \mu)$ for $\lambda < \mu$. By symmetry, we know that $p(\lambda, \mu) + p(\mu, \lambda) = 1$, which would allow us to compute $p(\lambda, \mu) = 1 - p(\mu, \lambda)$ in the case $\lambda > \mu$. Alternatively, we could replicate the previous reasoning while assuming $\lambda > \mu$, and then use the property $p(\lambda, \mu) + p(\mu, \lambda) = 1$ as a sanity check. However, this won't be necessary for us to obtain the solution. We could also check that the probabilities we computed are correct by performing Monte-Carlo simulations.

# Computing the Nash equilibrium and the answer

We now compute the Nash equilibrium of this game, as well as the answer
to the original question.

**Fact**: The optimal strategies $(λ_0, μ_0)$ are such that $\nabla p(λ_0, μ_0) = 0$.  
*Explanation*: If $∇p(λ_0, μ_0) \neq 0$, it would be possible for one of the 
robots to change their strategy and obtain a higher probability of 
winning.

**Fact**: The optimal strategies $(λ_0, μ_0)$ are such that $λ_0 = μ_0$.  
*Explanation*: the game is symmetric.

**Fact**: The optimal strategy satisfies $\frac{\partial p}{\partial \lambda}(λ_0, λ_0) = 0$.  
*Explanation*: by the two facts above.

In our symbolic calculus script, we can compute $\frac{\partial p}{\partial \lambda} (\lambda, \lambda)$ with the code 

{% highlight python %}
dp = diff(p, l).subs({m:l})
{% endhighlight %}

We conclude that

$$
    \frac{\partial p}{\partial \lambda} (\lambda, \lambda) = \frac{3\lambda + (-\lambda^3 + 3\lambda - 2) e^\lambda}{6(2\lambda + (\lambda^2 - 2\lambda + 1)e^\lambda - 2)}.
$$

Solving the equation $3\lambda + (-\lambda^3 + 3\lambda - 2) e^\lambda = 0$ numerically, we obtain that $\lambda_0 = 0.416195355$. As a sanity check, we can plot the probability that $R_1$ wins for varying values of $\lambda$ and $\mu$.

The figure below shows the probability that $R_1$ wins, in the case that $R_2$ uses the fixed strategy $\lambda_0$, for different values of $R_1$'s strategy $\lambda$. We see that if $R_2$ is using strategy $\lambda_0$, the strategy that $R_1$ should choose to maximize the probability of winning really is $\lambda = \lambda_0$.

![]({{ page.image1 | relative_url }})

The figure below shows the probability that $R_1$ wins, in the case that $R_1$ uses the fixed strategy $\lambda_0$, for different values of $R_2$'s strategy $\mu$. We see that if $R_1$ is using strategy $\lambda_0$, the strategy that $R_2$ should choose to minimize the probability of losing is $\mu = \lambda_0$.

![]({{ page.image2 | relative_url }})

Finally, we compute the answer to the original question:

$$
    \begin{align*}
        P(S_{\lambda_0} = 0)
        &= F_{\lambda_0}(0) \\
        &= 1 + (h_{λ_0,0} + λ_0 - 1) \exp(λ_0) \\
        &= 1 + (λ_0 - 1) \exp(λ_0) \\
        &= 0.114845886.
    \end{align*}
$$
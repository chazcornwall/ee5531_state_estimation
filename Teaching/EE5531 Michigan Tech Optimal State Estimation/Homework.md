## 1. Does sensor fusion always reduce variance?
In class we discussed the optimal linear fusion of two, one-dimensional, Gaussian information sources is

$$
\hat{x} = \frac{r}{q + r} x_p + \frac{q}{q + r} z
$$

where $x_p \sim \mathcal{N}(0,q)$ and $z \sim \mathcal{N}(0,r)$. Prove 

$$
p \leq \min\{q, r\}
$$

where $p$ is the variance of the estimate.

## 2. Does a Kalman Filter always reduce variance?
What is the relationship between the process noise $q$ and the measurement noise $r$ such that $p[t]$ is marginally stable in a Kalman Filter (Hint: marginal stability implies $0 < \lim_{t \rightarrow \infty} p[t] < \infty$ )? How is this different from Question 1?

## 3. Under what conditions is a Kalman Filter optimal?

a) List at least 6 assumptions made when using a Kalman Filter
b) How are the assumptions from (a) relaxed/changed for a EKF? UKF?

## 4.  Creating an Extended Kalman Filter

Design a discrete-time EKF to estimate the position of a robotic end effector using only measurements of $\theta_1$ and $\theta_2$ (e.g. [Identify $F_t$, $Q_t$, $H_t$, and $R_t$](https://en.wikipedia.org/wiki/Extended_Kalman_filter)).  Assume a constant velocity model such that
$$
\theta_1[t] = \theta_1[t-1] + \omega_1[t-1]\Delta t + \nu_1
$$

and

$$
\theta_2[t] = \theta_2[t-1] + \omega_2[t-1]\Delta t + \nu_2
$$

where $\nu_1 \sim \mathcal{N}(0,\sigma_{\theta_1}^2)$ and $\nu_2 \sim \mathcal{N}(0,\sigma_{\theta_2}^2)$. **Positive angles are clockwise!**

![Freehand Drawing.svg](../../_resources/Freehand%20Drawing.svg)
a) Identify the continuous-time kinematic equations relating joint angles to the end effector position. 
b) Discretize these equations using [Euler Integration](https://en.wikipedia.org/wiki/Euler_method) (also known as a first-order Taylor Series approximation).
c) Ensure these equations are [**Markov**](https://en.wikipedia.org/wiki/Discrete-time_Markov_chain) (i.e. a state at the next time step only depends on states at the current time step). By now, you should have a system of **6** equations describing the **process**. (Hint: You really only need to find 4 because two are given: $\theta_1[t]$ and $\theta_2[t]$)
d) List the states as a large vector $\mathbf{x}$. The system of equations should be expressed as a function $f: \mathbb{R}^6 \rightarrow \mathbb{R}^6$ that inputs the states and produces the state at the next time step.
e) You will now need to repeat steps (a)-(c) to identify the **observation or measurement** equations. These equations should be functions of the states you listed in (d) and return the desired measurement. These equations should be expressed as a function $h: \mathbb{R}^6 \rightarrow \mathbb{R}^n$ where $n$ is the number of observations.
- A subtle assumption made by an EKF is all states are contained in the real numbers. However, the angles $\theta_1$ and $\theta_2$ wrap. This can cause problems when trying to perform measurement updates. For example, if my measurement is $0$ and my current estimate is $359$, the difference is $359$ and not $1$ (this differencing occurs when calculating the **residual**). *Be sure to select the observation equations such that this wrapping error does not occur*. (Hint: $e^{j\theta} = \sin\theta + j\cos\theta$)

f) The equations $f$ and $h$ should not be linear with respect to the states. To linearize them, calculate the Jacobians for both. (Hint:

$$
F[i,j] = \frac{\partial f_i(\mathbf{x})}{\partial x_j}
$$

where $i$ and $j$ denote the row and column of the Jacobian, respectively.)
g) Assuming $\nu_1$ and $\nu_2$ are uncorrelated (i.e. not related to each other), create the **process noise** matrix $Q$ and the **observation noise** matrix $R$ (Refer [here](https://en.wikipedia.org/wiki/Extended_Kalman_filter) for how these matrices are used). Also assume $\nu_1$ and $\nu_2$ are the only process noise sources and all observations have uncorrelated, Gaussian, zero-mean noise with $\sigma^2 = 0.1$.

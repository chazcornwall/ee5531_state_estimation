# Assumptions of KF

* Markovity: 

$$
P(x_t | x_{t-1}) = P(x_t | x_{t-1}, x_{t-2})
$$

* Linearity
* "White", zero-mean, Gaussian, additive noise
	* White implies $E[x(t)x(t-\tau)] = q\delta(\tau)$
* Only aleatoric uncertainty (no epistemic uncertainty)

It's an engineering miracle the Kalman Filter works as well as it does. Any theories as to why it works so well?


# Violations of Assumptions

Draw a box with two sides: 

|Noise/Process Kinematics|Linear|Non-Linear|
|--|--|--|
|Gaussian|KF|EKF/UKF|
|Non-Gaussian|?|Particle Filter


# EKF

Show how to discretize and linearize. Example: Constant velocity unicycle model. We only observe $v$ and $\theta$.

$$
\begin{aligned}
\dot{x} &= v\cos\theta \\
\dot{y} &= v\sin\theta \\
\dot{\theta} &= \omega \\
\dot{v} &= 0
\end{aligned}
$$


# UKF

Since nonlinear functions of Gaussian random variables are not guaranteed to be gaussian, the EKF creates a Gaussian distribution by approximating nonlinear functions with their linear counterparts. 

The UKF leverages the important observation that only mean and covariance are needed in the update and propagate equations. Instead of using linearization to simplify the process equations, the UKF uses a non-parameteric distribution over the state by using different "sigma" points $\mathbf{x}'$ such that


$$
\mathbf{\mu} = E[f(\mathbf{x})] = \sum_{i=1}^Nw_{m}[i]f(\mathbf{x}_i')
$$ 
and
$$
\Sigma = E[(f(\mathbf{x}) - \mathbf{\mu}_x)^2] = \sum_{i=1}^Nw_{c}[i](f(\mathbf{x}_i') - \mathbf{\mu}_x))(f(\mathbf{x}_i') - \mathbf{\mu}_x)^T
$$

# Analysis

How do we determine whether these algorithms are working properly?Do we have any "truth" available for evaluation? -> Residual

$$
\mathbf{y}_t = \mathbf{z}_t - H\hat{\mathbf{x}}_t
$$

We know it is Gaussian, but what is the mean, covariance, and autocorrelation? If we know what they should be, we can use the residual to tune a KF (manually or automatically).

## Residual Mean

$$
\begin{aligned}
E[\mathbf{y}_t] &= E[\mathbf{z}_t] - HE[\hat{\mathbf{x}}_t] \\
&= HE[\hat{\mathbf{x}}_t] - HE[\hat{\mathbf{x}}_t] \\
&= 0
\end{aligned}
$$

If the residual is not zero mean, something is typically wrong with $H$ in your observation equations.

## Residual Covariance

$$
\begin{aligned}
E[(\mathbf{y}_t - E[\mathbf{y}_t])(\mathbf{y}_t - E[\mathbf{y}_t])^T] &= E[\mathbf{y}_t\mathbf{y}_t^T] \\
&= E[(\mathbf{z}_t - H\hat{\mathbf{x}}_t)(\mathbf{z}_t - H\hat{\mathbf{x}}_t)^T] \\
&= E[\mathbf{z}_t\mathbf{z}_t^T - \mathbf{z}_t\hat{\mathbf{x}}_t^TH^T - H\hat{\mathbf{x}}_t\mathbf{z}_t^T+H\hat{\mathbf{x}}_t\hat{\mathbf{x}}_t^TH^T] \\
&= Cov(\mathbf{z}_t, \mathbf{z}_t) - \cancel{E[\mathbf{z}]E[\mathbf{z}^T]} - \cancel{Cov(\mathbf{z}_t, \mathbf{x_t})H^T} + \cancel{E[\mathbf{z}_t]E[\mathbf{x}_t^T]H^T}- \\ 
&\cancel{HCov(\mathbf{x}_t, \mathbf{z}_t)} + \cancel{HE[\mathbf{x}_t]E[\mathbf{z}_t^T]} + \\&HCov(\hat{\mathbf{x}}_t, \hat{\mathbf{x}}_t)H^T - \cancel{HE[\hat{\mathbf{x}}_t]E[\hat{\mathbf{x}}_t^T]H^T} \\
&= R + HPH^T = S
\end{aligned}
$$

If the residual's covariance does not match the predicted covariance (shown in the equation above), noise models ($Q$ and/or $R$) need to be adjusted to match reality. Remember, $Q$ is in $P$.

## Residual Autocorrelation

If I have two vectors $\mathbf{x}$ and $\mathbf{y}$, how do I know one vector can not tell me anything about the other vector? -> Orthogonal!

$$
\begin{aligned}
E[(\mathbf{y}_t - E[\mathbf{y}_t])(\mathbf{y}_{t+\tau} - E[\mathbf{y}_{t+\tau}])^T] 
&= S\delta(\tau)
\end{aligned}
$$

If the Kalman Filter is using all possible information, then a previous residual should be orthogonal to the current residual. This is also known as a **white** signal. If your residual is correlated or non-white, the process equations may not be modeling all time-dependent phenomena. 
Everything up until now was value based method -  learn what every possible action is worth and hope argmax will extract the best policy.
* Difficult to scale to high dimension.
  
Policy gradients: What if we just optimize policy directly to make good actions more likely?
the catch is we can't calculate the distribution analytically so we monte carlo sample
![Pasted image 20261101110116](pictures/Pasted%20image%2020261101110116.png)
We are making an assumption that all the data we collect is taken from the current policy (this is an on-policy method).
Roll out the policy seeveral times, estimate reward and average to estimate how good is the policy.

Since we have  to sample the policy before optimizing (stuck between maximzing rweard and calculating estimation) it's difficult because we don't know the dynamics of the world (need to leanr form experience)

We use the policy gradient theorem (we can't sample gradient of distribution , but we can sample gradient of the log)
![Pasted image 20261101161612](pictures/Pasted%20image%2020261101161612.png)
We can sample trajectories(!), sampling still non differentiable , but we can differentiate wrt theta!
Why this works with log?, because we need probability distribution 
1. Non negative (gradient can be negative)
2. Total area(intagral) equals to 1.
3. original probability p theta is valid probaiblity distribution.
4. derivative of probability p theta is not probability distribution (you cannot roll dice or run experiment on gradient distriubtion)

Notice that the expectation symbol says **$\tau \sim p_\theta(\tau)$**.

We are **not** sampling from $\nabla_\theta \log p_\theta(\tau)$. Instead:

1. **We sample trajectories ($\tau$) from the original, valid distribution $p_\theta(\tau)$**—meaning we just run our agent in the environment normally.
    
2. Once we have those sampled trajectories, we **plug them into** the analytical formula for $\nabla_\theta \log p_\theta(\tau)$ and evaluate it.

In short: **We avoid differentiating through the sampling/environment process by using the sampled trajectories as data to evaluate our policy's gradient instead.**

The reinforce algorithm
![Pasted image 20261101163300](pictures/Pasted%20image%2020261101163300.png)
Intuition for PG,
![Pasted image 20261101165652](pictures/Pasted%20image%2020261101165652.png)

Causality (reward affecting only actions from this point onward and not past actions):
![Pasted image 20261101165929](pictures/Pasted%20image%2020261101165929.png)
High Variance:
getting similar high rewards although one fails and one succedes
We can substract the baseline from the reward to go.![Pasted image 20261101173301](pictures/Pasted%20image%2020261101173301.png)

The reward to go is actually a monte carlo estimate of the Q function!
![Pasted image 20261101173554](pictures/Pasted%20image%2020261101173554.png)

Adantage function:
How much better is it to take action a in state s?
![Pasted image 20261101173726](pictures/Pasted%20image%2020261101173726.png)
If we plug this into the formula is better than our average performence (better, will increase expectation, worse, will decrease expectation.

Vanilla policy gradient is on policy (unlike off policy like DQN that can reuse past data) since we need to create new data with our new updated policy.

Policy graidents off policy
* Importance sampling, instead of sampling the actual distribution (current policy) we will sample a proxy distribution and weigh these samples according to importance.

![Pasted image 20261102113738](pictures/Pasted%20image%2020261102113738.png)


We usually will evaluate over time steps and not only on trajecetories

Now instead of collecting and updating batch of trajectories for graident update (since we update the same policy) like the vanilla one we just mentioned, we can create the trajectories once and update the gradient **several times**.

What we actually do is correcting the distribution shift between the old policy and the new one using the importance sampling mechanism.
![Pasted image 20261102114255](pictures/Pasted%20image%2020261102114255.png)

### The Catch (Why we can't do it _infinitely_)

While importance sampling lets you update multiple times, there is a catch: **if your new policy ($\theta$) drifts too far away from the old policy ($\theta_{\text{old}}$), the importance sampling ratio ($r_t$) becomes unstable.**

The numerator and denominator get too far apart, causing the variance to explode, which can ruin your training. This is why algorithms like **PPO (Proximal Policy Optimization)** use a "clipped" objective—it allows multiple update steps on the same data, but heavily penalizes or blocks the policy if it tries to change too drastically all at once.

Entropy regularization- 
![Pasted image 20261102115545](pictures/Pasted%20image%2020261102115545.png)
H is shannon entropy/continous differential entropy

PG with constraints:

Usually it might not be zero, but the difference between old policy and new policy is big (upper part, new policy, gets high probability and lower part, old policy gets lower proability so it explodes the gradient)
**Solution**:
* Constraint the policy so it won't - KL divergance between new and old policiies (Trust region policy optimization)
![Pasted image 20261102123447](pictures/Pasted%20image%2020261102123447.png)
Problem: to calculate the kL divergance is computatinally heavy (needs fisher information matrix) in o(N3) for trainable parameters, not so popular today

Another **solution** :
![Pasted image 20261102131324](pictures/Pasted%20image%2020261102131324.png)
Proximal policy optimization (PPO) - Clipping the importance ratio - will guarentee no explosion, but is near-on-polcy in term of number of samples(since we don't want to get a big gap between the new and old policy).

Final PPO represnetation:
![Pasted image 20261102145305](pictures/Pasted%20image%2020261102145305.png)
This is an actor-critic method , the actor (first term) uses policy graidents to predict continious actions.
The critic (second term) - uses value learning to evaluate goodness of being in a state

This all gives us low variance from critic ![Pasted image 20261102151837](pictures/Pasted%20image%2020261102151837.png)

![Pasted image 20261102152700](pictures/Pasted%20image%2020261102152700.png)

the key idea is that actor pulls all states from replay buffer but asks the current policy what action it would take here today.
The critic acts as learned simulator and estimates how much this action would reward be in the old policy even though it hasn;t been in this (s, a) pair before.
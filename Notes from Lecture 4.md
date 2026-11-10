* Limitiation on imitation learning - we cannot be better than the expert trajectories (that's why RL is needed)
* ![[Pasted image 20261021145418.png]]

* Although the reward function  might not be continious (failure/success binary) the expectation w.r.t to the policy parameters and actions is smooth
* We  marginalize the stochasitc policy![[Pasted image 20261021151501.png]]
* Convolution of the reward function(step heaviside function!) with the policies' density function (distribution)![[Pasted image 20261021152520.png]]
* How can we avoid unrolling the entire policy until the end (because only then we can do expectation over the reward function)?
	* Value function (conditioned on the state instead of averaged over the different states)
	* it works because the future is depend only on the current state (markovian property) and not the entired history (condition on current state is sufficent to summarize the past)
	*![[Pasted image 20261021160714.png]]
	* How do we calculate this efficiently?
		* Using this trick we can turn inifite series into one step equation (immediate reward + reward starting tomorrow)
		* this is a recursive equation (next state depends on previous state)
		* It means that I don't need to simulate the entire trajectory to evaluate the value, just to calculate the value function  of current state + rewward of next state and it folds the entire process.
		  This is called calculating using the bellman equation (I assume it helps because then we don't have time dependency , and we can parallelize![[Pasted image 20261021161655.png]]
### Why is this a game-changer for RL and Machine Learning?

- **Batching and Parallelization (Your Guess!):** Because you break long, dependent trajectories into individual transition tuples $(s, a, r, s')$, you can throw them all into a **Replay Buffer**. Instead of training sequentially over time, a deep learning model can sample random batches of transitions _in parallel_ on a GPU, completely ignoring the order in which they happened.
    
- **Online Learning:** The agent can learn and update its network _during_ the episode after every single step, rather than waiting for an episode to finish.
    
- **Massive Variance Reduction:** Waiting for a full rollout means accumulating the randomness of hundreds of stochastic actions, leading to high-variance updates. Bootstrapping stabilizes training by leaning on a value function estimate.

Optimal value= for a speciifc state , the best policy-> compresses the policy into one state
optimal policy = averaged on all starting states-> the best policy
The crux is that the optimal value compresses the longi horizon task into one step greedy decision (so long term reasoning is impliciltly encoded in the value function)

One is evaluating over actions at o(s^2*a), the other is evaluated over policies o(s^2) only

![[Pasted image 20261021173439.png]]

![[Pasted image 20261021174630.png]]

Limitaion of V(S) for robotics:
* We need a dynamic model to execute the actions and evaluate which state-action is the best during test time.
* Hard to model and world is messy.
Q- function is the solution
* The idea is to assimilate the dynamic model into the value function during training time (in test time, no need to query)
* The trade off is higher (conditioned on the action and not just the state)

![[Pasted image 20261021182517.png|467]]

(two variables inside the Q function)
![[Pasted image 20261021182525.png|451]]

Qvalue iteration in test time

![[Pasted image 20261021182619.png|446]]

Using the Q function,
Q-learning is a model free algorithm that learns from experience
and it is off-policy
![[Pasted image 20261021190333.png|368]]
![[Pasted image 20261021190322.png|379]]
In Q learning the optimal action is evaluated independely of how the data was collected.

![[Pasted image 20261021191132.png|341]]

in SARSA it will penalize nervous driver (since it knows that next action might be wrong due to exploration), but in Q learning it won't. it will just try to maximize the best next action out of all possible data it has.
SARSA learns safer behaviours.

Deep Q network - 
first network to learn directly from raw pixels (atari games), learning from experience.
two tricks
* Experience replay buffer - store transitions in buffer instead of discarding them -> sample random minibatch each update -> break temporal correlation - i.i.d samples - Each transition is used many times- sample efficient
* Delayed target network - fix a network for the next state and update this network only after a few steps -  increases network stability
* ![[Pasted image 20261029132907.png]]
DQN. works well but it has one central problem which is the overestimation bias.
The over estimation bias is caused due to the "max" operation in the Q learning function. Early in the training it amplifies  errors since according to Jensen's inequality taking the Expection of the max is always bigger than taking the max of the expectations
$$\mathbb{E}\left[\max_a Q(S, a)\right] \ge \max_a \mathbb{E}\left[Q(S, a)\right]$$
Double DQN fixes this
![[Pasted image 20261029141025.png]]


 DQN- scaling continus action spaces
 if we have high dimension of actions (several joints) it becomes even worse.
 to mitigate:
 * Discretization
 * Sampling (monte carlo, try several actions  in a sampling policy) + CEM (Cross entropy method?)
 * Learn ARG max with second network (instead of solving the optimization at test time we move it to the training) -DDPG(?)

Nice Example of discretization:
* take the height map-> evalautes Q value for each pixel in different orientation (16), the argmax is over the map and not over the entire space of actions

![[Pasted image 20261031132953.png]]


Monte carlo is possible (Just sampling different actions, calculating Q and get the max of these)
* Scales poorly (many forward passes)
* Quality depends on N passes (number of samples) and luck (no guarentee)

Cross entropy method is a smarter 
![[Pasted image 20261031165253.png]]
Pros: No gradient needed, work as black box optimizer
Cons: not multimodal, only unimodal gaussian, optimziation cost is at test time NXK can take a lot of time during optimzation at test time.

DDPG - 
based on actor and critic (two networks) - 
![[Pasted image 20261031173657.png]]Instead of optimizing the actions we build a function(network) over the different actions to maximize Q
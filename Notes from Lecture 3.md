
* Adding fake data (more real cameras, but labeling is   different than the original one, fake labeling)  to introduce augmentation to behavioral cloning.



![Pasted image 20261019172853](pictures/Pasted%20image%2020261019172853.png)

* in BC the problem is once we step into unkown state it increases the chance we will make also a mistake in the next one. distribution shift from expert policy increases quadratically!


* Dagger - dataset aggregation, 
  Rollout the policy
  see what states are visited, 
* relabel by expert (correct actions  or add actions) -> add these to daatset
* 
![Pasted image 20261019174440](pictures/Pasted%20image%2020261019174440.png)

Online intervention, only when necessaery

![Pasted image 20261019174627](pictures/Pasted%20image%2020261019174627.png)

* Problem of causal confusion - due to adding history

Since it always minimizes the loss it might not recognize the actual cause for the event (in this case it only takes into account the pressure sensor and not the visual- that it slipped)
![Pasted image 20261019181933](pictures/Pasted%20image%2020261019181933.png)

* In multi modal behaviour standard MSE fails vecause it averages the different modals (different ways to close a drwaer for example)
	* Handle ways:
		* Mixture of gaussians (predefine gaussians)
		* Discretization (discreteise the action) - per dimension discretization with sequence model -> using VLA
		* Diffusion
		* conditional VAE
* Difficulty in evaluating task success - tasks are often continious and not discrete
* Learn policy from Play data - 
	* We will take the last state , encode it (goal feature), add it to the initial state 
	* Do a KL loss between this distribution and the actual distribution of the end state 
	* ![Pasted image 20261020113135](pictures/Pasted%20image%2020261020113135.png)
	* Standard variational trick (like VAE)
	  ![Pasted image 20261020112719](pictures/Pasted%20image%2020261020112719.png)
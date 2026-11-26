World models
* Batter can hit fast balls by anticipating location - our perception allows decide when to hit the ball using an internal world model.
* The basebballer initiate a swing even before the ball reach it's visual location to hit since he knows when to swing w/o visual actual location.![Pasted image 20261110181208](pictures/Pasted%20image%2020261110181208.png)

* World models answer a different action, they try to undestand the actual state of the environment and not just what should we do next as regular vla policies etc.. 
* VLA does not have internal representation of the world, just predicting the next action 
* Why world models? we can predict how the world will look following a specific interaction with the world.
* World models might be thought of data driven simulators.
* The world model 
* ![Pasted image 20261111111223](pictures/Pasted%20image%2020261111111223.png)
* Video model is not a world model p(video| text)

Early world models:
* Pixel AC-WM(Action condition world model) - given an image, how will the next image change (create a flow field for the pixels, not the pixels themselves!, it's much easier)
* Visual MPC (model predictive control) for test time plannig
* ![Pasted image 20261111111836](pictures/Pasted%20image%2020261111111836.png)
* Advantages:
	* Self supervised training on robot videos
	* generalizes to unseen objects due to motion objective
* Disadvantage:
	* Blurry predictions (with L2 loss)
	* Pixel space is expensive to calculate on
	* Prediction accumulates errors
	
* What if we compressed observations into latent space and train a policy inside the learned world?
The original design of the WM consists of three parts:
* Vision model (V) - perception
* Memory RNN model (M) - temporal
* and a Controller (C) that uses both the V and the M model  - Control
* The original paper used MDN (mixture density network) - uses mixture of gaussians for the memory model
* Regarding the control model -  its a single FC layer, maps at=pi(zt,ht)
	* Usually the world model is pretty stochastic so the controller is trained using black box evolutionary optimisation algorithm
		* ![Pasted image 20261111114353](pictures/Pasted%20image%2020261111114353.png)
* The problem with these models that they might drift during the imagination phase
	* Since the VAE and RNN were trained seperately
* ![Pasted image 20261111121219](pictures/Pasted%20image%2020261111121219.png)
* Fix is recurrent state space model (RSSM)
	* Splitting the latent space creation into imagined part(based on memory only, prior part) and trained part(posterior, based on observation) and forcing them to stay close using KL loss
	* Training an RSSM:
	* ![Pasted image 20261111122342](pictures/Pasted%20image%2020261111122342.png)
	* RSMM closed loop inference imagination
	* The world model RNN is deterministc while the imagination part is the stochastic path
	* Specifically, the KL divergence ($D_{KL}(q \parallel p)$) aligns the **imagination prior** ($p(z_t \mid h_t)$) with the **training posterior** ($q(z_t \mid h_t, o_t)$), ensuring that the model's predictions without observations stay grounded in what it learned from real data. this helps to rollout very long runs and avoid accumulated errors![Pasted image 20261111122526](pictures/Pasted%20image%2020261111122526.png)
* Dreamer V1
	* Moves the planning into the imagination part
	* Does not plan from scratch with CEM at each timestep but amortises that planning into a learned policy.
	* Replace CEM with actor critic policy (like we learned in previous lectures)
	* ![Pasted image 20261111142906](pictures/Pasted%20image%2020261111142906.png)
* DayDreamer - (application of DreamerV1)
	* Dreamer on real robots with same hyperparams
	* 1 hour of quadruped interaction -> days in imagination training
	*
* Dreamer models v1-v4 improments for long horizon tasks
*
* What if we want to train the dreamer to more domains ?
	* We need to scale it to entire internet's videos
	* This is where generative video modelling comes into play
	* Problem:
		* Does not scale of we use tokenization of VIT for videos

* Video tokenization compressopn
	* Video is usually very redundant
	* Three compression axes
		* Spatial 
		* Temporal
		* Adaptive
	* Cosmos model has very advanced tokenizer that can compress in an advanced manner
![Pasted image 20261111153442](pictures/Pasted%20image%2020261111153442.png)
Video world model:
Where do actions live?
* AC-WM (dreamer, no video generation)
* WAM  - joint creation of video and action
* VAM action comes after created video (video is frozen backbone)
	* IDM- inverse dynamics model
![Pasted image 20261111163326](pictures/Pasted%20image%2020261111163326.png)

* VLAs: poor sample efficiency
	* VLMs are usualy blind to physical causality and temporal dynamics
		* large scale teleop data necessary ro convert a VLM to VLA.
	* Video backbones (instead of images text pairs like VLM)
		* offload dynamics learning to scalable action-free video data
		* Video gives us dynamics instead of images that hard to explain with them
		* mapping from video is easier because the physics is implied
			* Post training on robot actions is easier.
			* ![Pasted image 20261111165644](pictures/Pasted%20image%2020261111165644.png)
* Policy performence scales with video models
	* policy achieves perfect performence when condition on GT video
	* Finetuning videmo model on robot data improves video prediction quality -> hence improves policy
	* Policy performence scales with better video models
	
* Some models don't have seperation between the world model and the policy
	* Joint video-action prediction, no explicit IDM
	* ![Pasted image 20261111171427](pictures/Pasted%20image%2020261111171427.png)
	* This allows setting GT observation since it works in closed loop and alows to bypass the accumulating errors
* Summary of WAMS vs VAMS
![Pasted image 20261111171633](pictures/Pasted%20image%2020261111171633.png)

* JEPA framework - up until now we used at somepoint decoder to pixels - do we even need pixels?
	* Instead of predicting the next frame, we predict only the embedding of the next frame
* ![Pasted image 20261111172006](pictures/Pasted%20image%2020261111172006.png)

Conclusion:
![Pasted image 20261111172349](pictures/Pasted%20image%2020261111172349.png)
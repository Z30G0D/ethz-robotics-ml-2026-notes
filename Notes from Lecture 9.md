Generalist Robot Policies

In previous lectures we covered 
ACT (Action chunking transformer) and diffusion policies.

The problem is that they are very specialized and harder to scale for other robot embodiments.

The north star is to create a generalist foundation policy for and task any robot and any environment.

We want to develop foundation models similar to NLP.

Ingredients for Generalist models:
Large Datasets - robot data is scarce
Collecting data requires human supervision.

Large models - 
robot data is multimodal and heterogenousm robot needs High frequency Control (need to)

Scalable evaluation - 
Tedious and expensive
Diffcult to reproduce


Large robot data:
X-open embodiment data

Generalizing models can change across sensors, goals and action space (joint , ee, etc..)

Most generalizing models work with robotics as multimodal sequence modeling.
So everything is kind of sequences :)

Architecture of Octo (open source model by course lecturer):
![[Pasted image 20261123203050.png]]

Octo design decisions:
* Gripper -  same labeling for all datasets (Some were -1/+1 and some were +1/0 or some contionious labeling)

How to adapt generlize model to new obs/ action space?

Adaptable pre-trained representation for finetuning - this really helps to transfer from robot to robot.

Cross embodiment learning:
* ![[Pasted image 20261123210543.png]]

"One policy" article Cross embodied transformer
* 5HZ-50HZ control (output) can change dramatically between robots and you need to create a model that works on both of them.
* Does multitask pretraining help post training?
	* ![[Pasted image 20261124133219.png|465]]
	* Read: A careful examination of large behavior models for multitask dexterous manipulations
	* Conclusion: It does help, post training for a single task is more efficient (mostly helpful for low amount of data for your specific task)
		* How you normalize your data influence alot about the actual transfer of knowledge to your robot.
		*
	* Do internet scale priors like VLMs transfer to control?
		* Yes, You could transfer learn VLM to VLA (finetuning on actions instead of text output)
Tips on training a VLA for generalist policies:
* ![[Pasted image 20261124141345.png|374]]
* Reach 95% on token accuracy
	* Action token accuracy is compute on teacher forcing and does not represent the actual performence on actual rollouts
	* If we hand picked the exact action binning correctly
		* L1 and L2 are also good ofc
	* Batching heterogenous data
		* Several embodiments with several different cameras (different setups)
			* Simple Solution - pad the missing cameras or inputs
				* Disadvantage - compute wasted on empty data
			* Sequence packing - technique taiolred to handle variable length transformers
				* Concatenate everything to a single vector
					* Maintain cunulative sequence length variable
					* ![[Pasted image 20261125105856.png]]
					* ![[Pasted image 20261125110005.png|404]]
				* Block diagonal att. so different samples won't attend to other samples, need to maintain this per block
	* Dataloading:
		* Tw main methods:![[Pasted image 20261125110321.png]]
		* The issue with sequential read and shuffle (first method) is that if the buffer is small (compare to the trajectory length) the sampels are very correlated.
			* The solution they used in OCTO architecture is to shuffle and interleave frames (so every trajectory became smaller) before decoding and in that way they could increase the batch size dramatically and avoid correlated batch.
	* Cross embodiment heads
		* Two approaches for different heads![[Pasted image 20261125112331.png]]
		* Disadvantage of crossformer:
			* need to add a new head for a diffferent embodiment
		* Disadvantage םf Pi0:
			* Padding with zeros is a waste, predict for *all* embodiments regardless of their control frequency (this is important because sometimes we don't need prediction in a very high rate, depends on the robot).
				* Since the output is the same size it takes into account the control frequency as constant, and not changing (like crossformer style). 
				* This is wrong since every embodiment has its own frequency control.
		* Q: How does the model know at test time for which embodiment it should output?
		* A: In crossformer style: User specifies head in prompt, in Pi0 style- model infers embodiment from obs, proprio and task(no explicit head selection, just from the input)
		
	* Current VLA recipe
		* Next token predo![[Pasted image 20261125154120.png]]
		* VLM backbone trained on next token prediction: FAST robot actions + web data 
		* Action expert decides about the continious actions, it implements a stop gradient so that it will be able to read from the VLM but cannot update it.
* Scalable Evaluations of robot policies:
	* Evaluating polciies is expensive!
		* People bump into camera, gripper get stuck, real eval is slow and tedious
		* Hardware failures - changing engine can cause issue with training data
	* Sim evals democratize research
	* Sim evals makes it more reproduciable
	* SIMPLER framework helps for evaluating the policy in the real world
* What makes a good simluation
	* Mitigate the control gap between simulated and real world environment to ensure that the policies executed will yield the same results end effector in real world
		* You can do it using system ID (accurate control dynamics per system/environment) - elaboration in underacuated course, - in a nutshell the low level controller in the simulation and in the real world should produce the same output (same output of model causes same output in real life)
		* Minimze visual distribution shift (greenscreen, texture matching etc..)
		* 
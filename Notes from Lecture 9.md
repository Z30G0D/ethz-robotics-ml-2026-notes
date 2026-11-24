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
* Read the paper:
	* A careful examination of large behavior models for multitask dexterous manipulations
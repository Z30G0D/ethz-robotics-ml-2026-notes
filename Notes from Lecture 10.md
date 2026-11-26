Is scaling robot data all we need?
Yes

But it's not the full story.

Consider what data gives us, better priors for behaviour, compresses the distrbution into the model weifghts.

We need Intelligent reasoning (chatgpt , etc...) - reasoning is compostional, not just train on the entire internet.

Can we make a robot that thinks "harder" act better? Can it reason over an observation and act better?

Embodided Chain of Thought reasoning -  instead of having a policy or a VLA, we insert some intermediate reasonning steps.
* What are the next subtask? - reasoning large task into subtasks
	* Using suite of foundation models to generate reasoning data for training
	* Eventually it will trinkle down to actual location of 
![Pasted image 20261125184242](pictures/Pasted%20image%2020261125184242.png)

This really accelrates robot performence with no more data!
![Pasted image 20261125184513](pictures/Pasted%20image%2020261125184513.png)

This also let's use correct the robot since we have intermediate reasonig tokens and we can correct it (since it's interpertable).

The reasoning generalizes pretty well to unseen robots.
This might help to train robots
* Train robots VLA with reasoning, improves generalization , interactivity and robustness
* Inference cost of reasoning is high.
	* Open VLA runs in 4 actions per second
	* Embodied COT is 4 seconds per action(!)
* Can we get the benefits of reasoning w/o the high inference costs?

* Before that: Why does embodied reasonig help?
	* Better representation learning
![Pasted image 20261125193928](pictures/Pasted%20image%2020261125193928.png)

	* Imrpoved learning curriculum
* ![Pasted image 20261125193958](pictures/Pasted%20image%2020261125193958.png)
	* Increased policy expressivity - This was showed to be meaningful in LLMs.
![Pasted image 20261125194447](pictures/Pasted%20image%2020261125194447.png)

Reasoning pretraining and dropout paradigm
* The reasoning pretraining, two trainingstages
	* Take the VLM and finetine only for the reasoning (text)
	* Next, you finetune to robot actions![Pasted image 20261126113128](pictures/Pasted%20image%2020261126113128.png)
* Reasoning dropout:
	* Similar to previous one but you dropout the reasoning, the actions are not dependent on the reasoning (you can turn it on or off during the training)
	  ![Pasted image 20261126113217](pictures/Pasted%20image%2020261126113217.png)
* This helps:
	* Higher success rate than standard VLA
	* Lower Success rate than Embodided CoT
	* But comparable inference rate to standard VLA (unlike Embodied CoT that is very slow)
* ![Pasted image 20261126115234](pictures/Pasted%20image%2020261126115234.png)
* Summary for which should i use when:
* Unpaired reasoning means that the action is not paired with reasoning (not labeled, external reasoning source)
![Pasted image 20261126115826](pictures/Pasted%20image%2020261126115826.png)

* Test time compute scaling
	* Can we improve if we think longer in test time?![Pasted image 20261126120714](pictures/Pasted%20image%2020261126120714.png)
	* Best example is the canonical example of alphago:
	* The take away from the next diagram is that test-time search is essential in surpassing human performence, you need to increase  the model 100Kx to get to 5000 ELO points if you don't use test-time compute![Pasted image 20261126120754](pictures/Pasted%20image%2020261126120754.png)
	* Does this test time scaling works in Language as well?
		* Yes! - better to train a smaller model and let it think longer than just train a big model.
		* SFT on failed attempts +learned verifier - model learns produce better answer given previous wrongs ones, verifier picks best one
		* Generate multiple answer at test time, select best with verifier.
		* But for hard questions, no more matter how much we will let it think, it still won't recover the actual answers.![Pasted image 20261126155846](pictures/Pasted%20image%2020261126155846.png)
		* Reasoning tokens help since it effectively increases the model's number of steps taken before outputting an answer, if you immediately output the answer from the model it can handly handle maximum number of steps as the Transformer is bound to (constant size transformer generating o(T) reasoning tokens )
	* CoT is good only on large scale model, the capabilities emerge only on big models.
	* Original CoT prompting = few shot/in-context learning
		* Prompt sensitive, brittle
		* Requires human input
		* Doesn't generalize beyond the examples
	* Can we reason internally inside the model and not with human prompts?
		* ![Pasted image 20261126161737](pictures/Pasted%20image%2020261126161737.png)
	* Learning to reason with RL - GRPO (closely related to PPO)
		* The critic is quite expensive and difficult to train.
		* Model learns by comparing its own answer relative to each other (group relative policyi optimization)
		![Pasted image 20261126162925](pictures/Pasted%20image%2020261126162925.png)
		* Just using binary verifiable answers, reasoning emrges since we need to learn think carefully about the problem (all relative answers should be good comparing to the final verfiable answer hence larger reward).
	
	* Why didn't this work before?
		* Base lLM needs to be good enough to procuce any correct rollouts to reinforce.
		* What changed in moden LLM is that they crossed the capability that they can solve hard problems on their own.
		* Once this happened, RL takes over and systematically amplify that capability (since it has basic relative good answers).
	* Reasoning behaviours emerge from RL
		* At some point during the training there is an increase in "reflective words" and "wait" to reconsider the actual answer, this is not part of the Supervised data but an actual emerging capability of the model![Pasted image 20261126164628](pictures/Pasted%20image%2020261126164628.png)
		* Does this RL reasoning apply too in vision?
			* Yes, many papers.
			* Visual Reinforicenemt fine tuning.![Pasted image 20261126165109](pictures/Pasted%20image%2020261126165109.png)
			* The GRPO framework is completely agnostic to what is the verifier as long as we can apply the verifier into it.
			* The problem that in vision tasks hallucinated answer cannot be verified by final answer (like math) since the verifier  only sees and reward is given on the final answer, and not the entire reasoning trace. - hallucinated reasoning and correct answers aren't mutually exclusive.
			* Paper ARGOS- rewards not just the final answer but also the reasoning steps
				* test-time reasoning helps for high level planning
				* 3-4x better than supervised CoT on the hardest task
* ![Pasted image 20261126171009](pictures/Pasted%20image%2020261126171009.png)
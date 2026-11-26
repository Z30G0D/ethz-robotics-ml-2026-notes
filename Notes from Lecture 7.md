Two limitations of reactive policies (single frame, single prediction):
1. Long term memory (what did i do 10 sec ago) , estimating speeds of other objects etc..
2. Actions smoothnes, less jerkiness - single step model produce jerky actions

Autoregressive models
* Any joint distribution can be composed of product of conditional distributions.![Pasted image 20261108180857](pictures/Pasted%20image%2020261108180857.png)
How do we model these distributions?
* RNNs limitations (explidng gradients etc..)


* Transformers - Attention- gives us direct relationship between words regardless of their position (don't need to go through the entire sequence)
* Transformer is like soft KV matching (not hard, but gives contnious space )
* Positional encoding is important since we need order in the robot
* Tokenization for robotics - 
	* Byte pair encoding - compression factor scales with corpus size
Engineering improvements today
* Decoder only 
* Extending BPE to the byte level-> no unkown tokens)emojis for example)
* Rope -  another solution
* flash attention - o(n) memory instead of 0(n^2) by tiling computation in fast SRAM, never materializing to the full attention matrix

* The bitter lesson - scaling methods is better than the one that has better hand crafted approaches
* Scaling laws - chinchilla paper showed that for fixed compute budget we have optimal data size and model size
* We can extrapolate performence - increase model size with compute size and extrapolate how much a specific validation loss will cost us
![Pasted image 20261109130759](pictures/Pasted%20image%2020261109130759.png)

How can we extend LLMs and transformers to vision?
* An image is worth 16 * 16 words article - transformers for image recognition

How does the transformer know which text tokens correspond to which visual token
* CLIP - Contrastive learning in pairs
	* Cosine similarity between text and image tokens

![Pasted image 20261109133539](pictures/Pasted%20image%2020261109133539.png)

Llava:
* CLIP was about creating a shared embedding space and learn their modalities
* Llava - how do we inject image information directly to LLM so it can reason about the images through the LLM?
	* It prepends image tokens to the text sequence, LLM processes everything in single unfied self-attention pass
	* ![Pasted image 20261109133832](pictures/Pasted%20image%2020261109133832.png)

Native multimodal models:
* Instead of adding vision to an existing LLM just train everything together.
* ![Pasted image 20261109145750](pictures/Pasted%20image%2020261109145750.png)
vlm taxonomy
![Pasted image 20261109153749](pictures/Pasted%20image%2020261109153749.png)
The tradeoff is flexibility vs. scalability (how modular is the model's architecture vs. how specific and compute efficient its training at sacle).
* Early and late fusion have already trained LLM and you can incorporate to them a vision encoder directly in the input or in the cross attention mechanism.
*
Extending to robots:
* How can we extend this ?
	* Actions are continious but not discrete
		* RT2 and OPEN VLA  - 
			* Clip using the quantile (1 and 99)
			* Bin to 256 bins
			* N-dim action at time step t (N discrete tokens)
			* inject action tokens into pre-trained language model vocabulary - > train with next token prediction
	![Pasted image 20261109163342](pictures/Pasted%20image%2020261109163342.png)

* How  do we connect these action embedding to the LLM?
	* Override the least frequent tokens in vocabulary (emojis or chinese letters) with new action tokens
	
* Action chunking - solves the jittering (smoothnes we stated)
	* Predict the next k actions and not a single action
	* Temporal ensamble - in order to avoid the large gap of time where the robot is blind (full 4 time slots in this example)
	* these smoothness mechanism are important especially for dexterity needed tasks.![Pasted image 20261109165159](pictures/Pasted%20image%2020261109165159.png)
* Naive action tokenization is a problem
	* As frequency increases:
		* Information per token decreases(they hardly change)
		* Correlation between token increases 
* The solution is a compression, (capturing the meaningful representation of the tokens if they are very correlated to one another)
	* BPE not good, works on discrete signal (not continious )
	* VQ learned compression complex to train
	* solution: 
		* Discrete cosine transform (JPEG like)
* ![Pasted image 20261109172125](pictures/Pasted%20image%2020261109172125.png)
* This improves dramatically the results of high frequency manipulations
*
* Summary, Bitter lesson - find right representation , use scalable architecture, let the data do its thing 
# ETH robotics lecture notes

Personal study notes from an ETH robotics / robot-learning course. The notes run from lecture 3 through lecture 7 and follow one thread: how a robot goes from copying an expert, to learning a value or a policy, to generating actions, and finally to vision-language-action models.

These are working notes, written while watching the lectures. Wording is informal, and the figures are screenshots pasted from the slides. The repo is meant to stay private.

Course exercises live in a separate repository: [Z30G0D/ethz-robotics-ml-2026](https://github.com/Z30G0D/ethz-robotics-ml-2026).

## What is in here

| Path | What it is |
| --- | --- |
| `Notes from Lecture 3.md` … `Notes from Lecture 7.md` | One file per lecture |
| `pictures/` | Slide screenshots the notes embed (Obsidian-style `![[Pasted image ….png]]` links) |

The text itself is small. Almost all of the size is the images in `pictures/`.

## Lectures

**Lecture 3 — Imitation learning.** Behavioral cloning, why distribution shift compounds, DAgger and online intervention, causal confusion, and why a single mean-squared-error action fails when the expert is multimodal (mixture of Gaussians, discretization, diffusion, conditional VAEs). Closes with learning a policy from play data.

**Lecture 4 — Value-based reinforcement learning.** Why the expected return is smooth even when the reward is binary, the Bellman equation and why it lets you train on transitions instead of full rollouts, V versus Q, Q-learning versus SARSA, DQN (replay buffer and target network), overestimation and Double DQN, and continuous actions via discretization, the cross-entropy method, and DDPG.

**Lecture 5 — Policy gradients.** Optimizing the policy directly: the policy gradient theorem, REINFORCE, reward-to-go and the advantage, on-policy versus off-policy updates, importance sampling, and why the policy cannot move too far in one step. Ends with TRPO’s KL constraint and PPO’s clipped objective, read as an actor-critic method.

**Lecture 6 — Generative models for actions.** Why a robot policy has to represent many ways of doing the same thing. Autoencoders, VAEs and posterior collapse, VQ-VAE, diffusion (DDPM training, DDIM sampling, classifier-free guidance) applied as a diffusion policy, then flow matching, rectified flow, and the Pi0 sampling schedule.

**Lecture 7 — Sequence models and robot foundation models.** Memory and smooth actions, transformers and tokenization, scaling laws, CLIP and LLaVA, early versus late fusion, then action tokens (RT-2, OpenVLA), action chunking, and compressing high-frequency actions with a discrete cosine transform.

## How to read the notes

Open the lecture files in [Obsidian](https://obsidian.md), or any Markdown viewer that resolves `![[…]]` image links against `pictures/`. A plain GitHub preview will show the text and will not inline those wiki-style images.

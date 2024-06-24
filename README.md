
# Privacy/Security Ideas Brainstorm

* In general, privacy stuff needs to have more theoretical guarantees. However, if we can't guarantee this, we definitely need to show concrete, experimental results that show that it works well in practice (ie, showing attacks and how to defend, etc.)
* Some main methods for privacy preserving methods are below:

## Related Work
### Homomorphic Encryption
* Main idea - privacy preserving inference - langauge model needs to do language modeling using only encrypted *ciphertext*
* Many operations which are non-polynomial (or nonlinear?) are hard to compute on cipher data, so we need to approximate them
*  This paper [(THE-X: Privacy-Preserving Transformer Inference with Homomorphic Encryption)](https://aclanthology.org/2022.findings-acl.277.pdf) does this with transformers
* Niloofar: some previous work (ie, [Falcon](https://arxiv.org/pdf/2308.13189), but it seems people have not pursued this as much since the 2020s?)

### Approximative Encrpytion 
#### Lattice Gen - Give the langauge model at the i^{th} decoding step a *lattice* of generations (ie, mix in the real token, and fake tokens) so that the language model doesn't see what we are really decoding
* But doesn't this mean that we have more and more expensive compute at every token? Otherwise the language model will know based on the previous context which token we used....
	* Example?
		* Iteration 0: **a** is start token, **b** is injected token
		* Iteration 1 \<bos> **a** -> c, d;  \<bos> b -> e, f; Choose c and f
		* Iteration 2  \<bos> **a**, **c**,  \<bos> **b**,**f** 
	* EDIT: Niloofar mentioned that the cost is only N since we do pruning - otherwise cost would be exponential 
		* Running on GPT3/GPT4 ---> N more inference calls = N more cost?
* Why creative writing task only?
* Paper has attack details with beam search - other ideas for attacks with fluency?
* Paper link [here](https://aclanthology.org/2024.findings-naacl.171.pdf)

#### Preempt - Sanitizing Sensitive Prompts for LLMs
* Radially "sanitizes" sensitive information in prompts before feeding it into LMs
    * Helpful since the LM does not actually see the data, but instead gets a more obfuscated (but still related) version
* Issue: what if we want to do a computation or ask something about a piece of sensitive information? Ie, calculate the sum of my SSN
    * Niloofar: This paper (and other work) define different PII which they assume are sensitive and should never be revealed.
    * Homomorphic encryption (and other privacy methods) remove the need to define such PII - we assume the LM will not be able to see **any** of the data, so we don't have to choose what data to obfuscate/replace
* [Paper link](https://www.cs.toronto.edu/~dglukhov/Preempt.pdf)

#### MEMORIZATION FOR GOOD: ENCRYPTION WITH AUTOREGRESSIVE LANGUAGE MODELS
* Need to read main ideas of this paper more
* Niloofar: Claims to be differentially private but it is an approximation - we should be careful to make such claims
* [Paper link](https://arxiv.org/pdf/2305.10445)

#### Split Learning: Distributed deep learning without sharing raw data
* Client trains a *partial* network to a specific layer (the "cut layer", then sends this off to the server, which trains, the rest of the network
    * Question: How private is this? Could we have an attack where we extract data from the hidden states?
* [Paper link](https://arxiv.org/pdf/1812.00564); [Another link](https://www.media.mit.edu/projects/distributed-learning-and-collaborative-learning-1/overview/#:~:text=Split%20learning%20naturally%20allows%20for,detailed%20information%20about%20the%20model)

#### Shredder: Learning Noise Distributions to Protect Inference Privacy
* Learns to add noise to data sent to server to protect it
* *Where* we inject noise can affect how much privacy we ensure; the deeper the cutting point, the more privacy.
* [Paper link](https://dl.acm.org/doi/pdf/10.1145/3373376.3378522)

## Other/Main ideas - homomorphic encrpytion is slow/other privacy preserving techniques. Can we somehow accelerate it?
Three main ideas:

### Other approximations to preserve privacy but make the process faster
* Niloofar: Cryptographers and mathematicians have been trying to do this for a long time! This is really hard.	

### Using a combination of smaller and larger models (ala speculative sampling) for privacy stuff. Homormophpic speculative sampling? (https://arxiv.org/abs/2302.01318) 

* Niloofar: Neural architecture search was popular in 2020, looking for architectures that work well with HE - why hasn't it been revisited?
* Niloofar: Better distillation and neural architechture search - in small HE model
    * Maybe we can distill from existing models?
    * Or maybe we can use a mixture of experts - capable, smaller-scale HE to ensure 1) speed and 2) privacy

### Using encryption only on the parts that need encryption

* Niloofar: Intuitively makes sense but might not be the most exciting direction - but there are engineering challenges in how we 1) detect the sensitive parts 2) split queries and 3) allocate model sizes

#### Other threads we discussed
* Niloofar: homormophic encryption for SSM? Could be a low hanging fruit to try
* Precedent for accelerating encrpytion-based methods: [Cloak](https://dl.acm.org/doi/pdf/10.1145/3442381.3449965) identifies most necessary subset of features needed for prediction
# Other brainstorm 

### Membership inference attack - Do Membership Inference Attacks Work on Large Language Models? (https://arxiv.org/pdf/2402.07841)
* Membership inference attack (MIA): Is this datapoint a member of my model's training data?
* Tests how MIA works on different model sizes
* More epochs = more effective MIA
* Niloofar: Pythia (https://arxiv.org/abs/2304.01373) models may not be good to test on, since they only train on 1 epoch. Maybe try OLMo models (https://arxiv.org/abs/2402.00838)? 

#### Encrpytion - on Training Side
* Recent work ([MemDPT: Differential Privacy for Memory Efficient Language Models](https://arxiv.org/pdf/2406.11087)) proposes training adapters like LoRA in a differentiably private manner - basically defense against MIA - we train so that we can't uncover the source data
* Related work ([https://arxiv.org/pdf/2305.15594](Flocks of Stochastic Parrots: Differentially Private Prompt Learning for Large Language Models)) shows MIA are good at uncovering prompts, and develop a method to protect against this

### Follow ups
* Language models might have specific memorization budget - what type of stuff do they memorize? Does it fit distributionally with other similar data? How long does it take for these effects to show up? Lots of interesting questions here
* Niloofar: Hard to do experiments with these but valuable since we might see different behavior in <1B, <7B, and >=7B scale models
* Related paper in vision: [The Privacy Onion Effect: Memorization is Relative](https://arxiv.org/abs/2206.10469) - do we observe something like this in language?
* Related paper looking at memorization across model scales: [Memorization Without Overfitting: Analyzing the Training Dynamics of Large Language Models](https://arxiv.org/pdf/2205.10770)

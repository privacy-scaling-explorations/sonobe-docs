# Folding schemes and Sonobe

## Folding schemes overview

Folding schemes are a family of SNARKs for iterative computations, allowing to prove that a function $F$ applied $n$ times to an initial input $z_0$ results in $z_n$.

<img src="imgs/folding-main-idea-diagram.png" style="width:70%;" />

Where $w_i$ are the external witnesses used at each iterative step.

In other words, it allows to prove efficiently that $z_n = F(...~F(F(F(F(z_0, w_0), w_1), w_2), ...), w_{n-1})$.

<br>

The next 3 videos provide a good overview on the folding shcmes ideas:
- In [this presentation](https://www.youtube.com/watch?v=Jj19k2AXH2k) (5 min) Abhiram Kothapalli explains the main idea of Nova folding scheme.
- In [this presentation](https://youtu.be/IzLTpKWt-yg?t=6367) (20 min) Carlos Pérez overviews the features of folding schemes and what can be build with them.
- In [this presentation](https://www.youtube.com/watch?v=SwonTtOQzAk) (1h) Justin Drake explains the folding schemes and Nova concepts.


## Sonobe overview

Sonobe is a folding schemes modular library to fold R1CS instances in an Incremental Verifiable computation (IVC) style. It also provides the tools required to generate a zkSNARK out of an IVC proof and to verify it on Ethereum's EVM.

The development flow using Sonobe looks like:

1. Define a circuit to be folded
2. Set which folding scheme to be used (eg. Nova)
3. Set a final decider to generate the final proof (eg. Spartan over Pasta curves)
4. Generate the the decider verifier

![](imgs/sonobe-lib-pipeline.png)

The folding scheme and decider used can be swapped respectively with a few lines of code (eg. switching from a Decider that uses two Spartan proofs over a cycle of curves, to a Decider that uses a single Groth16 proof over the BN254 to be verified in an Ethereum smart contract).

Complete examples can be found at [sonobe/folding-schemes/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/folding-schemes/examples)

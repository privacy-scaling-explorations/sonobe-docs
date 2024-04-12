# Folding schemes and Sonobe

## Folding schemes overview

A folding scheme is a protocol that can help build incrementally verifiable computation (IVC). This is particularly interesting in the case of iterative computations. An IVC allows to prove that a function $F$ applied $n$ times to an initial input $z_0$ results in $z_n$.

<p align="center">
    <img src="imgs/folding-main-idea-diagram.png" style="width:70%;" />
</p>

Where $w_i$ are the external witnesses used at each iterative step.

In other words, it allows to prove efficiently that $z_n = F(...~F(F(F(F(z_0, w_0), w_1), w_2), ...), w_{n-1})$.

<br>

The next 3 videos provide a good overview of folding schemes:
- In [this presentation](https://www.youtube.com/watch?v=Jj19k2AXH2k) (5 min) Abhiram Kothapalli explains the main idea of Nova folding scheme.
- In [this presentation](https://youtu.be/IzLTpKWt-yg?t=6367) (20 min) Carlos Pérez overviews the features of folding schemes and what can be built with them.
- In [this presentation](https://www.youtube.com/watch?v=SwonTtOQzAk) (1h) Justin Drake explains what a folding scheme is and Nova-related concepts.

## Sonobe overview

Sonobe is a modular folding schemes library. It allows developers to fold R1CS instances in an Incremental Verifiable computation (IVC) style. It also provides tools required to generate a zkSNARK out of an IVC proof. Developers can configure sonobe so that those proofs can also be verified on Ethereum's EVM.

The development flow using Sonobe looks like:

1. Define a circuit to be folded. This is done using a frontend such as [`circom`](https://github.com/iden3/circom) or [arkworks](https://github.com/arkworks-rs/r1cs-std).
2. Set which folding scheme to be used (eg. Nova).
3. Set a final decider to generate the final proof (eg. Spartan over Pasta curves).
4. Generate the decider verifier.

<p align="center">
    <img src="imgs/sonobe-lib-pipeline.png" style="width:70%;" />
</p>

The folding scheme and decider used can be swapped respectively with a few lines of code (eg. switching from a Decider that uses two Spartan proofs over a cycle of curves, to a Decider that uses a single Groth16 proof over the BN254 to be verified in an Ethereum smart contract).

Complete examples can be found at [sonobe/folding-schemes/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/folding-schemes/examples).

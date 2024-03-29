# Usage

## Folding schemes overview
(wip)
<!-- [introductory text here (TODO)] -->

<img src="../imgs/folding-main-idea-diagram.png" style="width:70%;" />

[...] [this presentation](https://youtu.be/IzLTpKWt-yg?t=6367), where [Carlos Pérez](https://twitter.com/CPerezz19) overviews the features of folding schemes and what can be build with them.


## Sonobe overview
<!-- TODO explain the idea of sonobe, being a modular library to use different folding schemes -->
Suppose that the user inputs a circuit that follows the IVC structure, chooses which Folding Scheme to use (eg. Nova), and which Decider (eg. Spartan over Pasta curve).

Later the user can for example change with few code changes the Folding Scheme being used (eg. switch to ProtoGalaxy) and also the Decider (eg. Groth16 over bn254), so the final proof can be verified in an Ethereum smart contract.

![](../imgs/sonobe-lib-pipeline.png)

Complete examples can be found at [sonobe/folding-schemes/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/folding-schemes/examples)

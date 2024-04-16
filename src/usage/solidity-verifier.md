# Solidity verifier

Having used the `Decider` from `decider_eth.rs`, we can now verify it in Ethereum's EVM.

First we need to generate the Solidity contracts that verify the Decider proofs. Use the [solidity-verifiers-cli](https://github.com/privacy-scaling-explorations/sonobe/tree/main/cli) tool
```
> solidity-verifier-cli -p nova-cyclefold -d ./folding-verifier-solidity/assets/G16_test_vk_data
```

# Solidity verifier

Having used the `DeciderEth` (see [Onchain Decider](#Onchain-Decider) section), we can now verify it in Ethereum's EVM.

First we need to generate the Solidity contracts that verify the Decider proofs. Use the [solidity-verifiers-cli](cli) tool
```
> solidity-verifier-cli -p nova-cyclefold -d ./folding-verifier-solidity/assets/G16_test_vk_data
```

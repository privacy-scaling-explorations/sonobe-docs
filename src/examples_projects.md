# Examples and Projects

> This section contains examples and projects that use Sonobe. You can find isolated examples too in the dir [sonobe/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/examples).

- [sonobe-btc](https://github.com/dmpierre/sonobe-btc): implementation of an on-chain Bitcoin light client leveraging Sonobe: uses nova to verify bitcoin's proof of work over 100k blocks and groth16 to land the zkSNARK IVC proof on chain.
- [hash-chain-sonobe](https://github.com/arnaucube/hash-chain-sonobe): example using Sonobe & Circom circuits, proving chains of Sha256 and Keccak256 hashes.


## Papers

- [Mova: Nova folding without committing to error terms](https://eprint.iacr.org/2024/1220):  a folding scheme for R1CS instances that does not require committing to error or cross terms. It is implemented on top of Sonobe code base, and used for benchmarks, see their repo [here](https://github.com/NethermindEth/sonobe/tree/paper).
- [Eva: Efficient IVC-Based Authentication of Lossy-Encoded Videos](https://eprint.iacr.org/2024/1436): cryptographic protocol for authenticating lossy-encoded videos. Is implemented on top of Sonobe code base.

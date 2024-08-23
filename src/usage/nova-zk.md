# Nova's Zero-Knowledge Layer

In its seminal paper, [Nova](https://eprint.iacr.org/2021/370) detailed how to achieve zero-knowledge using an ad-hoc zk-SNARK ([Spartan]()). This involved compressing and hiding a proof of knowledge of a valid IVC proof. Indeed, an IVC proof is not zero-knowledge by default: the prover must provide an IVC verifier with witness values to verify the correctness of their IVC computation.

However, having to modify and use a special-purpose zk-SNARK for hiding and compressing a Nova IVC proof was somewhat inefficient. Fortunately, a subsequent [update](https://eprint.iacr.org/2023/573) (section D.4) to the [Nova](https://eprint.iacr.org/2021/370) paper specified a design for generating zk-IVC proofs. This allows an IVC prover to send an IVC verifier a zero-knowledge proof that hides the values attesting to the correctness of an IVC computation.

# Use Cases

Nova's zk-IVC proof generation is quite efficient, as it simply involves blinding private field elements with randomly sampled values. It can be applied to a variety of use cases.

Firstly, Nova's zero-knowledge layer allows users to compute folds and send a zk-IVC proof to any IVC verifier while blinding witness values. This is useful in itself; however, the proof will not be succinct. 

Nova's zero-knowledge layer can also be used to delegate the compression of an IVC proof to a more powerful but untrusted server. With this approach, a user can blind witness values attesting to the correctness of their computation and send a zk-IVC proof, which a server will "compress" using the SNARK of its choice.

Finally, users can leverage Nova's zero-knowledge layer to delegate both folding and compression to an untrusted server. This is an interesting use case, as it could enable an efficient, tree-like folding version of Nova. See [this issue](https://github.com/privacy-scaling-explorations/sonobe/issues/136) on the Sonobe repository.

# Example

It is straightforward to obtain a zk IVC proof from the nova computation you just performed: 

```rust
// where `&nova` is of type `Nova<...>`
// returns a `Result`, containing a zk IVC nova proof if successful
let proof = RandomizedIVCProof::new(&nova, &mut rng);
```

See [here](https://github.com/privacy-scaling-explorations/sonobe/blob/7097c001fc876578be2229d3590d506858bc0069/folding-schemes/src/folding/nova/zk.rs#L285) for a small demo on how to generate and verify a zk IVC nova proof.

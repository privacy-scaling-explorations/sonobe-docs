# Nova's Zero-Knowledge Layer

[Nova](https://eprint.iacr.org/2021/370) shows how to achieve zero-knowledge at the Decider level using an ad-hoc zk-SNARK ([Spartan](https://eprint.iacr.org/2019/550) in the original Nova paper). This involves compressing and hiding a proof of knowledge of a valid IVC proof. Indeed, an IVC proof is not zero-knowledge by default: the prover must provide an IVC verifier with witness values to verify the correctness of their IVC computation.

The updated version of the [HyperNova](https://eprint.iacr.org/2023/573) paper (section D.4), specifies a design for generating zk-IVC proofs which also applies to the Nova IVC. This allows an IVC prover to send an IVC verifier a zero-knowledge proof that hides the values attesting to the correctness of an IVC computation.

# Use Cases

Nova's zk-IVC proof generation is quite efficient, as it simply involves blinding private field elements with randomly sampled values and folding the instances with randomized instances. It can be applied to a variety of use cases.

We identify 3 main interesting places to use the nova zk-layer: one before all the folding pipeline (Use-case-1), one at the end of the folding pipeline right before the final Decider SNARK proof (Use-case-2), and a third one for cases where compressed SNARK proofs are not needed, and just IVC proofs (bigger than SNARK proofs) suffice (Use-case-3):

- Use-case-1: at the beginning of the folding pipeline, right when the user has their original instance prior to be folded into the running instance, the user can fold it with the random-satisfying-instance to then have a blinded instance that can be sent to a server that will fold it with the running instance.
  - In this one, the user could externalize all the IVC folding and also the Decider final proof generation to a server.
- Use-case-2: at the end of all the IVC folding steps (after n iterations of nova.prove_step), to 'blind' the IVC proof so then it can be sent to a server that will generate the final Decider SNARK proof.
  - In this one, the user could externalize the Decider final proof generation to a server.
- Use-case-3: the user does not care about the Decider (final compressed SNARK proof), and wants to generate a zk-proof of the last IVC state to an IVC verifier (without any SNARK proof involved). In this use-case, the zk is only added at the last IVCProof. Note that this proof will be much bigger and expensive to verify than a Decider SNARK proof.

The current implementation available in Sonobe covers the Use-case-3.
Use-case-1 can be achieved directly by a simpler version of the zk IVC scheme skipping steps and implemented directly at the app level by folding the original instance with a randomized instance (steps 2,3,4 from section D.4 of the [HyperNova](https://eprint.iacr.org/2023/573.pdf)
paper). And the Use-case-2 would require a modified version of the Decider circuits.


# Example (use case 3)

It is straightforward to obtain a zk IVC proof from the nova computation by: 

```rust
// where `&nova` is of type `Nova<...>`
// returns a `Result`, containing a zk IVC nova proof if successful
let proof = RandomizedIVCProof::new(&nova, &mut rng);
```

See [here](https://github.com/privacy-scaling-explorations/sonobe/blob/7097c001fc876578be2229d3590d506858bc0069/folding-schemes/src/folding/nova/zk.rs#L285) for a small demo on how to generate and verify a zk IVC nova proof for the described use-case 3.

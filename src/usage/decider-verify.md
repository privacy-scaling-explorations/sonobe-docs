# Decider verify
We can now verify the Decider proof

```rust
// this is the same that we defined for the prover
type DECIDER = Decider<
    Projective,
    GVar,
    Projective2,
    GVar2,
    CubicFCircuit<Fr>,
    KZG<'static, Bn254>,
    Pedersen<Projective2>,
    Groth16<Bn254>,
    NOVA,
>;

let decider_vp = (g16_vk, kzg_vk);
let verified = DECIDER::verify(
    decider_vp, nova.i, nova.z_0, nova.z_i, &nova.U_i, &nova.u_i, proof,
)
.unwrap();
assert!(verified);
```

In the Ethereum Decider case, we can generate a Solidity smart contract that verifies the proofs onchain. More details in the [next section](solidity-verifier.md).

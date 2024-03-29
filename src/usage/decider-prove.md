# Decider prove

Two options:

- onchain (Ethereum's EVM) mode
- offchain mode

Once we have been folding our circuit instances, we can generate the *"final proof"*, the Decider proof.


#### Onchain Decider

![](../imgs/decider-onchain-flow-diagram.png)

Generating the final proof (decider), to be able to verify it in Ethereum's EVM:

```rust
type DECIDER = Decider<
    Projective,
    GVar,
    Projective2,
    GVar2,
    CubicFCircuit<Fr>,
    KZG<'static, Bn254>,
    Pedersen<Projective2>,
    Groth16<Bn254>, // here we define the Snark to use in the decider
    NOVA,           // here we define the FoldingScheme to use
>;

// generate Groth16 setup
let circuit = DeciderEthCircuit::<
    Projective,
    GVar,
    Projective2,
    GVar2,
    Pedersen<Projective>,
    Pedersen<Projective2>,
>::from_nova::<CubicFCircuit<Fr>>(nova.clone());
let mut rng = rand::rngs::OsRng;

let start = Instant::now();
let (pk, vk) =
    Groth16::<Bn254>::circuit_specific_setup(circuit.clone(), &mut rng).unwrap();
println!("Groth16 setup, {:?}", start.elapsed());

// decider proof generation
let decider_pp = (poseidon_config.clone(), g16_pk, kzg_pk);
let proof = DECIDER::prove(decider_pp, rng, nova.clone()).unwrap();

// decider proof verification
let decider_vp = (poseidon_config, g16_vk, kzg_vk);
let verified = DECIDER::verify(decider_vp, nova.i, nova.z_0, nova.z_i, &nova.U_i, &nova.u_i, proof).unwrap();
assert!(verified);
```

As mentioned above, complete examples can be found at [sonobe/folding-schemes/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/folding-schemes/examples)


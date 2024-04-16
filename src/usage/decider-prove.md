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

let (pk, vk) =
    Groth16::<Bn254>::circuit_specific_setup(circuit.clone(), &mut rng).unwrap();

// decider proof generation
let decider_pp = (poseidon_config.clone(), g16_pk, kzg_pk);
let proof = DECIDER::prove(decider_pp, rng, nova.clone()).unwrap();
```

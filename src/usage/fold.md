# Fold

We plug our `FCircuit` into the library:

```rust
// The idea here is that eventually we could replace the next line chunk that defines the
// `type NOVA = Nova<...>` by using another folding scheme that fulfills the `FoldingScheme`
// trait, and the rest of our code would be working without needing to be updated.
type NOVA = Nova<
    Projective,
    GVar,
    Projective2,
    GVar2,
    Sha256FCircuit<Fr>,
    KZG<'static, Bn254>,
    Pedersen<Projective2>,
>;

let num_steps = 10;
let initial_state = vec![Fr::from(1_u32)];

let F_circuit = Sha256FCircuit::<Fr>::new(());

println!("Prepare Nova ProverParams & VerifierParams");
let (prover_params, verifier_params) = nova_setup::<Sha256FCircuit<Fr>>(F_circuit);

println!("Initialize FoldingScheme");
let mut folding_scheme = NOVA::init(&prover_params, F_circuit, initial_state.clone()).unwrap();

// compute a step of the IVC
for i in 0..num_steps {
    let start = Instant::now();
    folding_scheme.prove_step().unwrap();
    println!("Nova::prove_step {}: {:?}", i, start.elapsed());
}

let (running_instance, incoming_instance, cyclefold_instance) = folding_scheme.instances();

println!("Run the Nova's IVC verifier");
NOVA::verify(
    verifier_params,
    initial_state,
    folding_scheme.state(), // latest state
    Fr::from(num_steps as u32),
    running_instance,
    incoming_instance,
    cyclefold_instance,
)
.unwrap();
```

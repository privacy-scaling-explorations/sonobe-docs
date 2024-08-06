# Fold

We plug our `FCircuit` into the library:

```rust
// The idea here is that we could replace the next line chunk that defines the
// `type NOVA = Nova<...>` by using another folding scheme that fulfills the `FoldingScheme`
// trait, and the rest of our code would be working without needing to be updated.
type F = Nova<
    Projective,
    GVar,
    Projective2,
    GVar2,
    Sha256FCircuit<Fr>,
    KZG<'static, Bn254>,
    Pedersen<Projective2>,
    false,
>;

let num_steps = 10;
let initial_state = vec![Fr::from(1_u32)];

let F_circuit = Sha256FCircuit::<Fr>::new(());

let poseidon_config = poseidon_canonical_config::<Fr>();
let mut rng = rand::rngs::OsRng;

println!("Prepare Nova ProverParams & VerifierParams");
let nova_preprocess_params = PreprocessorParam::new(poseidon_config, F_circuit);
let nova_params = FS::preprocess(&mut rng, &nova_preprocess_params).unwrap();


println!("Initialize FoldingScheme");
let mut folding_scheme = FS::init(&nova_params, F_circuit, initial_state.clone()).unwrap();


// compute a step of the IVC
for i in 0..num_steps {
    let start = Instant::now();
    // - 2nd parameter: here we pass an empty vec since the FCircuit that we're
    // using does not use external_inputs
    // - 3rd parameter: is for schemes that support folding more than 2
    // instances at each fold, such as HyperNova. Since we're using Nova, we just
    // set it to 'None'
    folding_scheme.prove_step(rng, vec![], None).unwrap();
    println!("Nova::prove_step {}: {:?}", i, start.elapsed());
}

let (running_instance, incoming_instance, cyclefold_instance) = folding_scheme.instances();

println!("Run the Nova's IVC verifier");
FS::verify(
    nova_params.1,
    initial_state,
    folding_scheme.state(), // latest state
    Fr::from(num_steps as u32),
    running_instance,
    incoming_instance,
    cyclefold_instance,
)
.unwrap();
```

<br>

Now imagine that we want to switch the folding scheme being used. Is as simple as replacing `FS` by:

```rust
type FS = HyperNova<
    Projective,
    GVar,
    Projective2,
    GVar2,
    CubicFCircuit<Fr>,
    KZG<'static, Bn254>,
    Pedersen<Projective2>,
    1, 1, false,
>; 
```
and then adapting the `folding_scheme.prove_step(...)` call accordingly.


<br><br>

As in the previous sections, you can find a full examples with all the code at [sonobe/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/examples).

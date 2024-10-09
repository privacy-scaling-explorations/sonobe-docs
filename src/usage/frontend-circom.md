# Circom frontend
 > **Note**: Circom frontend will be significantly slower than the Arkworks frontend. We explain below how to implement a custom `step_native` function with your circom circuits to speed things up!
 
Experimental frontend using [arkworks/circom-compat](https://github.com/arkworks-rs/circom-compat).

We can define the circuit to be folded in Circom. The only interface that we need to fit in is:

```c
template Example(ivc_state_len, aux_inputs_len) {
    signal input ivc_input[ivc_state_len]; // IVC state
    signal input external_inputs[aux_inputs_len]; // external inputs, not part of the folding state

    signal output ivc_output[ivc_state_len]; // next IVC state
    
    // here it goes the Circom circuit logic
}
component main {public [ivc_input]} = Example();
```

The `ivc_input` is the array that defines the initial state, and the `ivc_output` is the array that defines the output state after the step. Both need to be of the same size. The `external_inputs` array expects auxiliary input values.


In the following image, the `ivc_input`=$z_i$, the `external_inputs`=$w_i$, and the `ivc_output`=$z_{i+1}$, and $F$ is the logic of our Circom circuit:
<p align="center">
    <img src="../imgs/folding-main-idea-diagram.png" style="width:70%;" />
</p>

<br>

So for example, the following circuit proves (at each folding step) knowledge of $x$ such that $y==x^3 + e_0x + e_1$ for a known $y$ ($e_i$ are the `external_inputs[i]`):

```c
pragma circom 2.0.3;

template CubicCircuit() {
    signal input ivc_input[1]; // IVC state
    signal input external_inputs[2]; // not part of the state

    signal output ivc_output[1]; // next IVC state

    signal temp1;
    signal temp2;
    
    temp1 <== ivc_input[0] * ivc_input[0];
    temp2 <== ivc_input[0] * external_inputs[0];
    ivc_output[0] <== temp1 * ivc_input[0] + temp2 + external_inputs[1];
}

component main {public [ivc_input]} = CubicCircuit();
```

<br><br>

Once your `circom` circuit is ready, you can instantiate it with Sonobe. To do this, you will need the `struct CircomFCircuit`.

```rust
// we load our circom compiled R1CS, along with the witness wasm calculator
let r1cs_path = PathBuf::from("./src/frontend/circom/test_folder/cubic_circuit.r1cs");
let wasm_path =
    PathBuf::from("./src/frontend/circom/test_folder/cubic_circuit_js/cubic_circuit.wasm"); 
let f_circuit_params = (r1cs_path, wasm_path, 1, 2); // state_len:1, external_inputs_len:2
let f_circuit = CircomFCircuit::<Fr>::new(f_circuit_params).unwrap();


// [optional] to speed things up, you can define a custom step function to avoid
// defaulting to the snarkjs witness calculator for the native computations,
// which would be slower than rust native operations
circom_fcircuit.set_custom_step_native(Rc::new(|_i, z_i, _external| {
            let z = z_i[0];
            Ok(vec![z * z * z + z + Fr::from(5)])
        }));

pub type N =
    Nova<G1, GVar, G2, GVar2, CircomFCircuit<Fr>, KZG<'static, Bn254>, Pedersen<G2>, false>;
pub type D = DeciderEth<
    G1,
    GVar,
    G2,
    GVar2,
    CircomFCircuit<Fr>,
    KZG<'static, Bn254>,
    Pedersen<G2>,
    Groth16<Bn254>,
    N,
>;

let poseidon_config = poseidon_canonical_config::<Fr>();
let mut rng = rand::rngs::OsRng;

// prepare the Nova prover & verifier params
let nova_preprocess_params = PreprocessorParam::new(poseidon_config, f_circuit.clone());
let nova_params = N::preprocess(&mut rng, &nova_preprocess_params).unwrap();

// initialize the folding scheme engine, in this case we use Nova
let mut nova = N::init(&nova_params, f_circuit.clone(), z_0).unwrap();

// run n steps of the folding iteration
for (i, external_inputs_at_step) in external_inputs.iter().enumerate() {
    let start = Instant::now();
    // the last parameter at 'nova.prove_step()' is for schemes that support
    // folding more than 2 instances at each fold, such as HyperNova. Since
    // we're using Nova, we just set it to 'None'
    nova.prove_step(rng, external_inputs_at_step.clone(), None)
        .unwrap();
    println!("Nova::prove_step {}: {:?}", i, start.elapsed());
}

```
You can find a full example using Nova to fold a Circom circuit at [sonobe/examples/circom_full_flow.rs](https://github.com/privacy-scaling-explorations/sonobe/blob/main/examples/circom_full_flow.rs).

# Circom frontend
 > **Note**: Circom frontend will be significantly slower than the Arkworks frontend. We explain below how to implement a custom `step_native` function with your circom circuits to speed things up!
 
Experimental frontend using [arkworks/circom-compat](https://github.com/arkworks-rs/circom-compat).

We can define the circuit to be folded in Circom. The only interface that we need to fit in is:

```c
template FCircuit(ivc_state_len, aux_inputs_len) {
    signal input ivc_input[ivc_state_len]; // IVC state
    signal input external_inputs[aux_inputs_len]; // not state, 

    signal output ivc_output[ivc_state_len]; // next IVC state
    
    // [...]
}
component main {public [ivc_input]} = Example();
```

The `ivc_input` is the array that defines the initial state, and the `ivc_output` is the array that defines the output state after the step. Both need to be of the same size. The `external_inputs` array expects auxiliary input values.

So for example, the following circuit does the traditional example at each step, which proves knowledge of $x$ such that $y==x^3 + x + e_0 + e_1$ for a known $y$ ($e_i$ are the `external_inputs[i]`):

```c
pragma circom 2.0.3;

template CubicCircuit() {
    signal input ivc_input[1]; // IVC state
    signal input external_inputs[2]; // not state

    signal output ivc_output[1]; // next IVC state

    signal temp1;
    signal temp2;
    
    temp1 <== ivc_input[0] * ivc_input[0];
    temp2 <== ivc_input[0] * external_inputs[0];
    ivc_output[0] <== temp1 * ivc_input[0] + temp2 + external_inputs[1];
}

component main {public [ivc_input]} = CubicCircuit();
```

Once your `circom` circuit is ready, you can instantiate it with Sonobe. To do this, you will need the `struct CircomFCircuit`.

```rust
// we load our circom compiled R1CS, along with the witness wasm calculator
let r1cs_path = PathBuf::from("./src/frontend/circom/test_folder/cubic_circuit.r1cs");
        let wasm_path =
            PathBuf::from("./src/frontend/circom/test_folder/cubic_circuit_js/cubic_circuit.wasm"); 
let mut circom_fcircuit = CircomFCircuit::<Fr>::new((r1cs_path, wasm_path, 1, 2)).unwrap(); // state_len:1, external_inputs_len:2

// to speed things up, you can define a custom step function to avoid defaulting to the snarkjs witness calculator 
circom_fcircuit.set_custom_step_native(Rc::new(|_i, z_i, _external| {
            let z = z_i[0];
            Ok(vec![z * z * z + z + Fr::from(5)])
        }));

// we initialize the required folding schemes parameters, the folding scheme and the final decider that we want to use 
let (fs_prover_params, kzg_vk, g16_pk, g16_vk) =
        init_ivc_and_decider_params::<CircomFCircuit<Fr>>(f_circuit.clone());

pub type NOVA = Nova<G1, GVar, G2, GVar2, CircomFCircuit<Fr>, KZG<'static, Bn254>, Pedersen<G2>>;
pub type DECIDERETH_FCircuit = DeciderEth<
        G1,
        GVar,
        G2,
        GVar2,
        CircomFCircuit<Fr>,
        KZG<'static, Bn254>,
        Pedersen<G2>,
        Groth16<Bn254>,
        NOVA,
>;

// initialize the folding scheme engine, in our case we use Nova
let mut nova = NOVA::init(&fs_prover_params, f_circuit.clone(), z_0).unwrap();

// run n steps of the folding iteration
for (i, external_inputs_at_step) in external_inputs.iter().enumerate() {
    let start = Instant::now();
    nova.prove_step(external_inputs_at_step.clone()).unwrap();
    println!("Nova::prove_step {}: {:?}", i, start.elapsed());
}

```
You can find an example for using Nova over a circom circuit [here](https://github.com/privacy-scaling-explorations/sonobe/blob/main/examples/circom_full_flow.rs).

# Solidity verifier

Having used the `Decider` from `decider_eth.rs`, we can now verify it in Ethereum's EVM.

This can be done either by using the `solidity-verifiers-cli` or directly in rust code.

## Using the CLI to generate the Solidity verifier
Use the [solidity-verifiers-cli](https://github.com/privacy-scaling-explorations/sonobe/tree/main/cli) tool
```
> solidity-verifier-cli -p nova-cyclefold -d ./folding-verifier-solidity/assets/G16_test_vk_data
```

## Generating the Solidity verifier in rust
Assuming that we already have our `Decider` initialized, we can now 


```rust
// prepare the call data
let function_selector =
    get_function_selector_for_nova_cyclefold_verifier(nova.z_0.len() * 2 + 1);
let calldata: Vec<u8> = prepare_calldata(
    function_selector,
    nova.i,
    nova.z_0,
    nova.z_i,
    &nova.U_i,
    &nova.u_i,
    proof,
)
.unwrap();

// prepare the setup params for the solidity verifier
let nova_cyclefold_vk = NovaCycleFoldVerifierKey::from((decider_vp, f_circuit.state_len()));

// generate the solidity code
let decider_solidity_code = get_decider_template_for_cyclefold_decider(nova_cyclefold_vk);

// verify the proof against the solidity code in the EVM
let nova_cyclefold_verifier_bytecode = compile_solidity(&decider_solidity_code, "NovaDecider");
let mut evm = Evm::default();
let verifier_address = evm.create(nova_cyclefold_verifier_bytecode);
let (_, output) = evm.call(verifier_address, calldata.clone());
assert_eq!(*output.last().unwrap(), 1);

// save smart contract and the calldata
println!("storing nova-verifier.sol and the calldata into files");
use std::fs;
fs::write( "./examples/nova-verifier.sol", decider_solidity_code.clone()).unwrap();
fs::write("./examples/solidity-calldata.calldata", calldata.clone()).unwrap();
let s = solidity_verifiers::utils::get_formatted_calldata(calldata.clone());
fs::write("./examples/solidity-calldata.inputs", s.join(",\n")).expect("");
```

You can find the full flow at [examples/full_flow.rs](https://github.com/privacy-scaling-explorations/sonobe/blob/main/examples/full_flow.rs).

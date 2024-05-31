# Circom frontend
 > **Note**: Circom frontend will be significantly slower than the Arkworks frontend.
 
Experimental frontend using [arkworks/circom-compat](https://github.com/arkworks-rs/circom-compat).

We can define the circuit to be folded in Circom. The only interface that we need to fit in is:

```c
template FCircuit() {
    signal input ivc_input[1]; // IVC state
    signal input external_inputs[2]; // not state

    signal output ivc_output[1]; // next IVC state
    
    // [...]
}
component main {public [ivc_input]} = Example();
```

The `ivc_input` is the array that defines the initial state, and the `ivc_output` is the array that defines the output state after the step.

So for example, the following circuit does the traditional example at each step, which proves knowledge of $x$ such that $y==x^3 + x + e_0 + e_1$ for a known $y$ ($e_i$ are the `external_inputs[i]`):

```c
pragma circom 2.0.3;

template Example () {
    signal input ivc_input[1]; // IVC state
    signal input external_inputs[2]; // not state

    signal output ivc_output[1]; // next IVC state

    signal temp1;
    signal temp2;
    
    temp1 <== ivc_input[0] * ivc_input[0];
    temp2 <== ivc_input[0] * external_inputs[0];
    ivc_output[0] <== temp1 * ivc_input[0] + temp2 + external_inputs[1];
}

component main {public [ivc_input]} = Example();
```

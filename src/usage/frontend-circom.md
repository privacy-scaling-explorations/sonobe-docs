# Circom frontend
 > Experimental frontend using [arkworks/circom-compat](https://github.com/arkworks-rs/circom-compat).
 
We can define the circuit to be folded in Circom. The only interface that we need to fit in is:

```c
template FCircuit() {
    signal input ivc_input[1];
    signal output ivc_output[1];   
    // [...]
}
component main {public [ivc_input]} = Example();
```

The `ivc_input` is the array that defines the initial state, and the `ivc_output` is the array that defines the output state after the step.

So for example, the following circuit does the traditional example at each step, which proves knowledge of $x$ such that $y==x^3 + x + 5$ for a known $y$:

```c
pragma circom 2.0.3;

template Example () {
    signal input ivc_input[1];
    signal output ivc_output[1];   
    signal temp;
    
    temp <== ivc_input[0] * ivc_input[0];
    ivc_output[0] <== temp * ivc_input[0] + ivc_input[0] + 5;
}

component main {public [ivc_input]} = Example();
```

# Noir frontend

Experimental frontend using [arkworks_backend](https://github.com/dmpierre/arkworks_backend). 

If you write your R1CS circuits using [Noir](https://noir-lang.org/), you can also use [sonobe](https://github.com/privacy-scaling-explorations/sonobe/) to fold your circuits. Under the hood, we bridge compiled Noir circuits to arkworks R1CS. Beware that sonobe assumes that the compiled Noir circuit that is being folded does not use any other opcode than an arithmetic gate: you can not fold circuits calling [oracles](https://noir-lang.org/docs/noir/concepts/oracles) or using [unconstrained](https://noir-lang.org/docs/noir/concepts/unconstrained/) functions. You should be able to use Noir's standard library though.

Using Noir with sonobe is similar to using any other frontend. Sonobe expects that the length of your public and private (external) inputs match what the Noir circuit expects. Note that sonobe does not expect your public inputs to follow some specific naming convention when using Noir: it will assume that whatever public input variable you have is the IVC state. 

This [example](https://github.com/privacy-scaling-explorations/sonobe/blob/main/examples/noir_full_flow.rs) shows how to fold a poseidon circuit from the Noir standard library. 


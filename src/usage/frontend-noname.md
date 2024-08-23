# Noname frontend

Experimental frontend using [ark-noname](https://github.com/dmpierre/ark-noname/tree/feat/sonobe-integration). 

If you write your R1CS circuits using [Noname](), you can also use [sonobe](https://github.com/privacy-scaling-explorations/sonobe/) to fold your circuits. Under the hood, we bridge compiled Noname circuits to arkworks R1CS. Our Noname integration does not support Noname's standard library for now. 

Using Noname with sonobe is similar to using any other frontend. Sonobe expects that the length of your public and private (external) inputs match what the Noname circuit expects. Note that sonobe does not expect your public inputs to follow some specific naming convention when using Noname: it will assume that whatever public input variable you have is the IVC state. 

This [example](https://github.com/privacy-scaling-explorations/sonobe/blob/main/examples/noname_full_flow.rs) shows how to fold a simple Noname circuit having both public and external, private inputs.


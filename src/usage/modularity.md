# Modularity
## Swapping curves and proving schemes
Thanks to the modularity of arkworks, we can swap between curves and proving systems.
Suppose that for the final proof (decider), instead of using Groth16 over the BN254 curve, we want to use Marlin+IPA over the Pasta curves, so we can enjoy of not needing a trusted setup.
It just requires few line changes on our previous code [...]

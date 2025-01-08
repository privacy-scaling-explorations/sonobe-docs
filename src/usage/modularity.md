# Modularity
## Swapping curves and proving schemes
Thanks to the modularity of arkworks, we can swap between curves and proving systems.
Suppose that for the final proof (decider), instead of using Groth16 over the BN254 curve, we want to use Marlin+IPA over the Pasta curves, so we can enjoy of not needing a trusted setup.

It just requires few line changes on our previous code, for example, imagine we're using the Nova folding scheme (where `FC` is the `FCircuit` that we want to fold):
```rust
type FS = Nova<G1, G2, FC, Pedersen<G1>, Pedersen<G2>, false>;
```

to switch the folding scheme to use HyperNova, is as simple as updating the previous line to:

```rust
type FS = HyperNova< G1, G2, FC, Pedersen<G1>, Pedersen<G2>, 1, 1, false>;
```

similarly for using ProtoGalaxy folding scheme:
```rust
type FS = ProtoGalaxy<G1, G2, FC, Pedersen<G1>, Pedersen<G2>>;
```

We can use any cycle of curves available in arkworks for the `G1` and `G2`, and the rest of the code can be kept the same.

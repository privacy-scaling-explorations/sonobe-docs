# Frontend

The frontend interface allows to define the circuit to be folded. The currently available frontends are [`circom`](https://github.com/iden3/circom) and [arkworks](https://github.com/arkworks-rs/r1cs-std). We will show here how to define a circuit using `arkworks`.

# The `FCircuit` trait

To be folded with sonobe, a circuit needs to implement the [`FCircuit` trait](https://github.com/privacy-scaling-explorations/sonobe/blob/main/sonobe/src/frontend/mod.rs). This trait defines the methods that sonobe expects from the circuit to be folded. It corresponds to the $F$ function that is being folded. The trait has the following methods:

```rust
/// FCircuit defines the trait of the circuit of the F function, which is the one being folded (ie.
/// inside the agmented F' function).
/// The parameter z_i denotes the current state, and z_{i+1} denotes the next state after applying
/// the step.
pub trait FCircuit<F: PrimeField>: Clone + Debug {
    type Params: Debug;

    /// Returns a new FCircuit instance
    fn new(params: Self::Params) -> Self;

    /// Returns the number of elements in the state of the FCircuit, which corresponds to the
    /// FCircuit inputs.
    fn state_len(&self) -> usize;

    /// Computes the next state values in place, assigning z_{i+1} into z_i, and computing the new
    /// z_{i+1}
    fn step_native(
        // this method uses self, so that each FCircuit implementation (and different frontends)
        // can hold a state if needed to store data to compute the next state.
        &self,
        i: usize,
        z_i: Vec<F>,
    ) -> Result<Vec<F>, Error>;

    /// Generates the constraints for the step of F for the given z_i
    fn generate_step_constraints(
        // this method uses self, so that each FCircuit implementation (and different frontends)
        // can hold a state if needed to store data to generate the constraints.
        &self,
        cs: ConstraintSystemRef<F>,
        i: usize,
        z_i: Vec<FpVar<F>>,
    ) -> Result<Vec<FpVar<F>>, SynthesisError>;
}
```


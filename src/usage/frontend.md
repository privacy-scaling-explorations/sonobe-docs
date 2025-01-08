# Frontend

The frontend interface allows to define the circuit to be folded.
The recommended frontend is to directly use [arkworks](https://github.com/arkworks-rs) to define the FCircuit, just following the [`FCircuit` trait](https://github.com/privacy-scaling-explorations/sonobe/blob/main/folding-schemes/src/frontend/mod.rs).

Alternatively, experimental frontends for [Circom](https://github.com/iden3/circom), [Noir](https://noir-lang.org/), and [Noname](https://github.com/zksecurity/noname) can be found at [sonobe/experimental-frontends](https://github.com/privacy-scaling-explorations/sonobe/tree/main/experimental-frontends), which have some computational (and time) overhead.


Defining a circuit to be folded is as simple as fulfilling the `FCircuit` trait interface. Henceforth, integrating a new zk circuits language into Sonobe, can be done by building a wrapper on top of it that satisfies the `FCircuit` trait.

# The `FCircuit` trait

To be folded with sonobe, a circuit needs to implement the [`FCircuit` trait](https://github.com/privacy-scaling-explorations/sonobe/blob/main/folding-schemes/src/frontend/mod.rs). This trait defines the methods that sonobe expects from the circuit to be folded. It corresponds to the $F$ function that is being folded. The trait has the following methods:

```rust
/// FCircuit defines the trait of the circuit of the F function, which is the one being folded (ie.
/// inside the agmented F' function).
/// The parameter z_i denotes the current state, and z_{i+1} denotes the next state after applying
/// the step.
/// Note that the external inputs for the specific circuit are defined at the implementation of
/// both `FCircuit::ExternalInputs` and `FCircuit::ExternalInputsVar`, where the `Default` trait
/// implementation. For example if the external inputs are just an array of field elements, their
/// `Default` trait implementation must return an array of the expected size.
pub trait FCircuit<F: PrimeField>: Clone + Debug {
    type Params: Debug;
    type ExternalInputs: Clone + Default + Debug;
    type ExternalInputsVar: Clone + Default + Debug + AllocVar<Self::ExternalInputs, F>;

    /// returns a new FCircuit instance
    fn new(params: Self::Params) -> Result<Self, Error>;

    /// returns the number of elements in the state of the FCircuit, which corresponds to the
    /// FCircuit inputs.
    fn state_len(&self) -> usize;

    /// generates the constraints for the step of F for the given z_i
    fn generate_step_constraints(
        // this method uses self, so that each FCircuit implementation (and different frontends)
        // can hold a state if needed to store data to generate the constraints.
        &self,
        cs: ConstraintSystemRef<F>,
        i: usize,
        z_i: Vec<FpVar<F>>,
        external_inputs: Self::ExternalInputsVar, // inputs that are not part of the state
    ) -> Result<Vec<FpVar<F>>, SynthesisError>;
}
```

## Side note: adhoc frontend dependencies for the experimental frontends
> Note: this affects only to the experimental frontends in the [sonobe/experimental-frontends](https://github.com/privacy-scaling-explorations/sonobe/tree/main/experimental-frontends) directory.

There are many ad hoc dependencies for each of the frontends integrated with Sonobe. Here are a few reasons why this is the case.

First, any frontend different from Arkworks needs a "bridge"—a way to "translate" constraints from one R1CS framework to another. Indeed, Sonobe uses Arkworks as its constraint writing framework under the hood. Therefore, any R1CS written with another framework (e.g., Circom) will need to port its constraint system to Arkworks.

Second, R1CS bridge libraries are often developed by teams assuming that public and private witness values will be assigned simultaneously. This is typically the standard approach. However, folding schemes differ, as they redirect the public outputs of one computation to the public inputs of the next step. This means that public inputs are already assigned when calling the "bridge" to port whatever frontend constraints to Sonobe.

Consequently, we had to make a few ad hoc changes to each of these bridges to make them compatible with Sonobe. We are aware that this significantly increases dependencies, which also raises maintenance requirements. The topic of how to reduce our reliance on such ad hoc changes in the future is being actively [discussed](https://github.com/privacy-scaling-explorations/sonobe/issues/146).  

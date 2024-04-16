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

# Example

Let's walk through different simple examples implementing the `FCircuit` trait. By the end of this section, you will hopefully be familiar with how to integrate an `arkworks` circuit into sonobe.

## Folding a simple circuit

The circuit we will fold has a state of 5 public elements. At each step, we will want the circuit to compute the next state by:

1. adding 4 to the first element
2. adding 40 to the second element
3. multiplying the third element by 4
4. multiplying the fourth element by 40
5. adding 100 to the fifth element

Let's implement this now:

```rust
// Define a struct that will be our circuit. This struct will implement the FCircuit trait.
#[derive(Clone, Copy, Debug)]
pub struct MultiInputsFCircuit<F: PrimeField> {
    _f: PhantomData<F>,
}

// Implement the FCircuit trait for the struct
impl<F: PrimeField> FCircuit<F> for MultiInputsFCircuit<F> {
    type Params = ();

    fn new(_params: Self::Params) -> Self {
        Self { _f: PhantomData }
    }

    fn state_len(&self) -> usize {
        5 // This circuit has 5 inputs
    }

    // Computes the next state values in place, assigning z_{i+1} into z_i, and computing the new z_{i+1}
    // We want the `step_native` method to implement the same logic as the `generate_step_constraints` method
    fn step_native(&self, _i: usize, z_i: Vec<F>) -> Result<Vec<F>, Error> {
        let a = z_i[0] + F::from(4_u32);
        let b = z_i[1] + F::from(40_u32);
        let c = z_i[2] * F::from(4_u32);
        let d = z_i[3] * F::from(40_u32);
        let e = z_i[4] + F::from(100_u32);

        Ok(vec![a, b, c, d, e]) // The length of the returned vector should match `state_len`
    }

    /// Generates R1CS constraints for the step of F for the given z_i
    fn generate_step_constraints(
        &self,
        cs: ConstraintSystemRef<F>,
        _i: usize,
        z_i: Vec<FpVar<F>>,
    ) -> Result<Vec<FpVar<F>>, SynthesisError> {
        // Implementing the circuit constraints
        let four = FpVar::<F>::new_constant(cs.clone(), F::from(4u32))?;
        let forty = FpVar::<F>::new_constant(cs.clone(), F::from(40u32))?;
        let onehundred = FpVar::<F>::new_constant(cs.clone(), F::from(100u32))?;
        let a = z_i[0].clone() + four.clone();
        let b = z_i[1].clone() + forty.clone();
        let c = z_i[2].clone() * four;
        let d = z_i[3].clone() * forty;
        let e = z_i[4].clone() + onehundred;

        Ok(vec![a, b, c, d, e]) // The length of the returned vector should match `state_len`
    }
}
```

## Folding a `Sha256` circuit

We will fold a simple `Sha256` circuit. The circuit has a state of 1 public element. At each step, we will want the circuit to compute the next state by applying the `Sha256` function to the current state. 

Note that the logic here is also very similar to the previous example: write a struct that will hold the circuit, implement the `FCircuit` trait for the struct, ensure that the length of the state is correct, and implement the `step_native` and `generate_step_constraints` methods.

```rust
// Define a struct that will be our circuit. This struct will implement the FCircuit trait.
#[derive(Clone, Copy, Debug)]
pub struct Sha256FCircuit<F: PrimeField> {
    _f: PhantomData<F>,
}

impl<F: PrimeField> FCircuit<F> for Sha256FCircuit<F> {
    type Params = ();

    fn new(_params: Self::Params) -> Self {
        Self { _f: PhantomData }
    }
    fn state_len(&self) -> usize {
        1
    }

    /// Computes the next state values in place, assigning z_{i+1} into z_i, and computing the new
    /// z_{i+1}
    fn step_native(&self, _i: usize, z_i: Vec<F>) -> Result<Vec<F>, Error> {
        let out_bytes = Sha256::evaluate(&(), z_i[0].into_bigint().to_bytes_le()).unwrap();
        let out: Vec<F> = out_bytes.to_field_elements().unwrap();

        Ok(vec![out[0]])
    }

    /// Generates the constraints for the step of F for the given z_i
    fn generate_step_constraints(
        &self,
        _cs: ConstraintSystemRef<F>,
        _i: usize,
        z_i: Vec<FpVar<F>>,
    ) -> Result<Vec<FpVar<F>>, SynthesisError> {
        let unit_var = UnitVar::default();
        let out_bytes = Sha256Gadget::evaluate(&unit_var, &z_i[0].to_bytes()?)?;
        let out = out_bytes.0.to_constraint_field()?;
        Ok(vec![out[0].clone()])
    }
}
```

## Folding a circuit with public and private inputs

Sometimes, the circuit to be folded will have private inputs. Let's see how we can setup such a circuit to be folded with sonobe. Again, the logic here is also very similar to our previous examples. The main difference is that the `struct` which will hold the circuit also holds a `Vec` of private inputs.

```rust
#[derive(Clone, Debug)]
pub struct ACircuitWithPrivateState<F: PrimeField> {
    _f: PhantomData<F>,
    private_state: Vec<F>, // private inputs, here a `Vec` of field elements, but you can specify whatever type you prefer here
}

impl<F: PrimeField> FCircuit<F> for ACircuitWithPrivateState<F> {
    type Params = Vec<F>;

    fn new(params: Self::Params) -> Self {
        Self {
            _f: PhantomData,
            private_state: params,
        }
    }

    fn state_len(&self) -> usize {
        3 // the length of the state should match the size of the public inputs, not including the private inputs
    }

    fn step_native(&self, i: usize, z_i: Vec<F>) -> Result<Vec<F>, folding_schemes::Error> {
        let a = z_i[0] + F::from(4_u32);
        let b = z_i[1] + F::from(40_u32);
        let c = z_i[2] * F::from(4_u32);
        let d = self.private_state[0] * a;
        let e = self.private_state[1] * c;

        Ok(vec![a, b, c])
        Ok(new_z_i)
    }

    fn generate_step_constraints(
        &self,
        cs: ConstraintSystemRef<F>,
        i: usize,
        z_i: Vec<ark_r1cs_std::fields::fp::FpVar<F>>,
    ) -> Result<Vec<ark_r1cs_std::fields::fp::FpVar<F>>, SynthesisError> {
        let four = FpVar::<F>::new_constant(cs.clone(), F::from(4u32))?;
        let forty = FpVar::<F>::new_constant(cs.clone(), F::from(40u32))?;
        let onehundred = FpVar::<F>::new_constant(cs.clone(), F::from(100u32))?;

        // we still need to allocate our private state as witnesses
        let priv_var_0 = FpVar::<F>::new_witness(cs.clone(), || Ok(self.private_state[0].clone()))?;
        let priv_var_1 = FpVar::<F>::new_witness(cs.clone(), || Ok(self.private_state[1].clone()))?;

        let a = z_i[0].clone() + four.clone();
        let b = z_i[1].clone() + forty.clone();
        let c = z_i[2].clone() * four;
        let d = priv_var_0 * a;
        let e = priv_var_1 * c;

        Ok(vec![a, b, c])
    }
}

```

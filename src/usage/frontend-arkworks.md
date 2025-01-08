# Arkworks frontend

Let's walk through different simple examples implementing the `FCircuit` trait. By the end of this section, you will hopefully be familiar with how to integrate an `arkworks` circuit into Sonobe.

You can find most of the following examples with the rest of code to run them at the [`examples`](https://github.com/privacy-scaling-explorations/sonobe/tree/main/examples) directory of the Sonobe repo.

## Cubic circuit example
This first example implements the `FCircuit` trait for the R1CS example circuit from [Vitalik's post](https://www.vitalik.ca/general/2016/12/10/qap.html), which checks $x^3 + x + 5 == y$.

$z_i$ is used as $x$, and $z_{i+1}$ is used as $y$, and at the next step, $z_{i+1}$ will be assigned to $z_i$, and a new $z_{i+1}$ will be computted.

```rust
#[derive(Clone, Copy, Debug)]
pub struct CubicFCircuit<F: PrimeField> {
    _f: PhantomData<F>,
}
impl<F: PrimeField> FCircuit<F> for CubicFCircuit<F> {
    type Params = ();
    type ExternalInputs = ();
    type ExternalInputsVar = ();

    fn new(_params: Self::Params) -> Result<Self, Error> {
        Ok(Self { _f: PhantomData })
    }
    fn state_len(&self) -> usize {
        1
    }
    fn generate_step_constraints(
        &self,
        cs: ConstraintSystemRef<F>,
        _i: usize,
        z_i: Vec<FpVar<F>>,
        _external_inputs: Self::ExternalInputsVar,
    ) -> Result<Vec<FpVar<F>>, SynthesisError> {
        let five = FpVar::<F>::new_constant(cs.clone(), F::from(5u32))?;
        let z_i = z_i[0].clone();

        Ok(vec![&z_i * &z_i * &z_i + &z_i + &five])
    }
}
```


## Multiple inputs circuit example

The following example has a state of 5 public elements. At each step, we will want the circuit to compute the next state by:

1. adding 4 to the first element
2. adding 40 to the second element
3. multiplying the third element by 4
4. multiplying the fourth element by 40
5. adding 100 to the fifth element


```rust
// Define a struct that will be our circuit. This struct will implement the FCircuit trait.
#[derive(Clone, Copy, Debug)]
pub struct MultiInputsFCircuit<F: PrimeField> {
    _f: PhantomData<F>,
}
impl<F: PrimeField> FCircuit<F> for MultiInputsFCircuit<F> {
    type Params = ();
    type ExternalInputs = ();
    type ExternalInputsVar = ();

    fn new(_params: Self::Params) -> Result<Self, Error> {
        Ok(Self { _f: PhantomData })
    }
    fn state_len(&self) -> usize {
        5
    }
    /// generates the constraints for the step of F for the given z_i
    fn generate_step_constraints(
        &self,
        cs: ConstraintSystemRef<F>,
        _i: usize,
        z_i: Vec<FpVar<F>>,
        _external_inputs: Self::ExternalInputsVar,
    ) -> Result<Vec<FpVar<F>>, SynthesisError> {
        let four = FpVar::<F>::new_constant(cs.clone(), F::from(4u32))?;
        let forty = FpVar::<F>::new_constant(cs.clone(), F::from(40u32))?;
        let onehundred = FpVar::<F>::new_constant(cs.clone(), F::from(100u32))?;
        let a = z_i[0].clone() + four.clone();
        let b = z_i[1].clone() + forty.clone();
        let c = z_i[2].clone() * four;
        let d = z_i[3].clone() * forty;
        let e = z_i[4].clone() + onehundred;

        Ok(vec![a, b, c, d, e])
    }
}
```


## Using external inputs

In this example we set the state to be the previous state together with an external input, and the new state is an array which contains the new state and a zero which will be ignored.

This is useful for example if we want to fold multiple verifications of signatures, where the circuit F checks the signature and is folded for each of the signatures and public keys. To keep things simpler, the following example does not verify signatures but does a similar approach with a chain of hashes, where each iteration hashes the previous step output ($z_i$) together with an external input ($w_i$).

```
       w_1     w_2     w_3     w_4     
       │       │       │       │      
       ▼       ▼       ▼       ▼      
      ┌─┐     ┌─┐     ┌─┐     ┌─┐     
─────►│F├────►│F├────►│F├────►│F├────►
 z_1  └─┘ z_2 └─┘ z_3 └─┘ z_4 └─┘ z_5


where each F is:
  w_i                                        
   │     ┌────────────────────┐              
   │     │FCircuit            │              
   │     │                    │              
   └────►│ h =Hash(z_i[0],w_i)│              
────────►│ │                  ├───────►      
 z_i     │ └──►z_{i+1}=[h]    │  z_{i+1}
         │                    │              
         └────────────────────┘
```

where each $w_i$ value is set at the `external_inputs` array.

The last state $z_i$ is used together with the external input w_i as inputs to compute the new state $z_{i+1}$.

```rust
use ark_crypto_primitives::{
    crh::{
        poseidon::constraints::{CRHGadget, CRHParametersVar},
        poseidon::CRH,
        CRHScheme, CRHSchemeGadget,
    },
    sponge::{poseidon::PoseidonConfig, Absorb},
};

#[derive(Clone, Debug)]
pub struct ExternalInputsCircuits<F: PrimeField>
where
    F: Absorb,
{
    _f: PhantomData<F>,
    poseidon_config: PoseidonConfig<F>,
}
impl<F: PrimeField> FCircuit<F> for ExternalInputsCircuit<F>
where
    F: Absorb,
{
    type Params = PoseidonConfig<F>;
    type ExternalInputs = [F; 1];
    type ExternalInputsVar = [FpVar<F>; 1];

    fn new(params: Self::Params) -> Result<Self, Error> {
        Ok(Self {
            _f: PhantomData,
            poseidon_config: params,
        })
    }
    fn state_len(&self) -> usize {
        1
    }
    /// generates the constraints and returns the next state value for the step of F for the given
    /// z_i and external_inputs
    fn generate_step_constraints(
        &self,
        cs: ConstraintSystemRef<F>,
        _i: usize,
        z_i: Vec<FpVar<F>>,
        external_inputs: Self::ExternalInputsVar,
    ) -> Result<Vec<FpVar<F>>, SynthesisError> {
        let crh_params =
            CRHParametersVar::<F>::new_constant(cs.clone(), self.poseidon_config.clone())?;
        let hash_input: [FpVar<F>; 2] = [z_i[0].clone(), external_inputs[0].clone()];
        let h = CRHGadget::<F>::evaluate(&crh_params, &hash_input)?;
        Ok(vec![h])
    }
}
```

<br><br>

# Complete examples
You can find the complete examples with all the imports at [sonobe/examples](https://github.com/privacy-scaling-explorations/sonobe/tree/main/examples).

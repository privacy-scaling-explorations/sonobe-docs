# decider for offchain verification

**Overview**: This section describes the *Decider* (compressed SNARK / final proof) for the non-ethereum use cases in which the verification of the Nova+CycleFold proofs is done offchain.
For onchain Ethereum use cases, check out the [decider-onchain section](decider_onchain.md).

## Setup
At the final stage of the Nova+CycleFold folding, after $d$ iterations, we have the committed instances $\{u_d, U_d, U_{EC,d} \}$ and their respective witnessess.

![cyclefold diagram](../imgs/cyclefold-paper-diagram.jpg)
<span style="padding:20px;">*Diagram source: CycleFold paper ([https://eprint.iacr.org/2023/1192.pdf](https://eprint.iacr.org/2023/1192.pdf)). In the case of this document $d=i+2$, so $u_{i+2} = u_d$, $U_{i+2}=U_d$, $U_{EC,i+2}=U_{EC,d}$.*</span>

We work with a cycle of curves $E_1$ and $E_2$, where $E_1.F_r = E_2.F_q$ and $E_1.F_q=E_2.F_r$.
We will use $F_r$ for referring to $E_1.F_r=E_2.F_q$, and $F_q$ for referring to $E_1.F_q=E_2.F_r$.
The main circuit constraint field is $F_r$, and $C_{EC}$ circuit constraint field is $F_q$.

The $u_d$ and $U_d$ contain: $\{ \overline{E} \in E_1, \overline{W} \in E_1, u \in F_r, x \in F_r \}$

And $U_{EC,d}$ contains: $\{ \overline{E} \in E_2, \overline{W} \in E_2, u \in F_q, x \in F_q^n \}$

## Decider high level checks
- $U_{n + 1}$ is the correct folding of $U_n, u_n$, i.e., $NIFS.V(r, U_n, u_n, \overline{T}) = U_{n + 1}$.
- $R_{r1cs}(W_{n + 1}, U_{n + 1}) = 1$:
  - The running instance $U_{n + 1}$ and witness $W_{n + 1}$ satisfy (relaxed) $r1cs$.
  - The commitments in $U_{n + 1}$ (i.e., $U_{n+1}.\{ \overline{E}, \overline{W} \} \in E_1$) open to the values in $W_{n + 1}$ (i.e., $W_{n+1}.\{ E, W \}$).
- $R_{r1cs_{EC}}(W_{EC, n}, U_{EC, n})$:
  - The CycleFold instance $U_{EC, n}$ and witness $W_{EC, n}$ satisfy (relaxed) $r1cs_{EC}$.
  - The commitments in $U_{EC, n}$ (i.e., $U_{EC,n}.\{ \overline{E}, \overline{W} \} \in E_2$) open to the values in $W_{EC, n}$ (i.e., $W_{EC, n}.\{ E, W \}$).
- $u_n$ is valid:
  - $u_n.x$ contains the correct hash of the initial and final states, i.e., $u_n.x_0 = H(n, z_0, z_n, U_n)$ and $u_n.x_1 = H(U_{EC,n})$
  - $u_n$ is indeed an incoming instance, i.e., $u_n.\overline{E}=0$ and $u_n.u=1$.

> **Note**: These are the same checks for both the Onchain & Offchain Deciders. The difference lays on how are performed.

## Offchain Decider approach
In the offchain case, since we can end up with proofs in both curves of the cycle, we try to fit all the computations natively in each curve respectively.

> We use the same checks numbers as the ones used in the [Onchain Decider](nova-decider-onchain.md) in order to make the relation of the checks easier to follow.

#### Circuit1 $\in Fr$ ($E_1.F_r$)

- 1: Enforce $U_{n+1}$ and $W_{n+1}$ satisfy $r1cs$, the RelaxedR1CS relation of the AugmentedFCircuit
- 2: Check that $u_n.\overline{E}=0$ and $u_n.u=1$
- 3: Check that $u_n.x_0 = H(n, z_0, z_n, U_n)$ and $u_n.x_1 = H(U_{EC,n})$
- 6.1: Partially enforce that $U_{n + 1}$ is the correct folding of $U_n, u_n$
    - By partially, we mean that only field elements in $U_{n + 1}$ are checked, while the group elements (i.e., commitments) are not, as they are in $E_1$ and are expensive non-native operations.
- 7.1: Check correct computation of the KZG challenges
    $$c_E = H(\overline{E}.\{x,y\}),~~c_W = H(\overline{W}.\{x,y\})$$
    which we do through in-circuit Transcript.
- 7.2: Check that the KZG evaluations are correct
    - $eval_W == p_W(c_W)$
    - $eval_E == p_E(c_E)$
    <br>where $p_W, p_E \in \mathbb{F}[X]$ are the interpolated polynomials from $W_{i+1}.W,~ W_{i+1}.E$ respectively.
    <br> ie. $p_W(x) = interpolate(W_{i+1}.W, 0)$, where $0$ is zero-padding to the next power of 2 length, and $interpolate()$ interpolates a (unique) polynomial from the vector

#### Circuit2 $\in Fq$ ($E_2.F_r$)

Note that in onchain decider, step 4 (Pedersen commitments verification of $U_{EC,n}.\{ \overline{E}, \overline{W} \}$) is checked in-circuit, necessitating `~5M` constraints.

In the case of offchain decider, we aim to reduce the number of constraints for commitment verification for CycleFold instances.
To this end, we apply the same approach as in step 7 of the onchain decider for checking commitments in primary instances.
That is, we now use KZG commitments as well for the CycleFold instances, enabling us to separate expensive parts in commitments verification from the circuit.

Below are checks for the Circuit2:

- 4.1: Check correct computation of the KZG challenges for $U_{EC}$
$$c_E = H(U_{EC}.\overline{E}.\{x,y\}),~~c_W = H(U_{EC}.\overline{W}.\{x,y\})$$
which we do through in-circuit Transcript.
- 4.2: check that the KZG evaluations for $U_{EC}$ are correct
    - $eval_W == p_W(c_W)$
    - $eval_E == p_E(c_E)$
    <br>where $p_W, p_E \in \mathbb{F}[X]$ are the interpolated polynomials from $W_{i+1}.W,~ W_{i+1}.E$ respectively.
- 5: Enforce $U_{EC,n}$ and $W_{EC,n}$ satisfy $r1cs_{EC}$, the RelaxedR1CS relation of the CycleFoldCircuit
    - This check becomes native because the constraint system is now over $F_q$.

#### Outside the circuits

- 6.2. Check the commitments in $U_{n + 1}$ are the correct folding of those in $U_n, u_n$
- 7.3: Verify the KZG proofs with respect to the commitments $U_{n + 1}.\{ \overline{E}, \overline{W} \} \in E_1$ and the challenges $c_E, c_W$.
- 4.3: Verify the KZG proofs with respect to the commitments $U_{EC,n}.\{ \overline{E}, \overline{W} \} \in E_2$ and the challenges $c_E, c_W$.


## Proving scheme
We could use a SNARK adapted to RelaxedR1CS, but before that is ready we use a regular R1CS SNARK and check the RelaxedR1CS relations in-circuit as described above.
Two proofs are generated, one for each circuit over their respective curves of the cycle.

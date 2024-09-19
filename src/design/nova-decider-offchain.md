# decider for offchain verification

**Overview**: This section describes the *Decider* (compressed SNARK / final proof) for the non-ethereum use cases in which the verification of the Nova+CycleFold proofs is done offchain.
For onchain Ethereum use cases, check out the [decider-onchain section](decider_onchain.md).

## Setup
At the final stage of the Nova+CycleFold folding, after $d$ iterations, we have the committed instances $\{u_d, U_d, U_{EC,d} \}$ and their respective witnessess.

![cyclefold diagram](/imgs/cyclefold-paper-diagram.jpg)
<span style="padding:20px;">*Diagram source: CycleFold paper ([https://eprint.iacr.org/2023/1192.pdf](https://eprint.iacr.org/2023/1192.pdf)). In the case of this document $d=i+2$, so $u_{i+2} = u_d$, $U_{i+2}=U_d$, $U_{EC,i+2}=U_{EC,d}$.*</span>

We work with a cycle of curves $E_1$ and $E_2$, where $E_1.F_r = E_2.F_q$ and $E_1.F_q=E_2.F_r$.
We will use $F_r$ for referring to $E_1.F_r=E_2.F_q$, and $F_q$ for referring to $E_1.F_q=E_2.F_r$.
The main circuit constraint field is $F_r$, and $C_{EC}$ circuit constraint field is $F_q$.

The $u_d$ and $U_d$ contain: $\{ \overline{E} \in E_1, \overline{W} \in E_1, u \in F_r, x \in F_r \}$

And $U_{EC,d}$ contains: $\{ \overline{E} \in E_2, \overline{W} \in E_2, u \in F_q, x \in F_q^n \}$

## Decider high level checks
*These are the same checks for both the Onchain & Offchain Deciders. The difference lays on how are performed.*

1. check $NIFS.V(r, U_n, u_n, \overline{T}) \stackrel{?}{=} U_{n+1}$
2. check that $u_n.\overline{E}=0$ and $u_n.u=1$
3. check that $u_n.x_0 = H(n, z_0, z_n, U_n)$ and $u_n.x_1 = H(U_{EC,n})$
4. correct RelaxedR1CS relation of $U_{n+1}, W_{n+1}$ of the AugmentedFCircuit
5. check commitments of $U_{n+1}.\{ \overline{E}, \overline{W} \}$ with respect $W_{n+1}$ (where $\overline{E}, \overline{W} \in E_1$)
6. check the correct RelaxedR1CS relation of $U_{EC,n}, W_{EC,n}$ of the CycleFoldCircuit
7. check commitments of $U_{EC,n}.\{ \overline{E}, \overline{W} \}$ with respect $W_{EC,n}$ (where $\overline{E},\overline{W} \in E_2$)

## Offchain Decider approach
In the offchain case, since we can end up with proofs in both curves of the cycle, we try to fit all the computations natively in each curve respectively.

> We use the same checks numbers as the ones used in the [Onchain Decider](nova-decider-onchain.md) in order to make the relation of the checks easier to follow.

#### Circuit1 $\in Fr$ ($E_1.F_r$)

- 1.1: check that the given NIFS challenge $r$ is indeed well computed. This challenge is then used outside the circuit by the Verifier to compute NIFS.V obtaining $U_{i+1}$
- 2: check that $u_n.\overline{E}=0$ and $u_n.u=1$
- 3: check that $u_n.x_0 = H(n, z_0, z_n, U_n)$ and $u_n.x_1 = H(U_{EC,n})$
- 4: correct RelaxedR1CS relation of $U_{n+1}, W_{n+1}$ of the AugmentedFCircuit
- 5.1: Check correct computation of the KZG challenges for $U_{n+1}$
    $$c_E = H(U_{n+1}.\overline{E}.\{x,y\}),~~c_W = H(U_{n+1}.\overline{W}.\{x,y\})$$
    which we do through in-circuit Transcript.
- 5.2: check that the KZG evaluations for $U_{n+1}$ are correct
    - $eval_W == p_W(c_W)$
    - $eval_E == p_E(c_E)$
    <br>where $p_W, p_E \in \mathbb{F}[X]$ are the interpolated polynomials from $W_{i+1}.W,~ W_{i+1}.E$ respectively,
    <br> ie. $p_W(x) = interpolate(W_{i+1}.W, 0)$, where $0$ is zero-padding to the next power of 2 length, and $interpolate()$ interpolates a (unique) polynomial from the vector

#### Circuit2 $\in Fq$ ($E_2.F_r$)

- 6: correct RelaxedR1CS relation of $U_{EC,d}, W_{EC,d}$
- 7.1: Check correct computation of the KZG challenges for $U_{EC}$
    $$c_E = H(U_{EC}.\overline{E}.\{x,y\}),~~c_W = H(U_{EC}.\overline{W}.\{x,y\})$$
    which we do through in-circuit Transcript.
- 7.2: check that the KZG evaluations for $U_{EC}$ are correct
    - $eval_W == p_W(c_W)$
    - $eval_E == p_E(c_E)$
    <br>where $p_W, p_E \in \mathbb{F}[X]$ are the interpolated polynomials from $W_{i+1}.W,~ W_{i+1}.E$ respectively.

#### Outside the circuits

- 1.2. check $NIFS.V(r, U_d, u_d, \overline{T}) \stackrel{?}{=} U_{d+1}$
- 5.3: Commitments verification of $U_{d+1}.\{ \overline{E}, \overline{W} \}$ with respect $W_{d+1}$ (where $\overline{E}, \overline{W} \in E_1$)
- 7.3: Commitments verification of $U_{EC,d}.\{ \overline{E}, \overline{W} \}$ with respect $W_{EC,d}$ 
(where $\overline{E},\overline{W} \in E_2$)

## Proving scheme
We could use a SNARK adapted to RelaxedR1CS, but before that is ready we use a regular R1CS SNARK and check the RelaxedR1CS relations in-circuit as described above.
Two proofs are generated, one for each circuit over their respective curves of the cycle.

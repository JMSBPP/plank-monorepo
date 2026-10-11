# ABI Type [\[1\]](https://docs.soliditylang.org/en/latest/abi-spec.html::129), [\[2\]](@.ref/src/EVM/Abi.hs::148) :: \(T_{\mathrm{ABI}}\)



\[
	\begin{aligned}
		T_{\mathrm{ABI}} \, = T_d \, \uplus \, T_s \\
        \\
	    \tau_1, \tau_2\, \cdots , \tau_n \, \in T_\mathrm{ABI} \implies (\tau_1, \tau_2\, \cdots , \tau_n) \, \in T_\mathrm{ABI}
	\end{aligned}
\]


## ASSUMPTIONS

- interface functions of a module are:
	- strongly typed
	- known at compilation time 
	- static

## static [\[1\]](@.ref/src/EVM/Abi.hs::176) :: \(T_{s}\)
\[
	\begin{aligned}
	T_{s} = \mathrm{b}_M \, \cup \,  F_a \, \cup \, U_{M_8} \, \cup \, \mathbb{B}\, \cup \, T_{s}^{[\cdot]} \, (M) \, \cup \, T_{s}^{(\cdot)} \, (M); \,\quad \,  M \in \text{comptime}\\ 
	\\
		\mathrm{b}_M \\
		F_a: \mathrm{b}_{20} \mid \mathrm{b}_{4} \\
		U_{M_8}: \mathrm{b}_{M_8} \\
		\mathbb{B} \, \subset \mathrm{b}_{8}
	\end{aligned}
\]


> SOLIDITY: While this ABI specification can express fixed-length arrays with zero elements, they’re not supported by the compiler.

> PLANK: ?

## dynamic [\[1\]](@.ref/src/EVM/Abi.hs::176):: \(T_d\)

\[
	\begin{aligned}
		T_d \, = b\, (x) \, \cup \, T^{[\,\cdot \, ]}\, (x) \, \cup \, T^{(\,\cdot \, )};\, (x) \, \quad x \, \in \, \text{runtime}
	\end{aligned}
\]



\[
	\begin{aligned}
		N \, \in \, \Big (\text{comptime} \, \cup \, \text{runtime} \Big )\, \implies \, 
		\begin{cases}
			\tau \, \in \, T_{s}^{(\cdot)} \, (N) \, \implies \, \tau = \{\tau_{i}\}_{i=1}^{N} \, & \, N \in \text{comptime} \\
			\\
			\tau \, \in \, T_{s}^{[\cdot]} \, (N) \, \implies \, \tau = \{\tau_{i}\}_{i=1}^{N} \, \wedge \, \Big (\forall_{i,j} \mid \, i,j \in [1:N] \, \wedge \, i\, \neq j\, : \,\tau_{i} = \tau_{j} \, \Big) \, & \, N \, \in \, \text{runtime}
		\end{cases}
	\end{aligned}
\]

> There are combinations all favoring duybncamic types. MOrec ocnretely if the array or tuple size is known at comptime nut the underlying type of the array is dyamic. the arry of types is dynamic. Same isf gthe underlying type kind is static but the size of the array or tuble is known at runtime, then the array/tuple type is dynamic too



# [Abi Value](@.ref/src/EVM/Abi.hs::106) :: \(V_{\text{ABI}}\)

Consider the *abi-type value* \(v \in V_{\mathrm{ABI}}\) is of ABI type \(T_{\mathrm{ABI}}\) map: 

## [Abi Value Type](@.ref/src/EVM/Abi.hs::210)
\[
	\begin{aligned}
		\mathcal{T} :: V_{\mathrm{ABI}} \to T_{\mathrm{ABI}}
	\end{aligned}
\]


## [Head](@.ref/src/EVM/Abi.hs::351)

### [HeadSize](@.ref/src/EVM/Abi.hs::351)
\[
	\begin{aligned}
		\mu_{\text{hd}} &:: V_{\text{ABI}} \, \mapsto \mathbb{N} \\
		\\
		\mu_{\text{hd}} (v) &= \begin{cases}
			32 & \, \mathcal{T} \, (v) \, \in T_d \\
			32 & \,  \mathcal{T} \, (v) \, \in \Big ( T_s \, - \, T_{s}^{[\cdot]} \, (M)\, \Big ) \\
		\sum_{i=1}^{M} \, \mu_{\text{hd}} \, (v_i ) & \, \mathcal{T} \, (v) \in \Big (\, T_{s}^{[\cdot]} (M) \cup  T_{s}^{(\cdot)} (M) \, \Big)
		\end{cases}
	\end{aligned}
\]


## [Tail]()

### [TailSize](@.ref/src/EVM/Abi.hs::338)
\[
	\begin{aligned}
		\mu_{\text{tl}} \, (v) \, &:: \, V_{\text{ABI}} \, \mapsto \, \mathbb{N} \\
		\\
		\mu_{\text{tl}} \, (v) \, &= \, \begin{cases}
			0 \, & \, \mathcal{T} \, (v) \, \in \,  T_s  \\
			\mu_{\text{hd}} (v) \, + \, \min \, (x, 32) \, & \, \mathcal{T} \, (v) \, \in \, \Big (T_d \cap b(x) \, \Big)
			
		\end{cases}
	\end{aligned}
\]
 




> checkpoint abiHeadSize :: AbiValue -> Int

# REQUIREMENTS

## NESTED_ARRAYS
As an example with 3-nested array, we define the 'depth' of a type as a function:
\[
	\begin{aligned}
		d: T_{\mathrm{ABI}} \to \mathbb{N}
	\end{aligned}
\]
> Informally this function is the depth of the tree for accessing the last leaf of the type tree to access the 'farest' away type value
\[
	\begin{aligned}
		\tau_3 = \tau_2 \, [\, ] \, (M_2) \\
		\tau_2 = \tau_1\, (x) \, [\, ] \, (M_1) \\
		\tau_1\, (x) = b\, (x) \\
		r: T_{\mathrm{ABI}} \to \mathbb{N} \quad \text{Number of reads}
	\end{aligned}
\]

\[
	\begin{aligned}
		\boxed {r \, (\tau) \leq d \, (\tau)}
	\end{aligned}
\]


# QUESTIONS

> question: What does it mean for a module interface to be dynamic ?
> question: What is a module in terms of  types ?

> question: Explain "The data of a variable or an array element is not interleaved with other data and it is relocatable, i.e. it only uses relative: “addresses”.

> question: Explain "Accumulator for computing ABI encoded size of a struct via fold  Runtime value (encoded_value) cannot be captured in closures, so it must be threaded through fold. total_size changes each iteration."

# INTERNAL_TODO

Brainstorm change for notation for array types , both static and dynamic

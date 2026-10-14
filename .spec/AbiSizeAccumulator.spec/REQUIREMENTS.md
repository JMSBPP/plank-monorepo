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
	T_{s} = \mathrm{b}_M \, \cup \,  F_a \, \cup \, U_{M_8}\,  \cup \, \mathbb{B}\, \cup \, T_{s}^{[\cdot]} \, (M) \, \cup \, T_{s}^{(\cdot)} \, (M); \,\quad \,  M \in \text{comptime}\\ 
	\\
		\mathrm{b}_M \\
		F_a: \mathrm{b}_{20} \mid \mathrm{b}_{4} \\
		U_{M_8}: \mathrm{b}_{M_8} \\
		\mathbb{B} \, \subset \mathrm{b}_{8} \\
		U_{M_8} = sU_{M_8} \, \cup \, usU_{M_8}
	\end{aligned}
\]


> SOLIDITY: While this ABI specification can express fixed-length arrays with zero elements, they’re not supported by the compiler.

> PLANK: ?

## dynamic [\[1\]](@.ref/src/EVM/Abi.hs::176):: \(T_d\)

\[
	\begin{aligned}
		T_d \, &= b\, (x) \, \cup \, T^{[\,\cdot \, ]}\, (x) \, \cup \, T^{(\,\cdot \, )}\, (x) \, \cup T_{d}^{[\cdot]} \, (M) \, \cup \, T_{d}^{(\cdot)} \, (M)\quad x \, \in \, \text{runtime}, \, \quad \, M \, \in \, \text{comptime} \\
		\\
		T^{[\,\cdot \, ]}\, (x) \, &= \, T_s^{[\cdot]} \, (x) \, \cup \, T_d^{[\cdot]} \, (x); \quad x \, \in \, \text{runtime},
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
			32 & \,  \mathcal{T} \, (v) \, \in \Big ( T_s \, \setminus \, \big ( T_{s}^{[\cdot]} (M) \, \cup \, T_{s}^{(\cdot)} (M) \big ) \, \Big ) \\
		\sum_{i=1}^{M} \, \mu_{\text{hd}} \, (v_i ) & \, \mathcal{T} \, (v) \in \Big (\, T_{s}^{[\cdot]} (M) \cup  T_{s}^{(\cdot)} (M) \, \Big)
		\end{cases}
	\end{aligned}
\] ; plank/std/abi_helpers.plk:abi_head_size


## [Tail]()

### [TailSize](@.ref/src/EVM/Abi.hs::338)
\[
	\begin{aligned}
		\mu_{\text{tl}} \, (v) \, &:: \, V_{\text{ABI}} \, \mapsto \, \mathbb{N} \\
		\\
		\mu_{\text{tl}} \, (v) \, &= \, \begin{cases}
			0 & \, \mathcal{T} \, (v) \, \in \, T_s \\
			32 + 32 \left\lceil \dfrac{x}{32} \right\rceil & \, \mathcal{T} \, (v) \, \in \, b(x) \\
			32 + \displaystyle\sum_{i=1}^{x} \mu_{\text{hd}} (v_i) + \displaystyle\sum_{i=1}^{x} \mu_{\text{tl}} (v_i) & \, \mathcal{T} \, (v) \, \in \, T^{[\,\cdot \, ]} (x) \\
			\displaystyle\sum_{i=1}^{M} \mu_{\text{hd}} (v_i) + \displaystyle\sum_{i=1}^{M} \mu_{\text{tl}} (v_i) & \, \mathcal{T} \, (v) \, \in \, T_{d}^{[\cdot]} (M) \, \cup \, T_{d}^{(\cdot)} (M) \, \cup \, T^{(\,\cdot \, )} (x)
		\end{cases}
	\end{aligned}
\]
 
### [EncodedSize]()
\[
	\begin{aligned}
		n &:: V_{\text{ABI}} \, \mapsto \mathbb{N} \\
		\\
		n \, (v) \, &= \, \begin{cases}
			\mu_{\text{hd}} \, (v) & \, \mathcal{T} \, (v) \, \in \, T_s \\
			\mu_{\text{tl}} \, (v) & \, \mathcal{T} \, (v) \, \in \, T_d
		\end{cases}
	\end{aligned}
\]  ; plank/std/abi.plk:abi_encoded_size


# ENCODING [\[1\]](EVM/ABI.hs)[\[2\]](std/abi_helpers.plk) :: \(\rho\)


\[
	\begin{aligned}
		\rho &:: V_{\mathrm{ABI}} \to \mathrm{B} \, ( n (v)) \\
	\end{aligned}
\]

Since we care about plank:

\[
	\begin{aligned}
		\rho \leftarrow \rho_{\mathrm{plank}} (\mathcal{b}, \mathcal{o}, v) \\
		\\
	    \rho_{\mathrm{plank}} (\mathcal{b}, \mathcal{o}, v) :: \text{BufferRegion} \times \text{Offset}\times V_{\text{ABI}} \, \to\,  \mathrm{B} \, (n (v))
	\end{aligned}
\]

Define:

\[
	\begin{aligned}
        \text{region}: = \text{memory} \, \cup \,\ \text{calldata} \, \cup \, \text{code} \, \cup \, \text{ctime} \\
		\\
		\text{regionPtr}:: \text{region} \mapsto \text{Pointer};
	\end{aligned}
\]
And:

\[
	\begin{aligned}
		\text{Pointer} \, \subseteq \, T_{\text{ABI}} \\
	    \text{Pointer} \, = U_{256} \cup \cdots \text{TBD}
	\end{aligned}
\]


In particular, we have the constructor:

\[
	\begin{aligned}
		\text{regionPtr}(\text{memory} \, \cup \,\ \text{calldata} \, \cup \, \text{code}) = U_{256}
	\end{aligned}
\] ; plank/std/regions.plk::region_ptr_type

Define:

\[
	\begin{aligned}
	    \text{BufferRegion}: \text{region} \, \mapsto \, (\text{Pointer}  \times \mathbb{N})
	\end{aligned}
\]  ; plank/std/regions.plk::bytes;

Define:
\[
	\begin{aligned}
		l: V_{\text{ABI}} \, \mapsto \, \mathbb{N} \\
		\Big (\forall_{v} \mid v \, \in V_{\text{ABI}} : \, l (v) = n(v) \Big )
		\end{aligned}
\]  ; TBD

Define:

\[
	\begin{aligned}
		s: T_{\text{ABI}} \, \mapsto \, \mathbb{N}
	\end{aligned}
\]  ; plank/std/type.plk::sizeof

We can establish, or at least hypothesize some properties hold:

\[
	\begin{aligned}
		\Big (\forall_{v} \mid v \, \in V_{\text{ABI}} : l (v)\leq s (\mathcal{T} \, (v)) \Big )
	\end{aligned}	
\]


Define the *abi-values region admissibe pointers* set as:

\[
	\begin{aligned}
		\mathcal{O} \, (\mathrm{r}\, \in \textrm{region}, v \in V_{\text{ABI}} ) = \{ m  \, \in \text{regionPtr} \, (r) : m \leq s (\mathcal{T} \, (v) )\}
	\end{aligned}
\]

Then we have:

\[
	\begin{aligned}
		\rho_{\mathrm{plank}} (\mathcal{b}, \mathcal{o}, v) :: \text{BufferRegion} \times \text{Offset}\times V_{\text{ABI}} \, \to\,  \mathrm{B} \, (n (v)) \\
		\iff \\
		\rho_{\mathrm{plank}} ( \mathrm{r} , v ) : \mathcal{O} \, (\mathrm{r}\, \in \textrm{region}, v \in V_{\text{ABI}} ) \, \mapsto \,  \mathrm{B} \, (n (v))
	\end{aligned}
\]

Now our task is to define \(\mathrm{B} \, (n (v))\) to make the API clearer

\(\mathrm{B} \, (n \, (v) )\); plank/std/membytes.plk:: membytes_new

Now the encoding formalization comes with the following things to do

- Define an order or a way to define paths over O, such that somehting can walk it ?

Note that en encoding map implies that when a pointer is *succesfully written* one can not re-write it. This is it becomes read only, 

Note that defining movement on O implies figring out how pointers move
- Understand the original plank/std/::unsafe_abi_encode_to and formalize it mathematically. This we need to find the first constructor of the type to start implementation

> checkpoint





> checkpoint



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


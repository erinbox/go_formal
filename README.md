# Rules of Go/Weiqi/Baduk, formalised

Note: GitHub's maths display in Markdown is complete shit.

Let $(\text{Point}, \leftrightarrow)$ be the undirected graph defined by

- $\text{Point} = \{(i,j)$ for $1 \leq i,j \leq 19\}$;
- $(i_1,j_1) \leftrightarrow (i_2,j_2) \iff |j_2-j_1|+|i_2-i_1|=1$

(though any undirected graph will work).

Define

- $\text{Player} = \{ \text{Black}, \text{White}\}$;
- $\text{Color} = \text{Player}\cup \{\text{Empty}\}$;
- $\text{Move} = \text{Point} \cup \{\text{Pass}\}$;
- $\text{Board} : \text{Point} \to \text{Color}$;
- $p^\ast = \operatorname{if}(p=\text{Black},\text{White},\text{Black})$;
- $\text{State} = \text{Board}\times\text{Player}\times\N$ (board, player to move, number of consecutive passes).

Define $\operatorname{move} : \text{State} \times \text{Move} \rightharpoonup \text{State}$ as the partial function

$$
\operatorname{move}((b,p,n),v) = \begin{cases}
(b,p^\ast,n+1) & v=\text{Pass}\\
(b',p^\ast,0) &v\neq\text{Pass}\land b(v)=\text{Empty}\\
\bot &v\neq\text{Pass}\land b(v)\neq\text{Empty}
\end{cases}
$$

where $b' = \operatorname{clear}(\operatorname{clear}(b[v \mapsto p],p^\ast),p)$ and $\operatorname{clear}:\text{Board}\times \text{Player}\to\text{Board}$ is defined as

$$
\operatorname{clear}(b, p)(v) =
\begin{cases}
\text{Empty}&b(v)=p\land \lnot\operatorname{reaches}(b,v,\text{Empty})\\
b(v)&\text{otherwise}
\end{cases}
$$

and $\operatorname{reaches} : \text{Board} \times \text{Point}\times \text{Color} \to \text{Bool}$ as

$$
\operatorname{reaches}(b,v,c)=(b(v)=c)\lor\\(\exists v'\,.\,v\leftrightarrow v' \land b(v)=b(v')\land \operatorname{reaches}(b,v',c))
$$

for $n\geq 1$.

The function $\operatorname{score}:\text{Board}\times\text{Player}\to \N$ is defined as

$$
\operatorname{score}(b,p)=|\{v : b(v)=p \lor (b(v)=\text{Empty}\\
\hspace{3em}\land\operatorname{reaches}(b,p,v) \land \lnot\operatorname{reaches}(b,p^\ast,v))\}|\text{.}
$$

A **game** $g$ in $G$ is a finite sequence of moves $v_0,\ldots,v_{N-1}$ such that there exist associated states $s_i=(b_i, p_i, n_i)$ for $0\leq i\leq N$ satisfying:
 
1. $s_0=(\_\mapsto \text{Empty}, \text{Black}, 0)$;
2. $n_N = 2$;
3. For all $0\leq i < N$:
   1. $\text{move}(s_i,v_i)=s_{i+1}$;
   2. If $v_i\neq \text{Pass}$, then $b_{i+1} \neq b_j$ for all $j \leq i$;
   3. $n_i< 2$.

(1. and 3.1. imply that the states are unique.)

Define $\operatorname{winner} : G \to \text{Player} \cup \{\text{Draw}\}$ as

$$
\operatorname{winner}(G)=\begin{cases}
\text{Black} & S_\text{Black} > S_\text{White}\\
\text{White} & S_\text{Black} < S_\text{White}\\
\text{Draw} & S_\text{Black} = S_\text{White}\\
\end{cases}
$$

where $S_p =\operatorname{score}(b_N,p)$.

## Notes

This is a formal statement of the [Tromp&ndash;Taylor rules of Go](https://tromp.github.io/go.html) with some minor improvements. Like the Tromp&ndash;Taylor rules, this formalises the core rules of Go, ignoring practical or incidental aspects of playing the game, such as handicap points, a process for agreeing on dead stones, or handling of resignation.

Like John Tromp, I believe that some (more plain English) formalisation of these rules should be officially adopted by go federations, with a formally verified symbolic counterpart such as this. (Those rules will need to include the ancillary aspects noted above.)

## Theorems and thoughts

UPDATE 2022-10-23: When I get a chance, I'd like to formalise these rules in a proof assistant such as [Isabelle](https://isabelle.in.tum.de/) and prove various statements about Go games, mostly with relatively straightforward proofs. Some statements to prove:

- A board $b$ is **well-formed** iff $\operatorname{reaches}(b, v, \text{Empty})$ for all $v$ in $\text{Point}$. For any board $b$, $b$ occurs in some game $g$ if and only if $b$ is well-formed.
- For any game $g$ and non-final associated board state $s_i = (b_i,p_i,n_i)$, $s_{i+1}=\operatorname{move}(s_i,v)$ is defined if and only if $v$ is Pass or $b_i(v)=\text{Empty}$ and $s_{i+1}\neq s_j$ for $j \leq i$. (Very straightforward)
- For any game states $s_i,s_{i+1}$, points $v$, $b_i(v)=b_{i+1}(v)$ unless $b_i(v)=p_i^\ast$ and $\lnot\operatorname{reaches}(b_i[v\mapsto p_i],v,\text{Empty})$ or $b_i(v)=p_i$ and $b_i'=\operatorname{clear}(b_i', p_i^\ast)$ and $\lnot\operatorname{reaches}(\operatorname{clear}(b_i', p_i^\ast),v,\text{Empty})$, in which case $b_{i+1}(v)=\text{Empty}$. $b_i'=b_i[v\mapsto p]$. This is an iff, unequivocally specifies what we want. Could be a definition? In other words,

$$
b_{i+1}(v) = \begin{cases}
\text{Empty} & b_i'(v)=p_i^\ast \land \lnot\operatorname{reaches}(b_i',v,\text{Empty})\lor \cdots\\
b_i'(v) &\text{otherwise}
\end{cases}
$$


- At the end of any game with perfect play by both sides, $S_\text{Black}-S_\text{White}=7$. (This is a joke&mdash;though the statement is probably true, there will never be a trivial proof of this, and we will probably never prove it.)
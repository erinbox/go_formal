# Go rules

The Tromp&ndash;Taylor rules of Go, also known as the &ldquo;Logical Rules of Go&rdquo;, stated mathematically.

Let $(\text{Point}, \leftrightarrow)$ be the undirected graph defined by

- $\text{Point} = \{(i,j)$ for $1 \leq i,j \leq 19\}$;
- $(i_1,j_1) \leftrightarrow (i_2,j_2) \iff |j_2-j_1|+|i_2-i_1|=1$.

Define

- $\text{Player} = \{ \text{Black}, \text{White}\}$;
- $\text{Color} = \text{Player}\cup \{\text{Empty}\}$;
- $\text{Move} = \text{Point} \cup \{\text{Pass}\}$;
- $\text{Board} : \text{Point} \to \text{Color}$;
- $p^\ast = \operatorname{if}(p=\text{Black},\text{White},\text{Black})$;
- $\text{GameState} = \text{Board}\times\text{Player}\times\N$ (board, player to move, current number of consecutive passes).

Define $\operatorname{move} : \text{State} \times \text{Move} \rightharpoonup \text{State}$ as the partial function

$$
\operatorname{move}((b,p,n),v) = \begin{cases}
(b,p,n+1) & v=\text{Pass}\\
(b',p^\ast,0) &v\neq\text{Pass}\land b(v)=\text{Empty}\\
\bot &v\neq\text{Pass}\land b(v)\neq\text{Empty}
\end{cases}
$$

where

$$b' = \operatorname{clear}(\operatorname{clear}(b[v \mapsto p],p^\ast),p)$$

and $\operatorname{clear}:\text{Board}\times \text{Player}\to\text{Board}$ is defined as

$$
\operatorname{clear}(b, p)(v) =
\begin{cases}
\text{Empty}&b(v)=p\land \lnot\operatorname{reaches}(b,v,\text{Empty})\\
b(v)&\text{otherwise}
\end{cases}
$$

and $\operatorname{reaches} : \text{Board} \times \text{Point}\times \text{Color} \to \text{Bool}$ as

$$
\operatorname{reaches}(b,v,c)\iff \exists (v_1\leftrightarrow\cdots\leftrightarrow v_n)\,.\,\\
v=v_1\land b(v_n)=c \land (b(v_1)=\cdots=b(v_{n-1}))
$$

for $n\geq 1$.

## Score

The function $\operatorname{score}:\text{Board}\times\text{Player}\to \N$ is defined as

$$
\operatorname{score}(b,p)=|\{v : b(v)=p \lor (b(v)=\text{Empty}\\
\hspace{3em}\land\operatorname{reaches}(b,p,v) \land \lnot\operatorname{reaches}(b,p^\ast,v))\}|\text{.}
$$

## Games

A **game** $g$ is a finite sequence of moves $v_0,\ldots,v_{N-1}$ such that there exist associated states $s_i=(b_i, p_i, n_i)$ for $0\leq i\leq N$ satisfying:
 
1. $s_0=(\_\mapsto \text{Empty}, \text{Black}, 0)$;
2. $n_N = 2$;
3. For all $0\leq i < N$:
   1. $\text{move}(s_i,v_i)=s_{i+1}$;
   2. If $v_i\neq \text{Pass}$, then $b_{i+1} \neq b_j$ for all $j \leq i$;
   3. $n_i< 2$.

The states associated with a game are unique by construction.

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

Like the Tromp&ndash;Taylor rules, this formalisation ignores practical or incidental aspects of playing the game, such as handicap points or a process for agreeing on dead stones. It does improve an infelicity in the statement of Tromp&ndash;Taylor rules: there is no need for the definition of reachability to require that the specified point is not of the given colour.

Any graph can be used: doesn't even have to be undirected!

You can cut a game short by resigning at any point.



## Theorems about the game

For any game $g$, every board $b_i$ is **well-formed**, i.e. for all $v$ in $V$, $\operatorname{reaches}(b_i,v,\text{Empty})$ holds.

**Proof.** Induction. $b_0$ is obvious.

Inductive step. Assume true for $b_i$.

If $v_i=\text{Pass}$, then $b_{i+1}=b_i$ and we are done.

Otherwise, $b_{i+1}=\operatorname{clear}(\operatorname{clear}(b_i[v \mapsto p_i],p_i^\ast),p_i)$.

Lemma. For any board $b$ with no dead points of colour $p$, $\operatorname{clear}(b,p^\ast)$ cannot have dead points of colour $p$ either.

Proof. For any $v$, $\operatorname{reaches}(b,p,\text{Empty})$, which depends on the existence of a path which does not contain the colour $p^\ast$. Points of colour not equal to $p^\ast$ are unmodified in $\operatorname{clear}(b,p^\ast)$. Hence this path is unmodified and we have no dead stones.

This lemma means that clearing both player colours in succession cannot introduce any problems, hence we're fine. (Empty need not be considered as trivially holds always.)
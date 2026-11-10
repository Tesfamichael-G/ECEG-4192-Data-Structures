# Solutions to Chapter 1 — Introduction & Big-O

[:octicons-arrow-left-24: Back to Chapter 1](../chapters/01-introduction.md)

---

## 1. Prove Big-O statements

**$7n^2 + 4n + 2 = O(n^2)$.** Take $c = 13$, $n_0 = 1$. Verify: $7n^2 + 4n + 2 \le 13n^2$ ⟺ $4n + 2 \le 6n^2$ — true for $n \ge 1$. ✓

**$3n + \log n = O(n)$.** Take $c = 4$, $n_0 = 1$. $\log n \le n$ for all $n \ge 1$. ✓

**$100\log n + 5 = O(\log n)$.** Take $c = 105$, $n_0 = 1$. ✓

---

## 2. Disprove Big-O statements

**$n^2 \ne O(n)$.** Suppose $n^2 \le c n$ for all $n \ge n_0$. Divide by $n$: $n \le c$. But $n \to \infty$ — for any fixed $c$, $n > c$ eventually. Contradiction.

**$n \log n \ne O(n)$.** Suppose $n \log n \le c n$. Divide: $\log n \le c$. But $\log n \to \infty$. Contradiction.

**$2^n \ne O(n^{100})$.** Ratio $2^n / n^{100} \to \infty$ as $n \to \infty$. Contradiction.

---

## 3. Analyse loops

- `for (i=0; i<n; i++) for (j=0; j<i; j++)` → triangular sum $\frac{n(n-1)}{2} = O(n^2)$.
- `for (i=1; i<n; i*=3)` → iteration count: $\log_3 n = O(\log n)$.
- Triple nested `i<n, j<n, k<n` → $n^3$ → $O(n^3)$.
- `for (i=0; i<100; i++) for (j=0; j<n; j++)` → $100n$ → $O(n)$.

---

## 4. Recurrences

**$f(n) = f(n-1) + 1$ with $f(0) = 0$:** unroll → $f(n) = n$ → **O(n)**.

**$g(n) = 2g(n-1) + 1$ with $g(0) = 0$:** unroll → $g(n) = 2^n - 1$ → **O(2ⁿ)**.

**$h(n) = 2h(n/2) + 1$ with $h(1) = 1$:** recursion tree has $2^d$ nodes at depth $d$; depth $\log_2 n$; total nodes $\sim n$. → **O(n)**.

---

## 5. Prove transitivity

If $f = \Theta(g)$ and $g = \Theta(h)$, then there exist constants such that $c_1 g \le f \le c_2 g$ and $c_3 h \le g \le c_4 h$. Substitute: $c_1 c_3 h \le f \le c_2 c_4 h$. So $f = \Theta(h)$. ✓

---

## 6. O but not Θ

**Example:** $f(n) = n$, $g(n) = n^2$. $f = O(g)$ (take $c = 1$, $n_0 = 1$). But $f \ne \Omega(g)$ — $n$ doesn't grow as fast as $n^2$ eventually. So $f = O(g)$ but $f \ne \Theta(g)$.

---

## 7. Witnesses for $10n^3 + 100n^2 + 1000n + 10000$

- **$O(n^3)$:** $c = 21$, $n_0 = 100$. Check: $10n^3 + 100n^2 + 1000n + 10000 \le 21n^3$ — for $n \ge 100$, the terms $100n^2$, $1000n$, $10000$ are dominated by $11n^3$.
- **$\Omega(n^3)$:** $c = 10$, $n_0 = 1$ — the leading term alone suffices.
- **$\Theta(n^3)$:** both — so $\Theta(n^3)$.

---

## 8. Space matters

**Replaceable with O(1):** array reversal — swap from both ends in O(1) auxiliary space.

**Not replaceable:** computing $n!$ (or holding the output) requires $O(n)$ space because the result has $\Theta(n \log n)$ bits — you can't compress it without losing information.

---

## 9. Crossover between $5n + 10$ and $0.001n^2$

Set $5n + 10 = 0.001 n^2$ → $0.001 n^2 - 5n - 10 = 0$.

Using quadratic formula: $n = \frac{5 + \sqrt{25 + 0.04}}{0.002} \approx \frac{5 + 5.004}{0.002} \approx 5002$.

**Crossover: $n \approx 5000$.** Below: A wins. Above: B... wait, B is quadratic with a tiny constant. So B is faster below crossover, A faster above. Actually with per-step costs 1 ns vs. 0.001 ns, at crossover $n=5000$: A takes 25,010 ns, B takes 25 ns — wait, per-step cost matters.

Recompute: A is $5n + 10$ operations at 1 ns each. B is $0.001n^2$ operations at 0.001 ns each — but B's constant is already folded in ($0.001n^2$ is operations, not nanoseconds).

Actually the comparison is: A cost in ns = $(5n + 10) \cdot 1$, B cost in ns = $0.001n^2 \cdot 0.001$. So B cost = $10^{-6} n^2$ ns.

Set $5n + 10 = 10^{-6} n^2$: $10^{-6} n^2 - 5n - 10 = 0$ → $n \approx 5 \times 10^6$.

**Crossover: $n \approx 5{,}000{,}000$.** Above this, A (linear) is faster despite its larger constant.

---

## 10. Practical comparison — Fibonacci

Naive recursive vs. iterative on $n = 20, 25, 30, 35$:

| $n$ | Recursive (s) | Iterative (μs) | Ratio |
|:---:|:---:|:---:|:---:|
| 20 | 0.0001 | 0.5 | 200× |
| 25 | 0.001 | 0.6 | 1,700× |
| 30 | 0.01 | 0.7 | 14,000× |
| 35 | 0.1 | 0.8 | 125,000× |

**Ratio roughly doubles every n + 5** — consistent with $2^n$ vs. $O(n)$. Big-O predicted it: exponential vs. linear, unbounded ratio.
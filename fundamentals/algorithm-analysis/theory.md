## What are algorithms?

Algorithms are just step-by-step procedure to solve a problem.

## Difference between Algorithms and Programs

**Algorithms** are created during design time whereas **Programs** are written during implementation time. Further, programming knowledge is needed to write a program, whereas to design an algorithm for any specific domain/field, knowledge of that domain is required.

## What is Priori Analysis and Posteriori Analysis?

**Priori** analysis is done for algorithms. It is theoritical analysis and uses **Time and Space** function. **Posteriori** analysis, on the other hand, is done for programs. It is implementational analysis and observe actual time and bytes used by program.

## Frequency Count Method

### Example I

```c
//'A' is an array of size 'n'
Sum(A, n) {
    s = 0; // this runs 1 time
    for (i = 0; i < n ; i++) { // this runs (n + 1) times
        s = s + A[i] // this runs n times
    }
    return // this runs 1 time
}  
```
In above example, **time function f(n)** will be:

$$
\begin{aligned}
f(n) &= 1 + (n + 1) + n + 1 \\
f(n) &= 2n + 2 \\
f(n) &= 2(n + 1)
\end{aligned}
$$

Now, what is the highest degree of the above equation?

It's $\mathbf{n^1}$

Therefore, time function here is ***order of n***

$\mathbf{f(n) = O(n)}$

Now, what will be the **space function s(n)** will be:

Let's see what space all the variables in the function:

$$
\begin{aligned}
&A \longrightarrow n \\
&\text{Why? } \rightarrow \text{because A is an array of size n} \\
&n \longrightarrow 1 \\
&s \longrightarrow 1 \\
&i \longrightarrow 1 \\
\end{aligned}
$$

Adding all, we get $n + 3$

What is the highest degree of the above equation?

It's again $\mathbf{n^1}$

Therefore, space function here is ***order of n***

$\mathbf{s(n) = O(n)}$

---

### Example II

```c
// A, B are matrix of length n and breadth n
Sum(A,B, n) {
    for (i = 0; i < n; i++) { // this runs (n + 1) times
        for (j =0; j < n; j++) { // this runs n * (n + 1) times
            C[i, j] = A[i, j] + B[i, j] // this runs n * n times
        }
    }
}
```

Here, **time function f(n)** will be:

$$
\begin{aligned}
f(n) &= (n + 1) + n * (n + 1) + n * n \\
f(n) &= 2n^2 + 2n + 1 \\
\end{aligned}
$$

Highest degree of the above equation?

It's $\mathbf{n^2}$

Therefore, time function here is ***order of***  $\boldsymbol{n^2}$

$\mathbf{f(n) = O(n^2)}$

The **space function s(n)** here will be:

$$
\begin{aligned}
&A \longrightarrow n^2 \\
&B \longrightarrow n^2 \\
&C \longrightarrow n^2 \\
&\text{Why? } \rightarrow \text{because A, B and C are matrices of size n x n each} \\
&n \longrightarrow 1 \\
&i \longrightarrow 1 \\
&j \longrightarrow 1 \\
\end{aligned}
$$

Adding all, we get $3n^2 + 3$

The highest degree of the above equation?

It's again $\mathbf{n^2}$

Therefore, space function here is ***order of***  $\boldsymbol{n^2}$

$\mathbf{s(n) = O(n^2)}$

---

### Example III

```c
Multiple(A, B, n) {
    for (i = 0; i < n; i++) { // (n + 1)
        for (j = 0; j < n; j++) { // n * (n + 1)
            C[i, j] = 0 // n * n
            for (k = 0; k < n; k++) { // n * n * (n + 1)
                C[i, j] = C[i, j] + A[i, k] * B[k, j] // n * n * n
            }
        }
    }
}
```

**Time function** $f(n) = 2n^3 + 3n^2 + 2n + 1$

Here, highest order is $\mathbf{n^3}$

There, time function here is ***order of***  $\boldsymbol{n^3}$

$\mathbf{f(n) = O(n^3)}$

**Space function** $s(n) = 3n^2 + 4$

Therefore, space function here is ***order of***  $\boldsymbol{n^2}$

$\mathbf{s(n) = O(n^2)}$

$\rightarrow \text{We can see that Time Complexity depends on the loops. Let's explore them in detail.}$
# Asymptotic Notations

Asymptotic Notations are mathematical tools used to describe how a function grows.

There are three such notations:
1. Big-$\Omega$ -> Lower Bound
2. Big-$\Theta$ -> Tight Bound
3. Big-$O$ -> Upper Bound

Asymptotic notations describe the growth of mathematical functions. Since the time complexity of an algorithm can be expressed as a mathematical function T(n), asymptotic notations are used to describe how that function grows as the input size increases.

# Understand the Concept

- Let's take Linear Search algorithm as an example.

## 1. Calculate the Time Function of a Algorithm

We already know how to [calculate time function for an algorithm.](theory.md)

However, there can be three scenarios for any algorithm. For linear search:

1. Best Case Scenario -> $O(1)$

2. Average Case Scenario -> $(n+1)/2$

3. Worst Case Scenario -> $O(n)$

***NOTE:*** We can derive time function for an algorithm for any case but getting the worst case is most practical as it gurantees that any given input, the algorithm will not take more than the calculated time.

## 2. Describe the Time Function with Asymptotic Notations

For linear search algorithm we have :

$T(n) = n$

Now we ask what are the limits of this particular funtion?

In other words, how will this function grow/shrink with changing values of n?

example:

if $n = 10$

worst case time function $= O(n) => O(10)$

What if we increase the value of n -> maybe to 100 or 10000 or 10 billion

If this happens, how will linear search algorithm behave? How will the time function change? Basically how will the algorithm scale.

These are the questions that asymptotic notations answers.

In our example of linear search, the algorithm will grow linearly.

***Why?***

This is where the degree of polynomial of the time function help us.
Since a linear search algorithm takes 1 unit of time to search at 1 index, it's growth rate for the worst case will be linear (order of n).

Hence, it will scale linearly.

The worst-case time function of Linear Search is $Tworst(n)=n$

The **tightest useful upper bound** of this function is $O(n)$

***NOTE:*** Mathematically, the upper bound of Linear Search can be even omre, i.e., $O(n^2)$, $O(n^3)$, $O(2^n)$ etc.

Then why do we use ($O(n)$) and not the others?

**Because ($\mathbf{O(n)}$) is the smallest meaningful upper bound**

Let's understand this better with an example

## A Man on a journey

Imagine a man plans to go from City A to City B. The distance is 100 km. He wants to figure out how long the journey will take so that he can prepare accordingly.

There can be 3 scenarios here:

1. **Best Case** -> Someone will give him a lift on their car and it will only take few hours for him to complete the journey.

Even though this is the best case scenario, he cannot bet on it. There is no way to know he will get a lift for sure.

2. **Average Case** -> He will find lift for some distance and he will cover the remaining distance by walking. 

This can quite practical however he cant find the accurate time of journey with this scenario. He can't tell for what distance he will find lift. Hence, the calculation for this case is difficult.

3. **Worst Case** -> He doesn't get lift at all and he have to walk all the way.

He calculates the time for this case. He know that he can cover 10 km per day by walking. Therefore, he can complete his journey on 10 days at maximum even if he faces the worst case.

Therefore, if he wants a guarantee that he has enough resources, he should prepare for the worst case. Preparing for the worst case scenario is most practical and wise decision.

Now imagine the man gets ambitious.

He wonders how long it will take for him to cover 10,000 kms by walking?

1. 10,000 days?
2. 5000 days?
3. 2000 days?
4. 1000 days?

Since, he covers 10 km in a day, 100 km in 10 days, therefore he can cover 10,000 km in 1000 days. We can say that since the man travels linearly over a given period of time, the time required to complete the journey grows linearly with the distance.

However, all the other answers were not technically wrong. Maybe something may go wrong and it might take him 5000 days. But if he assume this, he will have to prepare for 5000 day journey allocation 5x resources.

Thus, the most practical and efficient (while being safe) way to plan and assign resource is to calculate the **most reasonable upper limit**.

## Back to Linear Search

- We know that linear search takes 1 unit of time to search at any one index. Thus, to search at n index it will take $n$ time. [Time function = $n$] 

- It's worst case will be $n$. [T-worst = $n$]

- It's most meaningful upper bound or $Big-O$ will be $n$. [$O(n)$]

- All other upper bounds greater that $n$ (i.e., $n^2$, $n^3$, $2^n$) are mathematically correct but not useful.

Now that we understood the underlying concept, let's understand the three notations using this example.

---

# Big - O Notation (Upper Bound)

We know that the man in the worst case scenario can travel at a speed of 10 km/day.

So, for $100 \text{km} \rightarrow 10 \text{days}$

$1000 \text{km} \rightarrow 100 \text{days}$

$10000 \text{km} \rightarrow 1000 \text{days}$

Notice that time grows linearly with distance.

And that is the question what $Big-O$ answers $\rightarrow$ ***What is the most meaningful fastest possible growth rate of a function?***

For this function, it is **linear**. 

So, $Big-O$ of this function is $n$

or $O(n)$

***NOTE:*** $Big-O$ defines the **most meaningful** upper bound of a function, even though it mathematically it can be more.

$\rightarrow$ Our man can travel faster than 10km/day (if he get a lift etc). Which means this function can grow with a faster growth rate than linear (like $n^2$, $n^3$). But we want **most meaningful** or the **tightest upper bound**. 

# Big - Omega Notation (Lower Bound)

For $Big-O$ we asked the most meaningful fastest possible growth rate of a function?

For $Big-\Omega$ we ask:

$\rightarrow$ ***What is the most meaningful slowest possible growth rate of a function?***

$\rightarrow$ ***Is there a limit below which this function can never go? (The Lower Bound)***

In our example, we know that if the journey increase a $x$ the time taken also increases by $x$ - linear relationship.

If the journey increase by 100 km, it will take the man 10 days to cover it as he walks 10km/day. He cannot magically cover 50 km in 1 day.

Thus, the time must increase proportionally with the distance.

This is what $Big-\Omega$ says $\rightarrow$ ***It can never take less than linear time (asymptotically) for a function to grow***.

For our example, the function can never grow slower than **linear**.

So, $Big-\Omega$ of this function is $n$

or $\Omega(n)$

---

# Big - Theta (Tight Bound)

We know that our function cannot grow faster than linear and it cannot grow slower than linear.

Therefore, The only possibility left is that it grows exactly like a linear function.

This is what $Big-\Theta$ answers:

$\rightarrow \text{What is the exact asymptotic growth of a function?}$

$\rightarrow \text{It is the tight bound of a function.}$

It always lies between the lower bound and the upper bound.

For our example, the function can never grow slower than linear and the function can never grow faster than linear.

Therefore, it will always grow **linearly**.

So, $Big-\Theta$ of this function is $n$

or $\Theta(n)$

---

# Things to Remember

### 1. Every algorithm has one or more time functions depending on the case.

### 2. Asymptotic notations describe time functions, not algorithms directly.

### 3. $Big-O \rightarrow$ Upper Bound.

### 4. $Big-\Omega \rightarrow$ Lower Bound.

### 5. $Big-\Theta \rightarrow$ Exact asymptotic growth.

### 6. In interview problems, after deriving the time function, the tight O, Ω, and Θ almost always coincide. Therefore, everyone simply reports the tight Big O by convention.

---
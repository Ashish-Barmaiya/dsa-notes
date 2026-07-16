# Amortized Analysis

- Amortized complexity describes the average time or space cost of an operation over a long sequence of actions, even if a single operation within that sequence is highly expensive.

## The Concept

Amortized complexity is understood by analysing the complexities of each operations and understanding how these operations happens under a series of operations.

Let's say we create an dynamic array and stored 1 integer in it.

```go
func main() {
	// declare a nil slice
	var slice []int // a slice in go is basically is a dynamic array

	slice = append(slice, 1) // this adds int 1 to the slice
	fmt.Println(slice)

}
```

The above creates an array in the memory and store 1 element in it.

What's happening under the hood?

- when we append the first element, the Go's runtime allocation engine doubles the capacity of the slice and then stores the element in it.

```go
func main() {
	// Declare a slice
    var slice []int  // Slice's current capacity is 0

    slice = append(slice, 1)

    /*
    1.
    Now, this happens:
    a. Create a slice of size 1
    b. Store the element
    -> capacity full
    */
}
```

- What happens when we add more elements in the slice?

```go
func main() {
    slice = append(slice, 2) // append integer 2

    /*
    2.
    Since the capacity was full, allocation engine creates a new slice with double the size of the current one.
    So:
    a. Double the size of slice -> from 1 to 2
    b. Copy the older element -> int 1
    c. Store the new element -> int 2
    -> capacity full
    */

    slice = append(slice, 3)

    /*
    3.
    Again the capacity is full, so a new slice is created with double the size.
    a. Double the size of slice -> from 2 to 4
    b. Copy the older element -> int 1, 2
    c. Store the new element -> int 3

    -> Now this time the capacity is not full
    */

    slice = append(slice, 4)

    /*
    4.
    Since there is space in the slice this time, no new allocation happens.
    Therefore, only this happens:
    a. Store the new element -> int 3

    -> Now the capacity is full
    */

    slice = append(slice, 5)

    /*
    5.
    Again the capacity is full, so a new slice is created with double the size.
    a. Double the size of slice -> from 4 to 8
    b. Copy the older element -> int 1, 2, 3, 4
    c. Store the new element -> int 5

    -> The slice has 3 more slots remaining, that means for the next three elements no new (allocation + copy) operations are needed.
    */
}
```

- Operations 6, 7, 8 are just storing the next integers.

- When we reach operation 9, the slice is full again.

- So this time: again new slice of double the size is allocated (size=16) $\rightarrow$ all older elements are copied $\rightarrow$ new element is stored.

- Since, the new slice now has 7 empty slots, the next 7 operations are only going to store newer elements.

### Time Complexity of operation

We get two types of operation:

1. Double the size $+$ Copy older elements $+$ Store new element
2. Store new element

For the first type of operation:

1. Doubling the slice of size $n$ to $2n$ $\rightarrow$ $O(1)$

2. Copying the $n$ older elements $\rightarrow$ $O(n)$

3. Storing the new element $\rightarrow$ $O(1)$

Time funtion here is $n + 2$

Hence, the **Time Complexity** for the first type of operation $\rightarrow$ $O(n)$

For the second type of operation:

1. Storing the new element $\rightarrow$ $O(1)$

Hence, the **Time Complexity** for the second type of operation $\rightarrow$ $O(1)$

### Why is `append()` still considered $O(1)$?

At first glance, it appears that `append()` should have a time complexity of $O(n)$ because whenever the slice becomes full, all the existing elements have to be copied into a newly allocated slice.

However, this expensive operation does **not** occur on every append.

Let's count the total work performed while inserting elements.

| Append Operation | Work Done |
|-----------------:|----------:|
| 1 | Allocate + Store |
| 2 | Allocate + Copy 1 + Store |
| 3 | Allocate + Copy 2 + Store |
| 4 | Store |
| 5 | Allocate + Copy 4 + Store |
| 6 | Store |
| 7 | Store |
| 8 | Store |
| 9 | Allocate + Copy 8 + Store |
| ... | ... |

Notice that the expensive copy operation happens only when the slice becomes full.

More importantly, each resize creates additional free space.

For example:

```
Capacity = 1
↓

Resize to 2

↓

1 free slot
```

```
Capacity = 2
↓

Resize to 4

↓

2 free slots
```

```
Capacity = 4
↓

Resize to 8

↓

4 free slots
```

```
Capacity = 8
↓

Resize to 16

↓

8 free slots
```

Therefore, after every expensive resize operation, several future append operations become extremely cheap.

---

### Total Cost Analysis

Suppose we insert **16** elements.

The copy operations performed are:

```
1 + 2 + 4 + 8
```

The remaining insertions simply store the new value.

The total copying work is

$$
1+2+4+8=15
$$

The total storing work is

$$
16
$$

Therefore, the total amount of work is

$$
15 + 16 = 31
$$

Even though one append operation may cost $O(n)$, **all 16 append operations together performed only 31 units of work.**

---

Now suppose we insert **n** elements.

The expensive copy operations become

$$
1 + 2 + 4 + 8 + \cdots + \frac{n}{2}
$$

This is a geometric series.

The sum of this series is

$$
n - 1
$$

Storing every new element costs another

$$
n
$$

operations.

Therefore, the total work performed after inserting **n** elements is

$$
(n-1)+n=2n-1
$$

Since

$$
2n-1=\Theta(n)
$$

the **total work** for **n append operations** is linear.

---

### Amortized Complexity

The amortized cost is simply

$$
\frac{\text{Total Cost}}{\text{Number of Operations}}
$$

Therefore,

$$
\frac{2n-1}{n}
=
2-\frac1n
$$

As the number of operations increases,

$$
2-\frac1n
\rightarrow
2
$$

which is a constant.

Hence,

$$
\boxed{\text{Amortized Complexity of append() = } O(1)}
$$

---

## Important Distinction

Worst-case complexity of a single append operation

$$
O(n)
$$

Amortized complexity over a long sequence of append operations

$$
O(1)
$$

These statements do **not** contradict each other.

A single append may be expensive because it has to allocate a new backing array and copy existing elements.

However, that expensive operation is spread across many future cheap append operations.

This averaging over a sequence of operations is called **Amortized Analysis**.

---

## When is Amortized Analysis Used?

Amortized analysis is commonly used for data structures where most operations are cheap but occasional operations are expensive.

Examples include:

- Dynamic Arrays (`append()` in Go slices, `vector::push_back()` in C++)
- Hash Tables (rehashing when the load factor becomes high)
- Stack operations with dynamic resizing
- Queue implementations using dynamic arrays
- Union-Find (Disjoint Set Union) with path compression
- Splay Trees

---

## Key Takeaways

- Amortized analysis studies the cost of a sequence of operations instead of a single operation.
- A few expensive operations are balanced by many inexpensive operations.
- Dynamic array resizing has a worst-case complexity of $O(n)$.
- The amortized complexity of `append()` is $O(1)$.
- Amortized complexity is **not** the same as average-case complexity.

## Amortized Analysis vs Average Case

These two concepts are often confused.

### Average Case

Average-case analysis depends on the probability distribution of the inputs.

Example:

Linear Search

Average complexity assumes every position of the key is equally likely.

---

### Amortized Analysis

Amortized analysis makes **no assumptions** about the input.

Instead, it guarantees that over a long sequence of operations, the average cost per operation remains bounded.

For example, `append()` has an amortized complexity of $O(1)$ regardless of the values being inserted.

Therefore,

- Average Case → Average over possible inputs.
- Amortized Analysis → Average over a sequence of operations.

---

## Amortized Complexity vs Standard Complexity

Amortized complexity should not be directly compared with the standard worst-case complexity of an operation.

For example,

```
append()
```

has

- Worst-case complexity → **O(n)**
- Amortized complexity → **O(1)**

The amortized complexity does **not** mean that every append operation takes constant time.

Instead, it means that while a few operations are expensive (because of allocation and copying), they occur infrequently enough that the **average cost over a long sequence of operations remains constant**.

Therefore,

- A standard **O(1)** operation performs approximately the same amount of work every time.
- An amortized **O(1)** operation usually performs constant work but occasionally performs **O(n)** work.

Although both are asymptotically **O(1)**, the amortized version may experience occasional spikes in execution time due to these expensive operations.

---

### Engineering Perspective

Amortized complexity guarantees excellent throughput but not consistent latency.

For example, appending one million elements to a slice is very efficient overall because most append operations are inexpensive.

However, an individual append may occasionally trigger:

- Allocation of a larger backing array
- Copying of all existing elements
- Insertion of the new element

This causes a temporary spike in execution time.

Therefore, while the **average** cost of `append()` is **O(1)**, a single append operation may still take **O(n)** time.

This distinction is important in systems where predictable latency is more important than average throughput, such as real-time systems or latency-sensitive backend services.

#### Avoiding Resize Overhead

If the approximate number of elements is known in advance, the expensive resize operations can often be eliminated by **pre-allocating** enough capacity.

In Go, this is done using `make()`:

```go
slice := make([]int, 0, 1_000_000)
```

Now, appending up to one million elements will not require repeated reallocations or copying of existing elements.

Benefits of pre-allocation include:

- Eliminates most resize operations
- Avoids repeated copying of existing elements
- Reduces memory allocations
- Improves cache locality
- Provides more predictable latency
- Improves overall performance

For this reason, pre-allocation is a common optimization in performance-critical Go applications whenever the required capacity is known or can be estimated.
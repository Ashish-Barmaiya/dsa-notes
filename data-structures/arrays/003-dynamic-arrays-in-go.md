# Dynamic Array in Go (Slice)

- Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data.

- Slices hold references to an underlying array.

- A slice literal is declared just like an array literal, except you leave out the element count:

```go
letters := []string{"a", "b", "c", "d"}
```

- A slice can be created with the built-in function called make, which has the signature:

```go
func make([]T, len, cap) []T
// T stands for the element type of the slice to be created
```
$\rightarrow$ more on this later.

---

# Slice Internals

- A slice is a descriptor of an array segment.

- It consists of a pointer to the array, the length of the segment, and its capacity (the maximum length of the segment).

## Slice Header

- A slice variable is actually a slice header.

- A slice variable does not store array data directly. Instead, it is a Slice Header that lives on the stack.

In Go’s runtime library (specifically reflect.SliceHeader), it is structured structurally like this:

```go
type SliceHeader struct {
    Data uintptr // The raw memory address pointing to the first element of the underlying array
    Len  int     // Length: The number of elements currently visible in the slice view
    Cap  int     // Capacity: The maximum number of elements before the underlying array must resize
}
```
- When we create a slice, Go creates an array (a static array) and a slice header. Slice header contains a pointer that points to the base address of the underlying array. It also contains information about length and capacity of the underlying array.

## The "Dumb Array" Principle

- **The Array is Passive:** An underlying array in memory consists of raw, unstructured bytes. It possesses no metadata. It does not know its own length, its maximum capacity, or how many slots are "filled."

- **The Header is the Source of Truth:** The Len and Cap fields are simply integer variables stored inside the slice header on the stack. Go performs basic mathematical arithmetic to update these fields during actions like append. It never queries the underlying array to calculate length.

## Passing slices to functions

- Go is strictly a **pass-by-value** language.

- When a slice is passed to a function, Go creates a local copy of the slice header on the stack.

- Because the header copy contains the exact same Data memory pointer, modifying elements inside the function modifies the shared underlying array.

- However, because the header itself is a copy, modifications to the header fields (Len and Cap) are strictly local.

---

# Common Fallacies and Edge Cases

## Scenario A: The Length Disconnect (Mutating elements via a copy)

**Premise:** We pass a slice with Len: 5, Cap: 10 to a function. The function appends 2 elements.

- Inside the function the local header (slice_header_2) increments its Len field to 7. Two new elements are written to indices 5 and 6 of the underlying array.

- Inside main (caller) the original header (slice_header_1) maintains its Len field at 5.

- This behavior is intentional. Slices are isolated views. If slice_header_1 mutated its length automatically based on external actions, it would break data isolation and introduce race conditions in concurrent/multi-threaded code. Main's "window" remains constrained to 5 elements.

## Scenario B: Reallocation Triggers (append when Len == Cap)

**Premise:** A function appends elements to a copied slice header until Len == Cap, and then appends one more element, triggering an underlying memory reallocation.

### Sub-Case 1: The modified slice IS returned to main

1. When capacity is exceeded, Go allocates a new, doubled array in the heap.

2. Go copies all elements from the old array to the new array, adds the new element, and updates the local slice_header_2's Data pointer to point to the new array address.

3. When returned and assigned back to slice_header_1, slice_header_1 now points to newly created doubled array and the old array has zero references pointing to it. It becomes unreachable, and the Go Garbage Collector (GC) reclaims it during the next cycle.

```go
slice_header_1 = modifySlice(slice_header_1)
```

### Sub-Case 2: The modified slice IS NOT returned to main

1. The function triggers reallocation, moving slice_header_2 to a brand-new array address.

2. The function ends, and slice_header_2 is popped off the stack.

3. The new array now has zero references pointing to it and is instantly marked for garbage collection.

4. slice_header_1 in main remains completely unchanged, pointing safely to the original array.

## Scenario C: Silent Data Corruption

**Premise:** A function secretly appends elements to an underlying array using a copied header, filling it up to capacity (e.g., index 5 through 9 are filled). The function ends without returning the slice. Later, main decides to append an element to slice_header_1.

- **The Problem:** slice_header_1 checks its own header (Len: 5, Cap: 10). It sees plenty of capacity left. It assumes appending will be a fast, $O(1)$ operation in place.

- **The Reality:** Go calculates the write destination using Data + Len (index 5). Because slice_header_1 is unaware that the function previously wrote data there, it blindly overwrites index 5.

- **The Takeaway:** Go does not trigger a protective $O(n)$ copy here because the local header claims there is space. We don't experience a performance hit, but we suffer silent data corruption as the function's changes are obliterated.

#### In Short:  If a local copy of a slice header is created and modifications are performed in the local copy and the original slice header is not updated, there will be silent data corruption $\rightarrow$ Two slice headers pointing to same array but having different metadata.

## Rules of Idiomatic Go Slices

### 1. Always Capture the Return Value

- If a function performs an append or modifies a slice's boundaries, it must return the updated header, and the caller must capture it:

```go
s = append(s, item)
s = processData(s)
```

### 2. Use Three-Index Slicing for Protection:

- If we must pass a slice to a function but want to absolute guarantee it cannot overwrite extra capacity slots, clip its capacity using a three-index slice (slice[low:high:max]):

```go 
// Sets Len to 5 and Cap to 5. 
// If the function tries to append, it triggers a safe reallocation instantly, 
// ensuring main's underlying array remains untouched.
modifySlice(slice_header_1[0:5:5])
```

---

# Slicing

- A slice can also be formed by **slicing** an existing slice or array.

- Slicing is done by specifying a half-open range with two indices separated by a colon:

```go
newSlice := original[low : high] // low is the start index (inclusive), high is end index (exclusive)
```

- For Example:

```go
slice := []int {0, 1, 2, 3, 4, 5}

newSlice := slice[1:4] // 

// newSlice = [1, 2, 3]
```

- The start and end indices of a slice expression are optional; they default to zero and the slice’s length respectively:

```go
newSlice := slice[:2] // slicing starts from index 0 to index 1 (end index - 1)

// newSlice = [1, 2]
```

```go
newSlice := slice[2:] // slicing starts from index 2 to the nth index

// newSlice = [2, 3, 4, 5]
```

```go
newSlice := slice[:] // slicing starts from index 0 to the nth index

// newSlice = [0, 1, 2, 3, 4, 5]

// basically, slice[:] = slice
```

## The Mechanics of Re-Slicing

### What Actually Happens During Re-Slicing?

When we re-slice an existing slice using the syntax subSlice := mainSlice[low:high], Go does not allocate a new array, nor does it copy any data.

Instead, Go constructs a new Slice Header on the stack. The fields of this new header are mathematically calculated based on the original header:

- Data (New Pointer): Shifted forward by the low index.

- Len (Length): Calculated as high - low.

- Cap (Capacity): Automatically calculated as OldCapacity - low. It inherits the remaining capacity of the original underlying array.

### Edge Cases of Re-Slicing

#### Edge Case A: The "Borrowing Capacity" Overwrite Risk

When we do not explicitly define a capacity during a re-slice, the new slice borrows the full remaining capacity of the parent array. This creates a risk of silent data destruction.

```go
func main() {
    original := []int{10, 20, 30, 40, 50} // Len: 5, Cap: 5
    
    // Create a sub-slice of the first two elements
    sub := original[0:2] // Len: 2, Cap: 5 (Borrows remaining capacity!)
    
    // We think 'sub' is isolated, so we append to it
    sub = append(sub, 999)
    
    fmt.Println("Sub:", sub)            // [10, 20, 999]
    fmt.Println("Original:", original)  // [10, 20, 999, 40, 50] <-- SILENT CORRUPTION
}
```

This happens because sub had a borrowed capacity of 5, append realized it had room to grow in place. It mathematically calculated Data + Len (index 2) and blindly overwrote the value 30 in the original array with 999.

#### Edge Case B: The Hidden Memory Leak (Holding the Entire Array)

Because a re-slice points directly to the original underlying array, that entire underlying array cannot be garbage collected as long as the sub-slice is alive.

Imagine we read a large log file or database record into memory, extract a tiny piece of information, and keep only that piece:

```go
func getSmallToken() []byte {
    // Allocates a large 50MB array in the heap
    largeLog := readLargeFile() 
    
    // Re-slice a tiny 4-byte token from the front
    token := largeLog[0:4] 
    
    return token // largeLog goes out of scope, BUT...
}
```

We think we freed 50MB of memory and kept only 4 bytes. In reality, because token's Data pointer still hooks into the base of that large heap array, the Go Garbage Collector cannot free a single byte of that 50MB block. If we store this token in a long-lived global map, we have introduced a silent memory leak.

### How to Handle and Prevent These Issues

#### 1. Solution A: Three-Index Slicing (Fixes Overwrite Risks)

To stop a sub-slice from borrowing the parent’s capacity, use the three-index syntax: slice[low:high:max]. The capacity of the new slice becomes max - low.

```go
original := []int{10, 20, 30, 40, 50}

// Force Capacity to match Length exactly: max = 2
sub := original[0:2:2] // Len: 2, Cap: 2 (Cannot borrow extra space!)

sub = append(sub, 999) // Cap is full! Forces a safe O(n) reallocation.

fmt.Println("Original:", original) // [10, 20, 30, 40, 50] <-- SAFE!
```

#### 2. Solution B: Built-in copy() (Fixes Memory Leaks)

To completely detach a sub-slice from a large parent array, we must allocate a brand-new, isolated slice and use the built-in copy() function to move the data.

```go
func getSmallToken() []byte {
    largeLog := readLargeFile() // 50MB
    
    // 1. Allocate a completely independent 4-byte slice
    token := make([]byte, 4)
    
    // 2. Copy the data from the large array into the small one
    copy(token, largeLog[0:4])
    
    return token 
    // Now, largeLog has zero references. 
    // The GC will successfully wipe the 50MB array from memory!
}
```

## Idiomatic Rules of Re-Slicing

1. **Slices are Shared State by Default:** Always assume that any re-slice (a[x:y]) shares the same memory as the parent. If we mutate elements in one, we mutate the other.

2. **The Three-Index Mandate:** Never pass a re-sliced view into an untrusted function or concurrent goroutine without using three-index slicing (a[low:high:high]) to limit its capacity. If we don't limit capacity, that function can corrupt our local data via append.

3. **The Lifespan Rule:** If a tiny sub-slice needs to live longer than the giant slice it came from, do not re-slice. We must explicitly make() a new slice and copy() the data to free the parent array for the Garbage Collector.

---

# The make () Function

- make is a specialized built-in function that is evaluated at compile-time.

- It is used exclusively for allocating and initializing Go's built-in reference types: Slices, Maps, and Channels.

- Unlike new (which only allocates zeroed memory and returns a pointer), make allocates memory in the heap and initializes the internal descriptor structures (like the slice header fields) so the data structure is immediately ready for use.


## The make Function for Slices

### Syntax Variations and Signatures

Conceptually, make for slices behaves as if it has two distinct forms, depending on how many arguments we supply:

#### Form A: Two-Argument Syntax (Default Capacity)

```go
slice := make([]T, length)
```

- What it does: Allocates an underlying array of size length.

- The Resulting Header: The Len and Cap fields in the slice header are set to the exact same value.

- Zero-Initialization: Every single element in the allocated array is initialized to its type's zero value (e.g., 0 for integers, "" for strings, false for booleans).

#### Form B: Three-Argument Syntax (Explicit Capacity)

```go
slice := make([]T, length, capacity)
```

- What it does: Allocates a larger underlying array of size capacity, but restricts our initial visibility window to length.

- The Resulting Header: Len is set to length, and Cap is set to capacity.

- Zero-Initialization: Elements from index 0 to length-1 are zero-initialized and readable. Elements from index length to capacity-1 exist in memory but are hidden behind the bounds checker.

### Compile-Time Restrictions and Errors

Go enforces strict safety rules at compile-time and runtime when using make:

1. **The Cardinal Rule:** length can never be greater than capacity.

```go
// Compile error: len larger than cap in make([]int)
s := make([]int, 10, 5)
```

2. **Negative Bound Protection:** We cannot pass a negative integer variable or constant as length or capacity. Doing so triggers a compile error or a runtime panic.

3. **Type Constraint:** The first argument must be a type literal (e.g., []int, []string). We cannot pass a variable representing a type.

### The Performance Impact: Pre-allocating Capacity

A common mistake in Go is using the two-argument make syntax when you already know how many items your slice will eventually hold.

#### The Bad Way ($O(n)$ Allocations)

```go
// Creates a slice with Len: 0, Cap: 0
s := make([]int, 0) 

for i := 0; i < 10000; i++ {
    s = append(s, i) // Triggers multiple heap allocations and array copies
}
```

#### The Idiomatic Way ($O(1)$ Fixed Allocation)

```go
// Pre-allocates a 10,000 element array once in the heap
s := make([]int, 0, 10000) 

for i := 0; i < 10000; i++ {
    s = append(s, i) // Pure O(1) operations. ZERO re-allocations occur.
}
```

## The `make([]T, len)` vs `make([]T, 0, cap)` Gotcha

This is a logic trap that frequently causes bugs when parsing JSON or API payloads.

### The Trap: Accidental Zero-Padding

```go
// We want a slice for 5 elements
s := make([]int, 5) 

// We append data to it
for i := 1; i <= 3; i++ {
    s = append(s, i)
}

fmt.Println(s) // Output: [0 0 0 0 0 1 2 3] <-- TRAP!
```

**Why this happened:** make([]int, 5) instantly populates the first 5 elements with 0. When we called append, Go looked at Len: 5 and appended the new numbers at index 5, 6, and 7, leaving the zero-padded elements untouched.

### The Fix:

- If we intend to use append, initialize with a length of zero: `make([]int, 0, 5)`.

- If we intend to assign elements via direct indexing (s[i] = value), initialize with an active length: `make([]int, 5)`.

---
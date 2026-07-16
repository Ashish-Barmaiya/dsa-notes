# Static Array in Go

- An array is a fundamental and linear data structure that stores items at contiguous locations.

- In Go, an array type definition specifies a length and an element type. For example, the type [4]int represents an array of four integers.

- An array’s size is fixed; its **length is part of its type** because an arrays size is fixed at compile-time. ([4]int and [5]int are distinct, incompatible types).

```go
var a [4]int

// This creates an array 'a' with size 4, type int
// Array is zero-filled
```
- **Fixed Size:** Just like in C/C++, the size of a Go array is fixed at compile-time and forms an inseparable part of its type system (e.g., [5]int and [10]int are distinct, incompatible types).

- **No Variable Sizing:** You cannot use a runtime variable to declare an array's size. It must be a compile-time constant.

- **Pass-by-Value:** Go’s arrays are values. An array variable denotes the entire array; it is not a pointer to the first array element (as would be the case in C). This means that when you assign or pass around an array value you will make a copy of its contents (pass by value). Changes inside the function do not affect the original array.

- To avoid the copy you could pass a pointer to the array, but then that’s a pointer to an array, not an array (pass by reference).

- An array litereal can be specified like this:

```go
b := [2]string{"Penn", "Teller"}
```

- Or we can have the compiler count the array elements:

```go
b := [...]string{"Penn", "Teller"}
```

- In both cases, the type of b is [2]string.

## Where will it be created?

- In C, if we declare an array inside a function, the memory for this array will be created inside the stack.

- However, in Go we don't get to explicitly choose if a variable goes on the stack or the heap.

- **Escape Anaylysis:** The Go compiler performs escape analysis at compile-time. If the compiler can prove the array does not "escape" the function (e.g., you don't return a pointer to it or pass it to a goroutine), it allocates it on the stack. If it escapes, the compiler silently moves it to the heap.

---

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
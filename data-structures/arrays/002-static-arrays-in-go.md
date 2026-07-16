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

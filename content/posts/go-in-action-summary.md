---
author: "Juan Carlos Martinez de la Torre"
date: 2019-01-14
linktitle: go-in-action-summary
menu:
  main:
    parent: tutorials
next: nothing
prev: nothing
title: Go in Action Summary
description: Summary about Go in Action book, for later personal consulting on Go development.
weight: 10
---


## Notes Chapter 2

* If your main function doesn’t exist in package main, the build tools won’t produce an executable.

* All code files in a folder must use the same package name, and it’s common practice to name the package after the folder

* All init functions in any code file that are part of the program will get called before
the main function

*  An exported
identifier can be directly accessed by code in other packages when the respective
package is imported. These identifiers start with a capital letter. 

* In Go, all variables are initialized to their zero value. For numeric types, that value
is 0; for strings it’s an empty string; for Booleans it’s false; and for pointers, the zero
value is nil.

*  A slice is a reference type that implements a dynamic array. 

* The Fatal function accepts an error value and will log to the terminal window before
terminating the program. From the **log** package

```Go
log.Fatal(err)
```

* The short variable declaration operator (:=). This operator is
used to both declare and initialize variables at the same time. The type of each value
being returned is used by the compiler to determine the type for each variable.

```
feeds, err := RetrieveFeeds()
```

* Channels are also a reference type in Go like maps and slices, but channels implement a queue of typed values that are used to communicate data between goroutines.

* In Go, once the main function returns, the program terminates.

* To iterate over the slice of feeds, we use
the keywords for range. The keyword range can be used with arrays, strings, slices,
maps, and channels. When we use for range to iterate over a slice, we get two values
back on each iteratio

* Closures ¿? on goroutines: Let us pass variables from outer scope.

* The physical location of the package from within the standard library
doesn’t change this fact. As we access functionality from the json package, we’ll use
just the name json.


# Go in Action

## Chapter 2. Summary


Packages are found on disk based on their relative path to the directories referenced
by the Go environment.
Packages are found on disk based on their relative path to the directories referenced
by the Go environment.


Set GO path to various paths
```Bash
GOPATH = "/home/myproject:/home/mylibraries"
```

The important thing to remember is that the Go installation directory is the
first place the compiler looks and then each directory listed in your GOPATH in the
order that they’re listed.

#### Named imports

```go
import (
 "fmt"
 myfmt "mylib/fmt"
 )
 ```

matter in a sea of compiler warnings.
Sometimes you may need to import a package that you don’t need to reference
identifiers from. When this is
the case, you can use the blank identifier _ to rename an import.

#### Init Functions:
When a program imports this package, the init function will be called.

As we can’t import a package that you aren’t using, so renaming the
import with the blank identifier allows the init function to be discovered and scheduled to run without the compiler issuing an error about unused imports

How to run a program 

* Compiling to an executable
```
go build wordcount.go
go build .
```
* Directly compiling and executing
go run wordcount.go


### Further tools
```Go
go vet //Check code for common errors 
go fmt {package} // Format package or file
```

### Creating Repositories for sharing

When you’re using go get, you specify the full path to the package that should be
imported. This means that when you create a repository that you intend to share, the
package name should be the repository name, and the package’s source should be in
the root of the repository’s directory structure.


## Chapter 4

### Array

Once an array is declared, neither the type of data being stored nor its length can be
changed. When an array is initialized, each individual element that belongs to the array is initialized to its zero value.

Because an array is a value, you can copy arrays with assignation.
```Go
var array [5]int
array := [5]int{10, 20, 30, 40, 50}
// Capacity determined by number of elements
array := [...]int{10, 20, 30, 40, 50} 
// Initialize index 1 and 2 with specific values.
// Rest element contain zero values
array := [5]int{1: 10, 2: 20} 
```

##### Array of Pointers

```Go
// Declare an integer pointer array of five elements.
// Initialize index 0 and 1 of the array with integer pointers.
array := [5]*int{0: new(int), 1: new(int)}
// Assign values to index 0 and 1.
*array[0] = 10
*array[1] = 20
```

Copying an array of pointers copies the pointer values and not the values that the
pointers are pointing

#### Multidimensional Array
```Go
// Declare a two dimensional integer array 
var array [4][2]int
// Literal to declare and init
array := [4][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}}
// Declare and initialize index 1 and 3 of the outer array.
array := [4][2]int{1: {20, 21}, 3: {40, 41}}
// Declare and initialize individual elements of the outer
// and inner array.
array := [4][2]int{1: {0: 20}, 3: {1: 41}}

// Declare a two dimensional integer array of two elements.
var array [2][2]int
// Set integer values to each individual element.
array[0][0] = 10
array[0][1] = 20
array[1][0] = 30
array[1][1] = 40
```

#### Passing Array between functions

When you pass variables between functions, they’re always
passed by value. When your variable is an array, this means the entire array, regardless
of its size, is copied and passed to the function.

#### Slices

Slices are built around the concept of dynamic arrays that can grow and
shrink as you see fit. 

Slices are tiny objects that abstract and manipulate an underlying array. As an object it has three fields:
 1. a pointer to the underlying array, 
 2. the length or the number of elements the slice has access
 3. the capacity or the number of elements the slice has available for growth

Slices give you all the benefits of indexing, iteration, and garbage collection optimizations
because the underlying memory is allocated in contiguous blocks.

##### Make and Slices 
``` Go
slice := make([]string, 5) // Contains a length and capacity of 5 elements.
slice := make([]int, 3, 5) // Contains a length of 3 and has a capacity of 5.
```
When specifying ```capacity > length``` =>  you dont have to the underlying array access at first.

```Go
// Contains a length and capacity of 5 elements.
slice := []string{"Red", "Blue", "Green", "Yellow", "Pink"}
// Contains a length and capacity of 3 elements.
slice := []int{10, 20, 30}
```

Declaration differences between arrays and slices. 
```Go
// Create an array of three integers.
array := [3]int{10, 20, 30}
// Create a slice of integers with a length and capacity of three.
slice := []int{10, 20, 30}
```

##### Nil slice
Used on exceptions or empty query results.
```Go
var slice []int
slice := make([]int, 0)
slice := []int{}
```

#### Taking Slice of a slice

You need to remember that you now have two slices sharing the same underlying
array. Changes made to the shared section of the underlying array by one slice can be
seen by the other slice.

```Go
// Create a slice of integers.
// Contains a length and capacity of 5 elements.
slice := []int{10, 20, 30, 40, 50}
// Create a new slice.
// Contains a length of 2 and capacity of 4 elements.
newSlice := slice[1:3]
// Change index 1 of newSlice.
// Change index 2 of the original slice.
newSlice[1] = 35
```

#### Growing Slices

1. **append** When there is more capacity to allocate the new value. No new array is needed.

``` Go
slice := []int{10, 20, 30, 40, 50} // length and capacity of 5 elements.
newSlice := slice[1:3] // length of 2 and capacity of 4 elements.
newSlice = append(newSlice, 60) // Assign the value of 60 to the new element.
```

Since the original slice is sharing the underlying array, slice also sees the changes in index 3.
```Go
// slice = [10, 20, 30, 60, 50]
// newSlice = [10, 20, 60]
``` 

2. **append**: When there’s no available capacity in the underlying array for a slice, the append function will create a new underlying array.


```Go
slice := []int{10, 20, 30, 40} // Length and Capacity of 4 elements.
newSlice := append(slice, 50) // No more capacity => need to createa new array
```

When there’s no available capacity in the underlying array for a slice, the append function will create a new underlying array, copy the existing values that are being referenced, and assign the new value.There wont be sharding anymore between slices. The capacity of the array is doubled from its original size 

It is possible to append several elements at one call:

```Go
s1 := []int{1, 2}
s2 := []int{3, 4}
append(s1, s2...) -> // [1, 2, 3, 4]
```
#### Three index slices

This third index gives you control over the capacity of the new slice. The purpose is not to increase capacity, but to restrict the capacity
```Go
// Create a slice of strings.
// Contains a length and capacity of 5 elements.
source := []string{"Apple", "Orange", "Plum", "Banana", "Grape"}

// Slice the third element and restrict the capacity.
// Contains a length of 1 element and capacity of 2 elements.
slice := source[2:3:4]
```
With the third index we indicate up to what element the new slice has access

By having the option to set the capacity of a ```new_slice``` to be the same as the length,
you can force the first append operation to detach the ```new_slice``` from the underlying
array. Detaching the new slice from its original source array makes it safe to change, avoiding overwriting the original array.


#### Iterating over slices 

Since a slice is a collection, you can iterate over the elements. 

The keyword range, when iterating over a slice, will return two values. The first value
is the index position and the second value is a copy of the value in that index position.

It’s important to know that range is making a copy of the value, not returning a reference. 

```Go
// Contains a length and capacity of 4 elements.
slice := []int{10, 20, 30, 40}
// Iterate over each element and display the value and addresses.
for index, value := range slice {
  fmt.Printf("Value: %d Value-Addr: %X ElemAddr: %X\n",
    value, &value, &slice[index])
}
Output:
Value: 10 Value-Addr: 10500168 ElemAddr: 1052E100
Value: 20 Value-Addr: 10500168 ElemAddr: 1052E104
Value: 30 Value-Addr: 10500168 ElemAddr: 1052E108
Value: 40 Value-Addr: 10500168 ElemAddr: 1052E10C
//The address for the value variable is always the same because it’s a variable that contains a copy
```
#### Multidimensional Slices
Slices are one-dimensional, but they can be composed to create multidimensional slices 
``` Go
// Create a slice of a slice of integers.
slice := [][]int{{10}, {100, 200}}
```
The outer slice contains two elements, each of which are slices. The slice in the first
element is initialized with the single integer 10 and the slice in the second element
contains two integers, 100 and 200.

#### Passing slices between functions

Passing a slice between two functions requires nothing more than passing the slice by
value.
Only the slice is being copied, not the underlying array. (That is the array-address, the length and capacity).

On a 64-bit architecture, a slice requires 24 bytes of memory. The pointer field
requires 8 bytes, and the length and capacity fields require 8 bytes each one.

### Map 
The strength of a map is its ability to retrieve data quickly based on the key. A key works like an index, pointing to the value you associate with that key. Map is implemented using a hash table.


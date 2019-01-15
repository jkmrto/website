---
author: "Juan Carlos Martinez de la Torre"
date: 2019-01-014
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

Slices are tiny objects that abstract and manipulate an underlying array.

Slices give you all the benefits of indexing, iteration, and garbage collection optimizations
because the underlying memory is allocated in contiguous blocks.
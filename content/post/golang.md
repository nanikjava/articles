---
date: "2019-09-01"
title: "Golang"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---
Following are some of the thing that need to be remembered when programming in Golang. This information is taken from variety of sources.

--------
Variables
---------

*  For variable naming, omit words that are obvious given a variable's type declaration or otherwise clear from the surrounding context. Following is not a good example
{{< highlight go >}}
returnSlice := make([]string, len(src))
{{< /highlight >}}
* Variable names in Go should be short rather than long. This is especially true for local variables with limited scope. Prefer c to lineCount. Prefer i to sliceIndex.
* The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (i, r). More unusual things and global variables need more descriptive names.
* For example, "URL" should appear as "URL" or "url" (as in "urlPony", or "URLPony"), never as "Url". ServeHTTP not ServeHttp. 
* For identifiers with multiple initialized "words", use for example "xmlHTTPRequest" or "XMLHTTPRequest".
* Have a reasonable number of parameters and reasonably short variable names.
* Having one doubled up field in the struct makes the test data difficult to scan. Following is not a good example : 
{{< highlight go >}}
...
prefix, postfix string
{{< /highlight >}}
* Declare empty slices as
{{< highlight go >}}
var t []string (declares a nil slice value) 
{{< /highlight >}}
and not
{{< highlight go >}}
t := []string{} (is non-nil but zero-length)            
{{< /highlight >}}

* Interfaces & Receivers
    * Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.
    * Design the API so that it can be tested using the public API of the real implementation.
    * Do not define interfaces before they are used: without a realistic example of usage, it is too difficult to see whether an interface is even necessary, let alone what methods it ought to contain.
    * Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. 
{{< highlight go >}}
    package consumer  // consumer.go

    type Thinger interface { Thing() bool }

    func Foo(t Thinger) string { … }
{{< /highlight >}}
{{< highlight go >}}
    package consumer // consumer_test.go

    type fakeThinger struct{ … }
    func (t fakeThinger) Thing() bool { … }
    …
    if Foo(fakeThinger{…}) == "x" { … }
{{< /highlight >}}
    Instead of doing this
{{< highlight go >}}
    // DO NOT DO IT!!!
    package producer

    type Thinger interface { Thing() bool }

    type defaultThinger struct{ … }
    func (t defaultThinger) Thing() bool { … }

    func NewThinger() Thinger { return defaultThinger{ … } }
{{< /highlight >}}

        ...better do this
{{< highlight go >}}  
    package producer

    type Thinger struct{ … }
    func (t Thinger) Thing() bool { … }

    func NewThinger() Thinger { return Thinger{ … } }
{{< /highlight >}}

    * In Go, the receiver of a method is just another parameter and therefore, should be named accordingly. The name need not be as descriptive as that of a method argument, as its role is obvious and serves no documentary purpose. It can be very short as it will appear on almost every line of every method of the type; familiarity admits brevity. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.
    * If in doubt, use a pointer, but there are times when a value receiver makes sense, usually for reasons of efficiency, such as for small unchanging structs or values of basic type. Some useful guidelines:
        * _**NO POINTER**_ for the following:
            - If the receiver is a map, func or chan, don't use a pointer to them. If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
            - If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.            
        * _**USE POINTER**_ for the following:
            * If the method needs to mutate the receiver, the receiver must be a pointer.
            * If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying.
            * If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
            * Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
            * If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
            * Finally, when in doubt, use a pointer receiver.

-------
Testing
-------

* Tests should fail with helpful messages saying what was wrong, with what inputs, what was actually got, and what was expected.
{{< highlight go >}}  
if got != tt.want {
    t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
}
{{< /highlight >}}

--------
Comments
--------

* For comments use // instead of /* */
* Start comment with the name of the function 
* All top-level, exported names should have doc comments, as should non-trivial unexported type or function declarations.
* For readability, move the comment to appear on it's own line before the variable. Following is NOT a good example
{{< highlight go >}} 
...
OutPrefix   = "> " // OutPrefix notes output
{{< /highlight >}}
* Package comments, like all comments to be presented by godoc, must appear adjacent to the package clause, with no blank line.
{{< highlight go >}} 
// Package math provides basic constants and mathematical functions.
package math
{{< /highlight >}}
{{< highlight go >}} 
/*
Package template implements data-driven templates for generating textual
output such as HTML.
....
*/
package template
{{< /highlight >}}

* For "package main" comments, other styles of comment are fine after the binary name. For example, for a package main in the directory seedgen you could write:
{{< highlight go >}} 
// Binary seedgen ...
package main

// Command seedgen ...
package main

// Program seedgen ...
package main

// The seedgen command ...
package main

// The seedgen program ...
package main

// Seedgen ..
package main
{{< /highlight >}}
    
---------
Functions
---------
* If a function returns two or three parameters of the same type, or if the meaning of a result isn't clear from context, adding names may be useful in some contexts. 
* Don't name result parameters just to avoid declaring a var inside the function; that trades off a minor implementation brevity at the cost of unnecessary API verbosity.
* Naked returns are okay if the function is a handful of lines. Once it's a medium sized function, be explicit with your return values. Corollary: it's not worth it to name result parameters just because it enables you to use naked returns. Clarity of docs is always more important than saving a line or two in your function.
* Don't pass pointers as function arguments just to save a few bytes, however This advice does not apply to large structs, or even small structs that might grow.
    
       
--------------         
Error Handling
--------------

* Don't use panic for normal error handling. Use error and multiple return values.
* Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation
* Do not discard errors using _ variables. Handle the error, return it, or, in truly exceptional situations, panic.
* Do not use return value such as in C using -1 to flag error. Function can return multiple result so use one of the returned result to flag if there is an error. Eg:
{{< highlight go >}} 
value, ok := Lookup(key)
if !ok  {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
{{< /highlight >}}

* This is recommended
{{< highlight go >}} 
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
{{< /highlight >}}
     instead of
{{< highlight go >}} 
if err != nil {
    // error handling
} else {
    // normal code
}
{{< /highlight >}}

--------
Packages
--------

* Avoid renaming imports except to avoid a name collision; good package names should not require renaming. In the event of collision, prefer to rename the most local or project-specific import.
* Packages that are imported only for their side effects (using the syntax import _ "pkg") should only be imported in the main package of a program, or in tests that require them.
* In this case, the test file cannot be in package foo because it uses bar/testutil, which imports foo. So we use the 'import .' form to let the file pretend to be part of package foo even though it is not. Except for this one case, do not use import . in your programs. It makes the programs much harder to read because it is unclear whether a name like Quux is a top-level identifier in the current package or in an imported package.
* Avoid meaningless package names like util, common, misc, api, types, and interfaces

<h1>References</h1>

* Golang
	* http://peter.bourgon.org/go-in-production/#testing-and-validation --> best practises
    * Lesson learned from contributing minikube
        * https://github.com/kubernetes/minikube/pull/5718#discussion_r339308904
            - "Generally isn't useful to repeat the type of an object in the object name."  See also https://github.com/golang/go/wiki/CodeReviewComments#variable-names

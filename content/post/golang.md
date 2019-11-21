---
date: "2019-09-01"
title: "Golang"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---
Following are highlights some of the thing that need to be remembered when programming in Golang. This is taken from https://github.com/golang/go/wiki/CodeReviewComments

* Variables
    * For variable naming, omit words that are obvious given a variable's type declaration or otherwise clear from the surrounding context.

        not good example: 
            returnSlice := make([]string, len(src))
            
    * Variable names in Go should be short rather than long. This is especially true for local variables with limited scope. Prefer c to lineCount. Prefer i to sliceIndex.
    * The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (i, r). More unusual things and global variables need more descriptive names.
    * For example, "URL" should appear as "URL" or "url" (as in "urlPony", or "URLPony"), never as "Url". ServeHTTP not ServeHttp. 
    * For identifiers with multiple initialized "words", use for example "xmlHTTPRequest" or "XMLHTTPRequest".
    * Have a reasonable number of parameters and reasonably short variable names.
    * Having one doubled up field in the struct makes the test data difficult to scan.

        not good example : 
            ...
            prefix, postfix string
            
    * Declare empty slices as 

        var t []string (declares a nil slice value) 
     
      and not

        t := []string{} (is non-nil but zero-length)            

* Interfaces & Receivers
    * Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.
    * Design the API so that it can be tested using the public API of the real implementation.
    * Do not define interfaces before they are used: without a realistic example of usage, it is too difficult to see whether an interface is even necessary, let alone what methods it ought to contain.
    * Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. 

            package consumer  // consumer.go

            type Thinger interface { Thing() bool }

            func Foo(t Thinger) string { … }



            package consumer // consumer_test.go

            type fakeThinger struct{ … }
            func (t fakeThinger) Thing() bool { … }
            …
            if Foo(fakeThinger{…}) == "x" { … }

        Instead of doing this

            // DO NOT DO IT!!!
            package producer

            type Thinger interface { Thing() bool }

            type defaultThinger struct{ … }
            func (t defaultThinger) Thing() bool { … }

            func NewThinger() Thinger { return defaultThinger{ … } }
            
        ...better do this
        
            package producer

            type Thinger struct{ … }
            func (t Thinger) Thing() bool { … }

            func NewThinger() Thinger { return Thinger{ … } }
            
    * In Go, the receiver of a method is just another parameter and therefore, should be named accordingly. The name need not be as descriptive as that of a method argument, as its role is obvious and serves no documentary purpose. It can be very short as it will appear on almost every line of every method of the type; familiarity admits brevity. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.
    * If in doubt, use a pointer, but there are times when a value receiver makes sense, usually for reasons of efficiency, such as for small unchanging structs or values of basic type. Some useful guidelines:
        * NO POINTER for the following:
            - If the receiver is a map, func or chan, don't use a pointer to them. If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
            - If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.            
        * USE POINTER for the following:
            * If the method needs to mutate the receiver, the receiver must be a pointer.
            * If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying.
            * If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
            * Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
            * If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
            * Finally, when in doubt, use a pointer receiver.

* Testing
    * Tests should fail with helpful messages saying what was wrong, with what inputs, what was actually got, and what was expected.

        if got != tt.want {
            t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
        }



* Comments
    * For comments use // instead of /* */
    * Start comment with the name of the function 
    * All top-level, exported names should have doc comments, as should non-trivial unexported type or function declarations.
    * For readability, move the comment to appear on it's own line before the variable.	

        not good example : 
            ...
            OutPrefix   = "> " // OutPrefix notes output
    * Package comments, like all comments to be presented by godoc, must appear adjacent to the package clause, with no blank line.
    
        // Package math provides basic constants and mathematical functions.
        package math
        
        
        /*
        Package template implements data-driven templates for generating textual
        output such as HTML.
        ....
        */
        package template

    * For "package main" comments, other styles of comment are fine after the binary name. For example, for a package main in the directory seedgen you could write:

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




            
* Functions
    * If a function returns two or three parameters of the same type, or if the meaning of a result isn't clear from context, adding names may be useful in some contexts. 
    * Don't name result parameters just to avoid declaring a var inside the function; that trades off a minor implementation brevity at the cost of unnecessary API verbosity.
    * Naked returns are okay if the function is a handful of lines. Once it's a medium sized function, be explicit with your return values. Corollary: it's not worth it to name result parameters just because it enables you to use naked returns. Clarity of docs is always more important than saving a line or two in your function.
    * Don't pass pointers as function arguments just to save a few bytes, however This advice does not apply to large structs, or even small structs that might grow.
    





        
* Error Handling
    * Don't use panic for normal error handling. Use error and multiple return values.
    * Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation
    * Do not discard errors using _ variables. Handle the error, return it, or, in truly exceptional situations, panic.
    * Do not use return value such as in C using -1 to flag error. Function can return multiple result so use one of the returned result to flag if there is an error. Eg:
    
        value, ok := Lookup(key)
        if !ok  {
            return fmt.Errorf("no value for %q", key)
        }
        return Parse(value)

    * This is recommended
    
        if err != nil {
            // error handling
            return // or continue, etc.
        }
        // normal code
    
     instead of
     
        if err != nil {
            // error handling
        } else {
            // normal code
        }


   
* Packages
    * Avoid renaming imports except to avoid a name collision; good package names should not require renaming. In the event of collision, prefer to rename the most local or project-specific import.
    * Packages that are imported only for their side effects (using the syntax import _ "pkg") should only be imported in the main package of a program, or in tests that require them.
    * In this case, the test file cannot be in package foo because it uses bar/testutil, which imports foo. So we use the 'import .' form to let the file pretend to be part of package foo even though it is not. Except for this one case, do not use import . in your programs. It makes the programs much harder to read because it is unclear whether a name like Quux is a top-level identifier in the current package or in an imported package.
    * Avoid meaningless package names like util, common, misc, api, types, and interfaces



* Goroutine
    * When you spawn goroutines, make it clear when - or whether - they exit.
    * Goroutines can leak by blocking on channel sends or receives: the garbage collector will not terminate a goroutine even if the channels it is blocked on are unreachable.
    * Even when goroutines do not leak, leaving them in-flight when they are no longer needed can cause other subtle and hard-to-diagnose problems. 
    * Try to keep concurrent code simple enough that goroutine lifetimes are obvious. If that just isn't feasible, document when and why the goroutines exit.

    
* Do not use package math/rand to generate keys, instead, use crypto/rand's Reader.







Multi-Threading 
---------------
* Remember using 'channel'
	- If we close the channel and we tryto read data (even AFTER reading the data) no exception thrown

		func main(){
			threadChannel := make(chan string,1) // size chosen to ensure it never gets filled up completely.


			go func() {
				threadChannel <- "TESTING from function 1"
				defer close(threadChannel)
			}()

			fmt.Println("Obtained  1" , <-threadChannel)
			time.Sleep(1000 * time.Millisecond)
			fmt.Println("----------------")
			fmt.Println("Obtained  2" , <-threadChannel)
		}


	- If we DO NOT close the channel and try to read data (AFTER reading the data) deadlock exception will be thrown


		func main(){
			threadChannel := make(chan string,1) // size chosen to ensure it never gets filled up completely.


			go func() {
				threadChannel <- "TESTING from function 1"
				//defer close(threadChannel)
			}()

			fmt.Println("Obtained  1" , <-threadChannel)
			time.Sleep(1000 * time.Millisecond)
			fmt.Println("----------------")
			fmt.Println("Obtained  2" , <-threadChannel)
		}

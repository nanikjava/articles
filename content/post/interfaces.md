The following is a function type with function called ServeHTTP

	type HandlerFunc func(ResponseWriter, *Request)

	func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
		f(w, r)
	}

having function declared as follow

	
	func final(w http.ResponseWriter, r *http.Request) {
		log.Println("Executing finalHandler")
		w.Write([]byte("OK"))
	}

we can wrap the final(..) function as handler using the following

	finalHandler := http.HandlerFunc(final)

After wrapping, the finalHandler will hace access to the ServerHTTP(..) function
which basically is calling the final(....) function


This is the way to wrap a function (final) that has the same signature 
as the named wrapper function (ServeHTTP). The benefit of doing this is the ability
for other code to just know the type function (ServeHTTP) and call it with the comfort
of knowing that it will call the correct implementation function.

Another example:

	package main

	import (
		"fmt"
	)

	type MyHandlerFunc 	func(string, int)

	func (f MyHandlerFunc) FunctionOne(s string, i int){
		fmt.Println("FunctionOne = ", s , " with i = ", i)
		f(s,i)
	}

	func (f MyHandlerFunc) FunctionTwo(s string, i int) {
		fmt.Println("FunctionTwo = ", s, " with i = ", i)
		f(s,i)
	}

	func normalFunction(s string, i int) {
		fmt.Println("normalFunction = ", s, " with i = ", i)
	}


	func main() {
		finalHandler := MyHandlerFunc(normalFunction)
		finalHandler.FunctionOne("fromFunctionOne",10)
		finalHandler.FunctionTwo("fromFunctionTwo",10)
	}

Another example:

	package main

	import (
		"fmt"
		"log"
		"net/http"
	)

	func middlewareOne(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			log.Println("Executing middlewareOne")
			next.ServeHTTP(w, r)
			log.Println("Executing middlewareOne again")
		})
	}

	func middlewareTwo(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			log.Println("Executing middlewareTwo")
			if r.URL.Path != "/" {
				return
			}
			next.ServeHTTP(w, r)
			log.Println("Executing middlewareTwo again")
		})
	}

	func final(w http.ResponseWriter, r *http.Request) {
		log.Println("Executing finalHandler")
		w.Write([]byte("OK"))
	}

	func main() {
		finalHandler := http.HandlerFunc(final)
		http.Handle("/", middlewareOne(middlewareTwo(finalHandler)))
		http.ListenAndServe(":3000", nil)
	}

Another example:

	type tokenizer func() (token string, ok bool)

	func split(s, sep string) tokenizer {
		tokens, last := strings.Split(s,sep),0

		return func() (string, bool) {
			if  len(tokens) == last {
				return "", false
			}

			last = last+1
			return tokens[last-1], true
		}
	}


Interface using inside github.com/tdewolff/minify
--------------------------------------------------

minifier.go
------------
Global variable declared as


	var minifier = minify.New()

THe init() function do the following


	func init() {
		htmlMinify := &html.Minifier{
			KeepWhitespace:   true,
			KeepDocumentTags: true,
		}
		minifier.AddFunc("text/css", css.Minify)
		minifier.Add("text/html", htmlMinify)
		minifier.Add("text/x-template", htmlMinify)
		minifier.AddFunc("text/javascript", js.Minify)
		minifier.AddFunc("application/javascript", js.Minify)
		minifier.AddFunc("application/x-javascript", js.Minify)
		minifier.AddFunc("image/svg+xml", svg.Minify)
		minifier.AddFuncRegexp(regexp.MustCompile("[/+]json$"), json.Minify)
		minifier.AddFuncRegexp(regexp.MustCompile("[/+]xml$"), xml.Minify)
	}

the code inside init() add the relevant function for the different mime type:
eg: for text/css it will call css.minify

The following code is the code that is called in the above .AddFunc(..)

	func Minify(m *minify.M, w io.Writer, r io.Reader, params map[string]string) error {
		return (&Minifier{}).Minify(m, w, r, params)
	}

The following code 

	(&Minifier{}).Minify(m, w, r, params)


means to call the .Minify(..) pointer receiver that is part of the Minifier struct type. Following is 
the declaration of the pointer receiver


	func (o *Minifier) Minify(m *minify.M, w io.Writer, r io.Reader, params map[string]string) error {
		....
		isInline := params != nil && params["inline"] == "1"
		....
		if err := c.minifyGrammar(); err != nil && err != io.EOF {
			return err
		}
		return nil
	}



Reference reading:
-----------------
* https://karthikkaranth.me/blog/functions-implementing-interfaces-in-go/
* http://technosophos.com/2014/05/05/adding-methods-to-function-types-in-go.html

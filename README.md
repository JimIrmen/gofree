# gofree
Like many in the development community the company I currently work for has embraced the Kubernetes environment as their container manager/orchestrator of choice. Kubernetes, originating from the Google family of software products is written in **Golang** or as many prefer to call it **Go**. Many of the examples for custom Kubernetes controllers and operators are written in Go (although this is not necessary it seems to be a trend). This fact, has lead to an investigation and analysis of the Go language for understanding Kubernetes capabilities as well as future development efforts within the Kubernetes environment.
## Golang First Thoughts
Like many newbies to the language a few idiosyncrasies of the Go language become apparent. I list a few of the more obvious ones.
* Declaring variables and parameters is backwards. That is, instead of

```java
   int aFunc(String parm0, int parm1){
   
      String aStringVar;
      int aIntVar;
      return 0;
      }
```
you have:

```go
   func aFunc(parm0 string, parm1 int) int {
   
      var aStringVar string;
      var aIntVar int;
      return 0;
      }
```
Note the additional **var** keyword. This kind of makes switching from one language to another a little more difficult and it's hard to justify the strange syntax, given no obvious advantage to the change in syntax vs. C, C++, Java, et. al.
* Go prefers a much more limited way to do things. That is you only have one looping statement, the for statement vs. other languages that give the developer options. You also only have one increment/decrement operator ++/-- used as a suffix.
* Go is strongly typed but encourages loosely written applications. For example Go promotes the following syntax.

```go
   myVar := someFunction();
```
In this context myVar is allocated and assigned without the developer having to know what type the function returns. Contrast that with explicit typing such as:

```go
   var myVar CustomType = someFunction();
```
Although the former may look more elegant, I personally prefer the latter for maintenance and clarity of code.
* These are just few of the obvious differences and they all pale in comparison to the one difference that annoys me (and many others) the most. The Go team decided that one of the largest difficulties in programming today is statement termination. That is, the evil of forcing developers to use a semi-colon. So the Go team wrote a parser and scanner that automatically inserts semi-colons in the code whenever a newline is encountered. The result is Go developers don't have to use semi-colons to terminate a Go statement, instead a statement is terminated by a newline character (how is this better?). It is important to note that the actual specifications of the Go language specify the requirement for a terminating statement character, the semi-colon.
## Automatic Semi-Colon Insertion
On the surface automatic semi-colon insertion sounds like a good thing. In practice this seemingly helpful decision forces a strict code format on the developer. For example, developers are forced into the following code format:

```go
   // opening brace must be on same line as statement
   
   type MyType struct{
      a string
      b string
   }
      
   // opening brace must be on same line as block statement for/if/switch/struct..etc.
   if x == y {
      x = y * 2
   }
   
   // else/else if statement must be on same line as closing brace
   if x == y {
      x = y * 2
   } else {
      x = y
      }
      
   // conditions that span lines must end with condition operator
   
   if x == y &&
         y != 0 {
      x = 0
   }
   
   // fluent code must have continuation period on same line.
   
   fluent.
      func1().
      func2().
      func3()
```
In simple terms developer's are restricted to using a K&R(1TBS) style. Unfortunately I prefer a Ratliff style, others perhaps an Allman, or Whitesmiths. What annoys me the most is the fact that I'm rather old school. I was taught as a young CS student that programming languages should not make format part of the syntax, a design error by many early language developers. So the Go team forcing such format just rubs me the wrong way.
##Other Strange Syntax Effects
There is also one special rule about semi-colons and automatic insertion. Note the following is a valid Go statement with automatic semi-colon insertions.

```go
 // No semi-colons needed 
 
   if x == y {x = y * 2}else{y = x * 2}
```
Why no semi-colons? there is only one newline character, at the end of the line and clearly there are two statements without terminating semi-colon's or a newline character separating the statements. 

Turns out, the Go team decided to make an exception, or special case for semi-colon insertion. You don't need to have any terminating character newline, or semi-colon if the statement is followed by a closing brace. So not only is the above code snippet valid so is the following:

```go
 // crazy indentation allowed
 
   if x == y {
      x = y *2 } else {
      z = x + y
      q = y -x
      y = x *2 }
   if q > z { 
      doSomething()}
```

## The Fix
Faced with only two choices, abandoning Go all together, or making the current Go offering work the way I want it to. I chose the latter. Not only would it make working with Go much more pleasant but it would be an excellent way for me to learn and work with the Go programming language.

Initially my thoughts were around a compiler flag that could indicate turning off the automatic semi-colon insertion logic. Something like: 
```
   go build -noauto hello.go
```
Although this might work what about building an entire directory of go files where some need automatic semi-colon insertion and some do not? A simple build flag didn't look like the way to go.

So then I thought what about a simple directive, similar to the go or line directives? By placing this directive as the first line of a go program it could signal the need to turn automatic semi colon insertion off. For example:

```
//AutoSemiOff - Note AutoSemiOn is of course the default if nothing is specified
package main;
import "fmt";
import "os";

func main() {

      if len(os.Args) > 1 {
         fmt.Println("Program args: ",os.Args[1]);
         }
      else {
         fmt.Println("No Program args supplied");
         }
   }
```
This would work for my immediate purposes and I did this as a first proof of concept. Simple and effective. Then as I was pouring through the Go source code it hit me. I'm always looking at other developers code. Many times I modify or copy their code and incorporate it into my own projects. Shouldn't there be a way to turn automatic semi-colon insertion on and off? This would avoid the need to either adapt to the strict Go format for code written in that manner, or to re-format the entire code to have semi-colons. So my final design is to incorporate two directives, AutoSemiOff and AutoSemiOn to control the automatic insertion of semi-colons. So something like this is possible:

```
//AutoSemiOff
package main;
import "fmt";
import "os";


func main() {

// Ratliff Style

      if len(os.Args) > 1 {
         fmt.Println("Program args: ",os.Args[1]);
         }
      else {
         fmt.Println("No Program args supplied");
         }
   // This might be some legacy code no semi-colons
   //AutoSemiOn - K&R Style
      if len(os.Args) > 2 {
         fmt.Println("Extra Program args: ",os.Args[2])
      } else {
         fmt.Println("No Extra args supplied")   
         }
   }
```
For those that prefer a Allman or Whitesmiths style, yes this is possible:

```go
//AutoSemiOff
package main;
import "fmt";
import "os";


func main() {

// Allman Style

      if len(os.Args) > 1 
      {
         fmt.Println("Program args: ",os.Args[1]);
      }
      else 
      {
         fmt.Println("No Program args supplied");
      }
   // This might be some legacy code no semi-colons
   //AutoSemiOn - K&R Style
      if len(os.Args) > 2 {
         fmt.Println("Extra Program args: ",os.Args[2])
      } else {
         fmt.Println("No Extra args supplied")   
      }
   }
```
## How To Make It Work
So how do you install these modifications to get Go the way you like it? First download(clone) the go source code can be found [here](https://github.com/golang/go) Follow the instructions for [Install from source](https://golang.org/doc/install/source) in my case I did the following:
* Downloaded and installed latest version of Go (for Linux)
* Cloned Go repository
* checked out **go1.15rc1** tag (this is the only version I've worked with)
* Built Go per instructions

Once that is all completed copy the 3 source files found under [src/compile/cmd/internal/syntax](https://github.com/JimIrmen/gofree/tree/master/src/cmd/compile/internal/syntax) to the appropriate location of your go source code.

Then build... now play as you want
## Don't forget the Semi-Colons
As you write code that now requires semi-colons there will be times you scratch your head and ask, "Do I need a semi-colon?". This is mainly because the documentation is centered on the Go offering from the Google team and not the actual syntax dictated by the EBNF of Go. When in doubt consult the [EBNF](https://golang.org/ref/spec) or just note: **any newline = ;**.
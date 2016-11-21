# Golang
Notes from Code School's [On Track with Golang](http://campus.codeschool.com/courses/on-track-with-golang/) course

Go is an open-source programming language created by google in 2007
It's compiled, so a compiler generates a single executable file
Leaves less room for unintended problems when we run the program
Statically types
Fast -- concurrency is built in
Easy to deploy -- can be as simple as copying an executable to a server


Used for systems programming
Lower-level programs that provide services to other systems
Tend to run straight from the command line

Application programs provide services to humans, allowing users with no technical background to perform tasks (have a GUI)
System programs provide services to other systems (APIs, game engines, network applciations, command line apps)

Course synopsis:
  - Building and running programs
  - Importing and creating prackages
  - Basic constructs (functions, variables, loops)
  - Data types
  - Concurrency with goroutines

Simple Go program will have a single source code file, will compile to an executable, and we'll run it to print a message to the console.

## main
All runnnable Go programs must have a `main` package and one `main` function
The Go environment assumes all projects live under a src folder

    # src/hello/main.go
    package main

    import "fmt" # to print from main(), we'll use the `Println()` function from the `fmt` package

    func main() {
      fmt.Println("Hello, I am a Gopher")
    }

Packages used in a program MUST be explicitly imported
Semicolons are not required

In order to build a compiled executable file, run the built-in `go build` command from the project root, creates an executable file with no extension (`./hello`)
Or, you can build and run in a single command (also built-in) `go run main.go`

`go run` is rarely used in production, but great for development

`go fmt` is another built-in command used to format code --  `gofmt -w main.go` -- -w flag writes back to the source file rather than printing to the console

Benefits of falling `gofmt` standards:
  - universal style decisions -- uncontroversial
  - easier to write, read, and maintain programs
  - text editors, IDEs can automatically run `gofmt`

## Conditionals, Args, and Imports

Printing two different messages (depending on user-supplied argument). If no argument is given, print default message.

    # argument "Into the tunnel" gets printed
    $ go run main.go "Into the tunnel"

    # no argument -- prints default "Hello, I am Gopher"
    $ go run main.go

    # src/hello/main.go
    package main

    import (
      "fmt"
      "os"  # import package from the standard library
    )

    func main() {
      if len(os.Args) > 1 {
        fmt.Println(os.Args[1])
      } else {
        fmt.Println("Hello, I am a Gopher")
      }
    }

    # os.Args is an array with the program arguments, starting with the name of the executable
    # and followed by any user-supplied arguments

Use the `goimports` command to detect missing packages and automatically update import statements in the source code -- also formats code according to `gofmt` standards

    $ goimports -w main.go
    # write back to the source file rather than print to console with -w flag

## Variable and Types

Above main() function can be refactored so there are not repetitive references to `os.Args`

    # src/hello/main.go
    package main

    import (
      "fmt"
      "os"  # import package from the standard library
    )

    func main() {
      args := os.Args
      if len(args) > 1 {
        fmt.Println(args[1])
      } else {
        fmt.Println("Hello, I am a Gopher")
      }
    }

    # the `:=` operator tells Go to automatically find out the data type on the right
    # being assigned to the newly declared variable on the left
    # this is known as **type inference**

Storing Values as Data Types
Figure out how much memory is required to store a value

Important not to assign a value to a data type that allocates more memory than is necessary, or not enough

For every value, there is a **proper data type**that determines owthe value should be stored and te operations that can be done on values of that type

    # Type Inference
    # Data Type inferred during assignment
    # variableName := value
    message := "Hello, Gopher"

    # Manual Type Declaration
    # Data type is dceclared prior to assignment
    # var variableName dataType
    # variableName = value

    var message string
    message = "Hello, Gopher"

Static typing allows the Go compiler to check for type errors before the program is run

    # NOPE
    var age string
    age = 42

    # YEP
    var age int
    age = 42

Type inference requires less code, though type inference and manual type declaration can be used interchangeably

Data types:
  - int       --> 42
  - string    --> "Hello"
  - bool      --> true or false
  - []string  --> ["acorn", "basket"] # also called primitive data values

## Functions

Write a program that prints out a different message depending on the time of day it's run

    # the getGreeting() function signature
    # Named functions in Go must have a name, followed by any arguments they expect,
    # and end with the data type they return (if any).
    # This is commonly referred to as the function signature
    ...

    func main() {
      hourOfDay := time.Now().Hour()
      greeting := getGreeting(hourOfDay)
    }

    func getGreeting(hour ?) string {

    }

    # How do we find out the data type of the hour argument to be passed
    # into the getGreeting function?
    # We can refer to http://go.codeschool.com/go-time-package
    # According to the docs for the `time` package, `Hour()` returns a
    # value of type `int`

    func main() {
      hourOfDay := time.Now().Hour()
      greeting := getGreeting(hourOfDay)
      fmt.Println(greeting)
    }

    func getGreeting(hour int) string {
      if hour < 12 {
        return "Good Morning"
      } else if hour < 18 {
        return "Good Afternoon"
      } else {
        return "Good Evening"
      }
    }


## Working with errors

Do not allow the previous program to be run before 7am

### Declaring multiple return values
In Go we communicate errors via a separate return value.

    # add the error return value

    import (
      "errors"
    )

    ...

    func getGreeting(hour int) (string, error) {
      var message string
      if hour < 7 {
        err := errors.New("Too early for greetings!")
        return message, err  # return empty string and a new error
                             # message has not yet been assigned a value, so
                             # defaults to zero value of empty string
      }
      if hour < 12 {
        message = "Good Morning"
      } else if hour < 18 {
        message = "Good Afternoon"
      } else {
        message = "Good Evening"
      }
      return message, nil    # a `nil` value for error tells the caller this
                             # function has no error
    }

### Zero values
A zero value in Go is the **default value**assigned to variables declared without an explicit initial vaue
  - string --> ""
  - int    --> 0
  - bool   --> false
  - find more at http://go.codeschool.com/go-zero-value (floats 0.0, bytes 0, functions nil)

So, how to we incorporate the multi-value output in the `main()` function?

    # returns message and error

    import (
      "os"
      "errors"
    )

    func main() {
      hourOfDay := time.Now().Hour()
      greeting, err := getGreeting(hourOfDay)   # indicate two return values with comma

      if err != nil {
        fmt.Println(err)
        os.Exit(1)        # terminate the program with an exit code
                          # 1 means program finished, but with errors
                          # ...so other programs using our program will know
      }
      fmt.Print(greeting)
    }

## The for loop

the only looping construct in Go
No parentheses in for loops
the for keyword, followed by three components

    # init executed before first iteration
    # condition evaluated before each iteration
    # post executed at the end of each iteration

    for init; condition; post {

    }

    # e.g.:
    # src/hello/main.go

    package main

    import "fmt"

    func main() {
      for i := 0; i < 5; i++ {
        fmt.Println(i)
      }
    }

The for loop components are optional. We can create loops with variables declared previously in the code and a **single condition expression**

    # src/hello/main.go

    ...
    func main() {
      i := 0
      isLessThanFive := true
      for isLessThanFive {    # there is no way to break the loop
                              # because the value is true, so have to check
                              # value of i and set isLessThanFive to false
        if i >= 5 {
          isLessThanFive = false
        }
        fmt.Println(i)
        i++
      }
    }

    # also common to write for loops with no components
    func main() {
      i := 0

      for {
        if i >= 5 {
          break       # use `break` keyword to leave the loop when condition met
        }
        fmt.Println(i)
        i++
      }
    }

### Writing infinite loops
Infinite loops are widely used in networking programs.
Useful for **setting up listeners** and **responding to connections**

    func main() {

      for {
        someListeningFunction()
      }
    }

    $ go run main.go   # never exits the process, unless something internal
                       # to the someListeningFunction() function breaks the loop
                       # or until we kill the process

## Arrays and Slices

### Declaring arrays
When creating arrays via manual type declaration, we must set the **max number of elements**

    package main

    import "fmt"

    func main() {
      var langs [3]string    # can hold no more than 3 strings
      langs[0] = "Go"
      langs[1] = "Ruby"
      langs[2] = "JavaScript"

      langs[3] = "NOPE"      # adding to nonexistent space
      fmt.Println(langs)     # ERRORS OUT
    }

Arrays are not dynamic -- adding more elements than expected raises an **out-of-bounds error**
This removes some flexibility, which is why Go added slices, built on top of arrays

### Slices
Most array programming in Go is done with slices rather than simple arrays.

    package main

    import "fmt"

    func main() {
      var langs []string    # This creates a slice with a zero value of nil, which
                            # behaves exactly the same as an empty slice.
                            # It can be appended to using the built-in function
                            # `append()`, which takes two arguments:
                            # a slice and a variable number of elements

      langs = append(langs, "Go")
      langs = append(langs, "Ruby")
      langs = append(langs, "JavaScript")
      langs = append(langs, "YEP")
      fmt.Println(langs)    # prints all to the console
    }

## Slice Literals and Looping with Range

When we know beforehand which elements will be part of a slice, multiple calls to `append()` will start looking to verbose.
There's a better way...

Use a slice literal.

    # slice literal is a quick way to create slices wuth initial elements
    # via type inference
    # pass elements between curly braces
    package main

    import "fmt"

    func main() {
      langs := []string{"Go", "Ruby", "JavaScript"}  # element count is inferred from the
                                                     # number of initial elements
      fmt.Println(langs)
    }

Reading elements from a slice by index works exactly the same as in arrays, but it doesn't scale well once our slice grows or when the exact number of elements **is not known until the program is run**.

    func main() {
      langs := getLangs()

      fmt.Println(langs[???])
    }

    func getLangs() []string {
      ...
    }

So we can navigate a slice with `for` and `range`.
The `range` clause provides a way to iterate over slices.
When only one value is used on the left of `range`, then this value will be the index of each element on the slice, one at a time.

    func main() {
      langs := getLangs()

      for i := range langs {
        fmt.Println(langs[i])   # we can now safely use the index (of type `int`)
                                # to fetch each element from the slice
      }
    }

    func getLangs() []string {
      ...
    }

    # calling `range` on a slice actually returns two elements
    # the index and the element associated with it
    # If we don't use a variable that's been assigned, Go will produce an error

    func main() {
      langs := getLangs()

      for i, element := range langs {
        fmt.Println(element)   # NOT USING i variable, so will error
                               # instead, can use the underline character
      }
    }

    # the underscore tells Go that this value will not be used from
    # anywhere else in the code

    func main() {
      langs := getLangs()

      for _, element := range langs {
        fmt.Println(element)   # compiler ignores the index returned
                               # NOTE: if you only supply one return value
                               # from `range`, Go returns the index, not the element
      }
    }

## Structs

Add structure to data

Example: young gophers can jump high...

    gopher1Name := "Phil"
    gopher1Age := 30

    gopher2Name := "Noodles"
    gopher2Age := 90


    if gopher1Age < 65 {
      highJump(gopher1Name)
    } else {
      lowJump(gopher1Name)
    }

    func highJump() (name string) {
      fmt.Println(name, "can jump HIGH")
    }

    func lowJump() (name string) {
      fmt.Println(name, "can still jump")
    }

    # in above example we can hide unnecessary implementation details from the caller
    # using.... encapsulation

    gopher1 := gopher{name: "Phil", age: 30}
    gopher2 := gopher{name: "Noodles", age: 90}

    fmt.Println(gopher1.jump())
    fmt.Println(gopher2.jump())

    # Much better... but how to we get to this ^^^
    # using a **struct**

Declare a new `struct` type for a gopher.
A `struct` is a built-in type that allows us to group properties together under a single name.

    # type <name-of-struct> <primitive-type> {
    #   <property-1> <data-type>
    #   <property-2> <data-type>
    # }
    #
    # the `type` keyword indicates that a new type is being created

    type gopher struct {
      name string
      age int
    }

Need to allocate memory or the `struct`, so can use a struct literal:

    # the below allocate memoryand assign the result to variables

    func main() {
      gopher1 := gopher{name: "Phil", age: 30}
      gopher2 := gopher{name: "Noodles", age: 90}
    }

    # NOTE: the `struct` **creation** code must go in the main() function, but
    # the `struct` **declaration** code must go outside main()

A `struct` contains behavior in the form of **methods**.
The way we define methods on a `struct` is by writing a regular function and specifying **the `struct` as the explicit receiver**.

    # specify the receiver in parenstheses

    func (g gopher) jump() string {
      if g.age < 65 {                       # g.age and g.name are properties from the `struct`
        return g.name + " can jump HIGH"
      } else {
        return g.name + " can still jump"
      }
    }

    func main() {

    }

How to know when to refactor to hide implementation details from the caller?
Use the "Tell, Don't Ask" principle:

    Rather than asking for data and acting on it, we instead **tell** the program what to do.

## References and Values

### Working with pointers

Example: validating a gopher's age
A new function will set a value on a property from the struct passed as argument

    type gopher struct {
      name string
      age int
      isAdult bool    # new property to  determin whether gopher is adult
    }

    # a new function takes type gopher
    # <new-function> (gopher)

    # if gopher is pf age, `isAdult` property is set to true; otherwise, set to false
    # Note: this pattern of **modifying arguments passed to a function** can be found
    # in parts of the Go standard library.
    # The Scan() method from the database package is an example.

    # zero value for bool is `false`, so if we create a new gopher without
    # explicitly specifying the value of its `isAdult` property, it will be
    # set to false

    func main() {
      phil := gopher{name: "Phil", age: 35}
      fmt.Println(phil)     # {Phil 35 false}
    }

    # We'll create a validate age function to evaluate
    # whether a newly created gopher is an adult.

    func main() {
      phil := gopher{name: "Phil", age: 35}
      validateAge(phil)
      fmt.Println(phil)
    }

    func validateAge(g ???) {    # function doesn't return anything, but what type should we pass in?
      g.isAdult = g.age >= 21
    }

    # Let's pass in `gopher` as the type in the validateAge function above.
    # Passing a `struct` as an argument **creates a copy**
    # of all the values assigned to its properties
    # Above, the **copy** of the data is what is received as argument.
    # So... the `g.isAdult` value is assigned to the **copy** as well,
    # not to the original

Aside: understanding **values** and **references**
Example: music playlists

Say we want to create a playlist of songs from two different albums with songs A, B, and C on the first, and songs D, E, F on the second.

If we created a playlist with **values**, we might create copies of songs A, C, and F and build a playlist from them.

The more efficient way is to create a playlist using **references** to the songs. The playlist would actually be slots that point back to the original songs.

### Pass by Value/Reference in Go

    # A) Passing values (default behavior)
    language := "Go"

    # 1) the primitive value is assigned to new variable and
    # stored in a new memory address 0x10444444

    favoriteLanguage := language

    # 2) A new memory address 0x10555555 is allocated for the new
    # variable that receives a **copy of the original data**
    # So we have the exact same data stored in two different memory addresses

    # B) Passing References
    language := "Go"

    # 1) the primitive value is assigned to new variable and
    # stored in a new memory address 0x10444444

    favoriteLanguage := &language   # returns a pointer

    # 2) Using the `&` operator, a **reference** to the existing memory
    # address is assigned to the new variable

Passing `struct`s by reference:
In order to assign a `struct` reference to a new variable, we use the `&` operator to return a pointer.

    type gopher struct {
      name string
      age int
      isAdult bool
    }

    func main() {
      phil := &gopher{name: "Phil", age: 35}
      validateAge(phil)
      fmt.Println(phil)
    }

    func validateAge(g *gopher) {    # `*` indicates a **pointer** to the original gopher
      g.isAdult = g.age >= 21
    }

    # now when we run `go run main.go`, we get &{Phil 35 true}

## Interfaces

Example: calling `jump()` on multiple gophers
Static typing allows the Go compiler to ensure every element on the collection returned by `getList()` responds to the `jump()` method.

    type gopher struct { ... }
    func (g gopher) jump() string { ... }   # all gophers respond to `jump()`
    func main() {
      gopherList := getList()               # grab collection of gophers
      for _, gopher := range gopherList {
        fmt.Println(gopher.jump())          # safely call `jump()` on every element returned
      }
    }

    func getList() []*gopher {   # `*` means the return value is a slice of **pointers to gopher**
      ...
    }

    # from the `getList()` function, we can:
    # 1) create two gophers
    # 2) grab their pointers
    # 3) return them as part of a slice

    func getList() []*gopher {
      phil := &gopher{name: "Phil", age: 30}
      noodles := &gopher{name: "Noodles", age: 90}   # return pointers for created gophers

      list := []*gopher{phil, noodles}               # create slice with pointers
      return list                                    # return the slice from the function
    }

What if, for example, other types can also jump?
For example, a horse might be a different `struct` with different properties but the same **method signature**.

    type gopher struct { ... }
    func (g gopher) jump() string { ... }

    type horse struct {
      name string
      weight int
    }

    func (h horse) jump() string {    # SAME METHOD SIGNATURE AS `gopher` `jump()`
      if h.weight > 2500 {
        return "I'm too heavy, can't jump"
      }
      return "I will jump, neigh!!"
    }

But different types are... different.
We **cannot** combine both gopher and horse `struct`s under a single slice of type `*gopher`.

    func getList() []*gopher {
      phil := &gopher{name: "Phil", age: 30}
      noodles := &gopher{name: "Noodles", age: 90}
      barbaro := &horse{name: "Barbaro", weight: 2000}  # NOPE

      list := []*gopher{phil, noodles, barbaro}   # will not work
      return list
    }

But... we can use an **interface**
Interfaces provide a way to **specify behavior**: "If something can do **this**, then it can be used **here**"

    type jumper interface {
      jump() string     # method expected to be present in all types that implement this interface
    }

    func getList() []jumper {  # can be used as return type (`*` not necessary with interfaces)
      phil := &gopher{name: "Phil", age: 30}
      noodles := &gopher{name: "Noodles", age: 90}
      barbaro := &horse{name: "Barbaro", weight: 2000}

      list := []jumper{phil, noodles, barbaro}   # will not work
      return list
    }

    # Bringing it all together....

    type gopher struct { ... }
    func (g gopher) jump() string { ... }
    type horse struct { ... }
    func (h horse) jump() string { ... }

    type jumper interface {
      jump() string
    }

    func main() {
      jumperList := getList()
      for _, jumper := range jumperList {
        fmt.Println(jumper.jump())
      }
    }

    func getList() []jumper {
      ...
    }

### Naming convention for interfaces
A convention for naming one-method interfaces in Go is to use the method name plus an **-er** suffix.

    type jumper interface {
      jump() string
    }

Examples form the Go standard library thst follow this convention are the `Reader` interface with the `Read()` method, and the `Writer` interface with the `Write()` method.

## Boundaries

### Creating packages

When a single file goes too long, move code into different packages. New project structure:

    -src
    ---hello
    -----main.go    # entry point for our program
    -----model      # new package with a single file
    -------main.go  # naming convention in go for single files within a package

    # src/hello/model/main.go  (NEW PACKAGE FILE)

    package model     # NEW PACKAGE DEFINITION

    type gopher struct { ... }
    func (g gopher) jump() string { ... }
    type horse struct { ... }
    func (h horse) jump() string { ... }
    type jumper interface { ... }

    func GetList() []jumper { ... }   # by making first letter of function name uppercase,
                                      # we make it accessible from outside packages

    # src/hello/main.go  (ORIGINAL PACKAGE FILE)

    package main

    import (
      "fmt"
      hello/model
    )

    func main() {
      jumperList := model.GetList()
      for _, jumper := range jumperList {
        fmt.Println(jumper.jump())   # NOPE -- cannot call the `jump()` method like this
                                     # will get method undefined because that method was not exported
                                     # go back and change the `jump()` method names to uppercase
      }
    }

    # src/hello/model/main.go
    ...
    type gopher struct { ... }
    func (g gopher) Jump() string { ... }
    type horse struct { ... }
    func (h horse) Jump() string { ... }
    type jumper interface {
      Jump()   # only the method names need to be exported, not the structs or the interface
    }

    # src/hello/main.go
    ...
    func main() {
      jumperList := model.GetList()
      for _, jumper := range jumperList {
        fmt.Println(jumper.Jump())   # YEP
      }
    }

## Multiple tracks
Multiple tasks broken into chunks which are executed independently and may appear to complete simultaneously.
Parallel programs (programs that can be executed across multiple CPUs) can execute the different tasks **actually** simulataneously and only take as long to complete as the longest task.

#### Concurrency is not parallelism, but concurrency allows parallelism

### Goroutines
A `goroutine` is a special function that executes **concurrently** with other functions.
We create them with the `go` keyword

    # in the below, function `g()` must wait until function `f()` is completed
    f()     # takes ---------ms
    g()     # takes          ------ms

    # with concurrency on a single-core machine, the tasks may be chunked,
    # but still takes the same amount of time to run, even with `goroutine`
    go f()  # takes ---    ---- --ms
    g()     # takes    ----    -   -ms

Example to demonstrate concurrency and parallelism:

    # Looping and printing names
    package main

    import "fmt"

    func main() {
      names := []string{"Phil", "Noodles", "Barbaro"}
      for _, name := range names {
        printName(name)
      }
    }

    func printName(n string) {
      fmt.Println("Name:", n)   # prints argument to the console
    }

    # we can use the Unix `time` command to track the duration of the execution
    $ time go run main.go

    # Name: Phil
    # Name: Noodles
    # Name: Barbaro
    #
    # real 0m0.321s

    # So let's add a time-consuming math operation from Go's standard math library

    import (
      "fmt"
      "math"
    )
    ...
    func printName(n string) {
      result := 0.0
      for i := 0; i < 10000000; i++ {
        result := math.Pi * math.Sin(float64(len(n)))
      }
      fmt.Println("Name:", n)   # prints argument to the console
    }

    $ time go run main.go

    # Name: Phil
    # Name: Noodles
    # Name: Barbaro
    #
    # real 0m11.603s    # blocks execution at each iteration until expensive math operation completes

Going concurrent...
Go programs **are NOT automatically aware** of newly created `goroutines`, so the main function exits before the `goroutines` are finished.

    ...
    func main() {
      names := []string{"Phil", "Noodles", "Barbaro"}
      for _, name := range names {
        go printName(name)    # executes each function call on a new `goroutine`
      }
    }

    func printName(n string) {
      ...
    }

The above is enough to get the three iterations to run concurrently (and gets the duration back down to ~1/3 of a second), but go exits the program without printing the names to the console.
So, we need to add synchronization with `WaitGroup`

On the `sync` package from Go's standard library, there's a `WaitGroup` data type.
We can use this type to make our program wait for `goroutines` to finish.

    import (
      "fmt"
      "sync"    # import the new package
      "math"
    )

    func main() {
      names := []string{"Phil", "Noodles", "Barbaro"}
      var wg sync.WaitGroup   # declare a new variable of the `sync.WaitGroup` data type
      wg.Add(len(names))      # sets the number of `goroutines` to wait for
      for _, name := range names {
        go printName(name, &wg)   # pass in the reference to the `WaitGroup` to the function
      }
      wg.Wait()               # prevents the program from exiting before all `goroutines`
                              # being tracked by our `WaitGroup` are finished executing
                              # the `Done()` method must be called at the end of the
                              # `goroutine` once it's finished to update the `WaitGroup`
    }

    func printName(n string, wg *sync.WaitGroup) {  # pass in the `WaitGroup` as second argument
      ...
      wg.Done()
    }

Now we run the function, but we want to specify it to be run on a single processor

    $ time GOMAXPROCS=1 go run main.go
    # Name: Phil
    # Name: Noodles
    # Name: Barbaro
    #
    # real 0m11.675s ... still taking around 11 seconds

The good news is that the Go runtime **defaults to using all processors available**.
Most machines today have more than one processor and our concurrent Go code can run in parallel with no changes!

    $ time go run main.go
    # Name: Phil
    # Name: Noodles
    # Name: Barbaro
    #
    # real 0m4.172s ... much faster using multiple CPUs














# Errors

Go does not have exceptions. 

Despite this absence being a potential show-stopper for many programmers, Go's FAQ delivers a rather bombastic response:

> We believe that coupling exceptions to a control structure,
> as in the try-catch-finally idiom, results in convoluted code.<sup>[link](https://golang.org/doc/faq#exceptions)</sup>

The absence of exceptions has far reaching effects for Python code, as [duck typing](https://en.wikipedia.org/wiki/Duck_typing) and its main method of error handling - exceptions - is no longer feasible in Go.

Instead of exceptions, Go offers two methods of error handling: a simple interface called [`error`](http://golang.org/pkg/builtin/#error) that can be used within normal program execution, and the built-in methods [panic](http://golang.org/pkg/builtin/#panic) and [recover](http://golang.org/pkg/builtin/#recover) that - respectively - halt and regain program execution.

None of these, however, are a replacement for exceptions. Code bases that depend on exceptions for program flow may require a significant re-write.

But the gulf between Pythonic code and idiomatic Go is far less than many programmers suspect...


### Programmer Error

First, the good.

By being type-safe, Go removes entire classes of errors normally found in dynamic languages, including:

- Failing to initialize/pass a parameter
- Passing/returning the wrong type
- Forgetting to use/return a variable
- Typos

All of these errors become compile-time errors in Go. The result is not only a significant reduction in runtime bugs, but more meaningful unit tests, since many runtime errors are now impossible and will no longer require testing.


### Handling Errors

Since Go supports multiple return values, a common idiom throughout the standard library - and many third party packages - is the return of a result type and an `error`. For example, in Go's `strconv` package:

```go
func ParseBool(str string) (value bool, err error)
```

Handling a failed conversion of a string type to a boolean in Go then becomes:

```go
value, err := strconv.ParseBool(input)
if err != nil {
    return err // Or set a default value
}
```

Although constant error checking is quite ugly, it is often a truer representation of a program's flow. Take for instance the conversion of a integer string representation to an integer type in Python:

```python
value = int("1")
```

Although succinct, consider the error condition:

```python
value = int("a")
```
```
ValueError: invalid literal for int() with base 10: 'a'
```

To handle this error immediately within program flow - without being overly broad - the code would be:

```python
try:
    int(input)
except ValueError as err:
    print(err) # Or default / exceptional case
``` 

The preceding code also ignores additional cases - admittedly exceptional - such as passing the `NoneType`:

```python
int(None)
```
```
TypeError: int() argument must be a string, a bytes-like object or a number, not 'NoneType'
```

The Python code quickly becomes preoccupied with defensive programming and exceptional cases while the analogous Go function - the `ParseInt` function in `strconv` - can remain focused on the original question: can the given string value be converted to an integer type?


### Ignoring Errors

Additionally, checking the error within Go is only necessary when the result type's return value - likely the type's zero initialization value - is an exceptional case.

For example, when an error is returned from `ParseBool`, so will the zero value of the return value, which in the case of a boolean is `false`.

If only the `true` case is important to the program, the error can be safely ignored:

```go
value, _ := strconv.ParseBool(input)
if value {
    // Do something
}
```

The assignment and conditional can be inlined if the value is only needed within the scope of the conditional:

```go
if value, _ := strconv.ParseBool(input); value {
    // Do something
}
```

Please note, however, that not all third-party Go libraries respect this idiom. Some do so out of negligence, but others for good reason: the `error` is reserved for exceptional cases, such as network timeout or insufficient permissions, and the result type must still be checked for validity even in absence of an error.


### It's My Party, and I'll Error If I Want To

Custom errors can be constructed using the standard library. The `fmt` package provides a function called [`Errorf`](https://golang.org/pkg/fmt/#Errorf) that uses the same verbs as string formatting but returns an `error`:

```go
func Errorf(format string, a ...interface{}) error
```

There is also an [`errors` package](http://golang.org/pkg/errors/) with a sole function `New` that can be used to construct errors that do not require the formatting options provided by `fmt.Errorf`.

Most importantly, the built-in `error` type is actually an interface:

```go
type error interface {
    Error() string
}
```

Any custom type that implements the interface's `Error` method can behave as a built-in error. Types such as [`SyntaxError`](http://golang.org/pkg/encoding/json/#SyntaxError) from the standard library package `encoding/json` implement this interface. Doing so allows the type to function anywhere an `error` is needed, but still provide additional exported fields if the programmer is willing to engage in runtime [type assertion](https://golang.org/ref/spec#Type_assertions).


### Do Panic

Although strict error checking encourages strong defensive programming and thoughtfulness in regards to error conditions, it may require errors to bubble up through deeply nested code. 

There remains a significant use case for exceptions that Go's errors cannot handle - since they are bound to normal program execution - and that is the immediate halt of programs because of exceptional cases.

Go's best answer to this question is a controversial one - `panic` and `recover`. They are both built-in functions, and they respectively provide a way to halt or regain program execution.

They are used sparingly in the Go standard library. Examples include: when a [bad base has been provided to `fmt`](https://github.com/golang/go/blob/release-branch.go1.4/src/fmt/format.go#L231):

```go
panic("fmt: unknown base; can't happen")
```

Or in [`database/sql` when a registering a duplicate driver name](https://github.com/golang/go/blob/release-branch.go1.4/src/database/sql/sql.go#L35):

```go
panic("sql: Register called twice for driver " + name)
```

Likewise, `recover` is used to prevent the failed program flow from propagating, such as [`net/http` stopping requests from crashing the entire server](https://github.com/golang/go/blob/release-branch.go1.4/src/net/http/server.go#L1126-L1137):

But despite their use in the standard library, the "stop the world" behavior of `panic` and indiscriminate nature of `recover` should be avoided wherever possible. They should only be employed when all other error conditions have been exhausted or are meaningless.

For example, since the `net/http` library wraps all its handlers in deferred `recover` statements, calling `panic` when the request can no longer be meaningfully completed is a valid strategy - for example on the failure of the local filesystem, a critical datastore, or an integral remote service.


### Duck Typing

Without a class of errors that can halt program execution in specific ways, code will fail hard and in ambiguous ways. Ultimately, duck typing is a contract that a certain chunk of code can behave in certain ways. Removing specific exceptions cripples this contract - as in Go, where errors become hyper-specific and often mandatory, or overlay broad as in `panic`/`recover`.

To prevent this disaster, the programmer must either have an intimate  knowledge of all code used within the system - including third party libraries - or there must be a better way to enforce the contract, which Go does, through the interface.


### Additional Reading

- [Error handling and Go](http://blog.golang.org/error-handling-and-go)
- [Defer, Panic, and Recover](http://blog.golang.org/defer-panic-and-recover)


From [Golang for Pythonistas](https://github.com/aodin/golang-for-pythonistas)

Happy hacking!

aodin, 2015

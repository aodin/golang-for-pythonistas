# Packages and Imports

Python and Go both have a concept of packages and import statements. The practice of importing, however, varies drastically between the two, mostly because of how Python imports on a per-file basis (a.k.a. "modules"), while Go imports directories.

Importing entire directories in Python [is possible](https://stackoverflow.com/q/1057431), but relies on additions to the directory's `__init__.py` that may vary between versions and cause some unexpected behavior.


### Package Names

Go files must declare their package name on the first non-commented line of code. Any other code before the package name is a compiler error.

Package names should be kept short, preferably a lowercase word or words with legible abbreviations. While underscores and mixed cap names are permitted, they are actively discouraged. Here are some examples from the standard library:

```go
package fmt
package strconv
package httputil
```

A Go package does not derive its name from its repository, filename, or path, only the `package` declaration at the top of the file. It is best practice to have the last part of the path match the package name, but this convention is broken by many popular Go packages.

```go
import "github.com/dgrijalva/jwt-go" // Imports as jwt
```

The [gopkg.in](gopkg.in) service makes use of this separation, providing Go packages with versioned suffixes for use with `import`, but leaving the package name as is.

```go
import gopkg.in/yaml.v2 // Imports as yaml
```

This ambiguity between package and import path can be clarified using aliases, which will be explained in the next section.

All runnable source files within a directory must have the same package name. An exception is made for the special case of `package main`, which will be explained later.

It is possible to have a different package name for all test files within the same directory, but tests will only be able to access the packages' public interfaces. The distinction between public and private interfaces will also be explained later.


### Import Statements

A Go file does not require `import` statements. But if they do exist, they must be declared after the `package` statements and before any other code.

They can be declared in single or multi line syntax:

```go
import "log"

import (
    "log"
)
```

This import will allow the programmer to access all public interfaces of the package `log` using the namespace `log`:

```go
log.Println("Hello, log!")
```

The command line program `go fmt` will alphabetize packages within the multi line syntax.

Import statements are commonly grouped in a manner similar to [PEP8's recommendation](https://www.python.org/dev/peps/pep-0008/#imports):

1. Standard library imports
1. Related third party imports
1. Local application/library specific imports

An example using Go's multi-line import syntax:

```go
import (
    "log"
    "net/http"

    "golang.org/x/net/context"

    "github.com/mypackage/a"
    "github.com/mypackage/b"
)
```

If a package is imported, but not used, the Go compiler will error with the straightforward statement `imported and not used`. Some packages, however, are imported solely for their side-effects during initialization. To prevent this compiler error while imported, packages can be given the alias of the blank identifier `_`.

Examples of this include the registration of `crypto` packages in the standard library and third party packages that provide an implementation of `database/sql/driver`:

```go
import "database/sql"
import _ "github.com/lib/pq"
```

Note that `_` cannot be used as a value. If you later decide you wish to use the package, simply remove the blank identifier.

It is also possible to alias Go imports in a manner similar to Python `import .. as ..`:

```go
import pq2 "github.com/lib/pq"
```

Aliases are handy for clarifying packages where the imported repository does not match the package name that will be used as its namespace, such as the `jwt` package from earlier:

```go
import jwt "github.com/dgrijalva/jwt-go"
```

It is possible to have duplicate `import` statements for a package, either using the blank identifier or multiple aliases. The compiler will not error when given the following (as long as `pq` and `pq2` are used in the file:

```go
import _ "github.com/lib/pq"
import _ "github.com/lib/pq"
import "github.com/lib/pq"
import pq2 "github.com/lib/pq"
```

Despite duplicate imports, Go's dependency resolution guarantees that the package's initialization behavior will only be run once. Duplicated import statements should still be fixed by the author as they are an anti-pattern.

Finally, it is possible to import packages without their namespace:

```go
import . "fmt"

func main() {
    Println("No namespace")
}
```

You should, however, resist the urge to use dot imports. Code using a common entrypoint such as `list.New()` will lose its significance and force the programmer to reevaluate file imports. 

Additionally, attempting to dot import multiple packages with overlapping public interface names will cause a compile time `redeclared during import` error.


### Public versus Private

In Python, every interface - variable, function, class - is public. The language will not prevent you from accessing private or internal interfaces. This behavior is often accompanied with the quote ["we're all consenting adults here"](https://mail.python.org/pipermail/tutor/2003-October/025932.html).

Private interfaces, however, can be useful to hide code likely to change - in name or behavior - therefore simplifying the public interface and support for backwards compatibility.

The usefulness of private interfaces has led the Python community to develop a number of ways to create "pseudo-private" interfaces. [PEP8](https://www.python.org/dev/peps/pep-0008/#public-and-internal-interfaces) suggests that all documented interfaces be considered public and all undocumented interfaces private. It also recommends that public interfaces be declared in the module's `__all__` variable. It also suggests that private or internal methods be prefixed with a singled underscore.

While other languages, such as Java and C, introduce additional keywords to declare public and private interfaces, Go uses uppercase versus lowercase respectively to do so.

Therefore, a variable named `private` will only be accessible within the same package (which may be spread across any number of files in the directory), while a variable named `Public` will also be accessible from outside the package. This behavior is consistent for all interfaces in Go (all types, variables, and functions).


### Main Files

Any file in python can be run directly, even if it has no useful behavior as a script.

For files that the programmer would like to have both functionality as a library and a script, the following design pattern is used:

```py
if __name__ == '__main__':
    scripty_stuff()
```

It makes use of the global variable `__name__`, which will be set to `__main__` when the Python file is run directly, but set to its module name when imported.

For a Go file to be run directly, it must have its package be declared as `package main` and possess the entrypoint function `main()`, which has no arguments or return values.

Any number of main files can exist in the same directory, even alongside other packages, but it is best practice to separate them from library files, one per directory.


### Initialization

All Go files, including main files, can also make use of the special function `init()`, which like `main()`, has no arguments or return values.

The function `init()` will be run during package import once and only once, according to the rules of Go's dependency resolution. In a main file, it will be run before `main()`.

It is commonly used to provide runtime specific configuration or register the package's implementing interfaces in other package registries.


### Further Reading

* [The Go Blog: Package names](https://blog.golang.org/package-names)


From [Golang for Pythonistas](https://github.com/aodin/golang-for-pythonistas)

Happy hacking!

aodin, 2017

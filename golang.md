# Not So Random 

Go's `rand` package makes it easy to generate all sorts of random numbers.
What they don't tell you though is that the default seed is `1`. So if you
write a program like so:

```go
package main

import "fmt"
import "math/rand"

func main() {
    stuff := []string{
        "one",
        "two",
        "three",
        "four",
    }
    fmt.Println(stuff[rand.Intn(len(stuff))])
}
```

and then run it, you will get output like:

```
three
```

and any subsequent runs of the program will continue to produce `three`. Not
exactly what we are looking for.

If you want your program to be a little less predictable, you will want to
seed it yourself, perhaps with the current time, instead of `1`. Try adding
the following to the beginning of the `main` function:

```go
rand.Seed( time.Now().UTC().UnixNano())
```

You'll also want to import the `time` package.

Things should *appear* to be a bit more random now.

source: [Jake Worth](https://twitter.com/jwworth) and
[Stackoverflow](http://stackoverflow.com/questions/12321133/golang-random-number-generator-how-to-seed-properly)

# Replace The Current Process With An External Command 

Go's `syscall.Exec` function can be used to execute an external program.
Instead of forking a child process though, it runs the external command in
place of the current process. You need to give the function three pieces of
information: the location of the binary, the pieces of the command to be
executed, and relevant environment. Here is a simple example.

```go
package main

import "fmt"
import "os"
import "syscall"

func main() {
    // get the system's environment variables
    environment := os.Environ()

    // get a slice of the pieces of the command
    command := []string{"tmux", "new-session", "-s", "burrito"}

    err := syscall.Exec("/usr/local/bin/tmux", command, environment)
    if err != nil {
        fmt.Printf("%v", err)
    }
}
```

When this program is executed, it will replace itself with a new tmux
session named *burrito*.

# Sleep For A Duration 

Many languages allow you to sleep for a certain number of milliseconds. In
those languages, you can give `500` or `1000` to the sleep function to
sleep for half a second and a second respectively. In Go, the duration of a
call to [`time.Sleep`](https://golang.org/pkg/time/#Sleep) is in
nanoseconds. Fortunately, there are constants that make it easy to sleep in
terms of milliseconds.

For example, you can sleep for a half a second (500 milliseconds) like so:

```go
package main

import (
    "time"
)

func main() {
    time.Sleep(500 * time.Millisecond)
}
```

Other available time constants are `Nanosecond`, `Microsecond`, `Second`,
`Minute`, `Hour`.

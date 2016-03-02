# Aggregation Using merge-with 

Clojure provides the `merge-with` function as a way of conjoining a series
of maps. You must provide `merge-with` a function that it can use to merge
two values for matching keys. For instance, imagine you have a bunch of maps
that contain counts for entities identified by keywords. You can consolidate
the sum of all the counts into a single map using the `merge-with` function
combined with the `+` function.

```clojure
> (merge-with + {:a 1 :b 3} {:b 2 :c 3} {:c 1 :d 4})
{:a 1, :b 5, :c 4, :d 4}
```

For different kinds of data, a different function argument may be more
appropriate. For instance, aggregating lists instead of integers calls for
the `concat` function:

```clojure
> (merge-with concat {:a '(1 2) :b '(3 4)} {:c '(3 4) :a '(5 4 1)})
{:a (1 2 5 4 1), :b (3 4), :c (3 4)}
```

# Argument Requirements For A Function 

When defining a function, you must declare one or more function definitions,
each of which will require a different set of arguments. These argument
lists are stored as metadata for the function. So, if you are trying to
figure out what arity a function is or what variations of arguments it
takes, you can check the metadata like so:

```clojure
> (:arglists (meta #'str))
([] [x] [x & ys])
```

[source](http://stackoverflow.com/questions/1696693/clojure-how-to-find-out-the-arity-of-function-at-runtime)

# Evaluate One Liners With lein-exec 

You can install the [`lein-exec`](https://github.com/kumarshantanu/lein-exec)
plugin and then use it to evaluate one liner bits of code with the `-e`
flag.

```bash
$ lein exec -e '(println "foo" (+ 20 30))'
foo 50
```

# Expanding Macros 

Macros are an important part of Clojure's syntax. They allow you to write
cleaner, terser, more expressive code.
Though sometimes you may want to inspect the
clojure code that is actually produced by a particular macro. The
`macroexpand` function allows for just this.

For instance, if you have a snippet of code using the `->>` operator:

```clojure
(->> 4 (+ 1) (- 2) (* 3))
```

You can wrap that form with the `macroexpand` function to see the form that
is ultimately evaluated:

```clojure
> (macroexpand (->> 4 (+ 1) (- 2) (* 3)))
; (* 3 (- 2 (+ 1 4)))
```

It doesn't buy us much in a contrived example like this, but can prove
useful for better understanding clojure and the more complex code we write.

# Get The Value Of An Environment Variable 

You can get the value of an environment variable on your system using the
`System/getenv` function. Just pass it the environment variable as a string:

```clojure
> (System/getenv "HOME")
"/Users/jbranchaud"
```

It returns `nil` when the environment variable doesn't exist.

```clojure
> (System/getenv "HOUSE")
nil
```

# Load A File Into The REPL 

You can quickly load a file into a REPL session using the `load-file`
function. You can specify an absolute or relative path and it will

> sequentially read and evaluate the set of forms contained in the file.

```clojure
(load-file "path/to/file.clj")
```

See the [`load-file` docs](https://clojuredocs.org/clojure.core/load-file)
for more details.

# Mapping With An Index 

If you're mapping over a collection and you need an index for each item, you
can reach for `map-indexed`. The `map-indexed` function can be used like so:

```clojure
> (def foods ["pizza" "hotdog" "chicken alfredo"])
#'cljs.user/foods
> (map-indexed (fn [idx item] (str idx " - " item)) foods)
("0 - pizza" "1 - hotdog" "2 - chicken alfredo")
```

# Open JavaDocs 

Clojure has all kinds of Java interop including a fancy function called
`javadoc`. The `javadoc` function _opens a browser window displaying the
javadoc for the argument_.

Thinking about using Java's `ArrayList` class, but want to read up on it
first? Try

```
> (javadoc java.util.ArrayList)
true
```

The javadoc page for
[ArrayList](http://docs.oracle.com/javase/6/docs/api/java/util/ArrayList.html)
will be opened up in your default browser.

# Pretty Print The Last Thing 

Clojure provides `pp` as a convenience macro for pretty printing the last
thing that was output. If you are playing around with a function in the
repl, trying to get the output just right, `pp` can come in handy.

```clojure
> (fancy-func)
{:one {:a 1, :b 2, :c 3, :d 4}, :two {:b 2, :c 3, :d 4, :e 5}, :three {:c 3,
:d 4, :e 5, :f 6}, :four {:d 4, :e 5, :f 6, :g 7}}
> (clojure.pprint/pp)
{:one {:a 1, :b 2, :c 3, :d 4},
 :two {:b 2, :c 3, :d 4, :e 5},
 :three {:c 3, :d 4, :e 5, :f 6},
 :four {:d 4, :e 5, :f 6, :g 7}}
nil
```

See `(doc pp)` for more details.

# Quick Clojure Docs 

Assuming you already have the
[`fireplace.vim`](https://github.com/tpope/vim-fireplace) plugin installed
and a repl running, you can quickly pull up the docs for any clojure
function. Navigate the cursor over the keyword in question and hit `K` in
normal mode. This will instruct `fireplace.vim` to use the `doc` function on
the symbol for that keyword. The docs associated with that keyword will be
displayed at the bottom of the window.

# Reductions 

One of the core functions in Clojure is
[`reduce`](https://clojuredocs.org/clojure.core/reduce). It allows you to
build up some result based on applying a function to each value in a
collection. Clojure provides a similar function that builds up a (lazy)
sequence of intermediate values as it is performing a reduce.
This function is
[`reductions`](https://clojuredocs.org/clojure.core/reductions).

Using `reduce` to sum a collection of integers looks like this

```clojure
> (reduce + 0 [1 2 3 4 5])
=> 15
```

whereas `reductions` performing the same task will look like this

```clojure
> (reductions + 0 [1 2 3 4 5])
=> (0 1 3 6 10 15)
```

h/t Josh Davey

# Set Max Heap Size 

For `lein`-based projects, the maximum heap size can be set by including the
`-Xmx` option in the `jvm-opts` portion of the `project.clj` file. For
instance, to set the maximum JVM heap size to 2 gigabytes, include:

```clojure
  :jvm-opts ["-Xmx2g"]
```

in your `project.clj` file.

# Specify the Directory of a Shell Command 

Clojure gives us access to Java's shell capabilities through
`clojure.java.shell`. For instance, if you want to list the contents of your
project's directory, you can issue an `ls` command:

```
> (clojure.java.shell/sh "ls")
; {:exit 0,
;  :out "LICENSE\nREADME.md\ndoc\nproject.clj\nresources\nsrc\ntarget\ntest\n",
;  :err ""}
```

The default will always be to execute the command in the directory of the
containing project. It is likely that you'd like to specify a different
directory though. There is a function for that:

```clojure
(clojure.java.shell/with-sh-dir "some/dir" (clojure.java.shell/sh "ls"))
```

Or more concisely, you can specify the directory as part of the `sh`
function:

```clojure
(clojure.java.shell/sh "ls" :dir "some/dir")
```

# Splitting On Whitespace 

If you have a string with spaces and you want to split the string into a
vector of strings (delimited by the spaces), then you can do something like
this:

```clojure
(clojure.string/split "split me up" #" ")
; ["split" "me" "up"]
```

However, if you have extra spaces in your string, the output may not be quite
what you want:

```clojure
(clojure.string/split "double  spacing  wtf?" #" ")
; ["double" "" "spacing" "" "wtf?"]
```

A quick fix might look like this:

```clojure
(clojure.string/split "double  spacing  wtf?" #"[ ]+")
; ["double" "spacing" "wtf?"]
```

That's nice, but it is going to fall over as soon as we run into input with
tabs and new lines. Assuming we want to split on all whitespace, we should
tell our regular expression to do just that:

```clojure
(clojure.string/split "we\thave new\nlines and  tabs\n" #"[\s]+")
; ["we" "have" "new" "lines" "and" "tabs"]
```

# Swap Two Items in a Vector 

If you want to replace the value at an index in a vector, you can use the
`assoc` function supplied by `clojure.core` like so:

```clojure
> (assoc [5 6 7 8] 1 9)
; [5 9 7 8]
```

Likewise, if you want to replace two items in a vector, you can extend the
above like so:

```clojure
> (assoc [5 6 7 8] 1 2 2 4)
; [5 2 4 8]
```

We can take advantage of that second example to construct a function that
will swap two values:

```clojure
(defn swap
  [items i j]
  (assoc items i (items j) j (items i)))
```

This function will break on values of `i` and `j` that are out of the bounds
of `items`, but dealing with that is left as an exercise to the reader.

[source](http://stackoverflow.com/questions/5979538/what-is-the-idiomatic-way-to-swap-two-elements-in-a-vector)

# Try A Clojure Project In The REPL 

The `lein-try` plugin is a tool that makes it easy to quickly try out a
clojure project in the lein repl. Given the name and version of a clojure
project, it will drop you into a repl with that project loaded in. This is a
great way to get the feel for the features of an unfamiliar clojure library
before dropping it in as a dependency to your own project.

First, add the plugin to your `~/.lein/profiles.clj` file by including the
following line in the `:plugins` vector:

```
[lein-try "0.4.3"]
```

Then simply invoke the plugin with whatever project you want to try:

```
$ lein try automat
```

And to include a specific version number:

```
$ lein try automat "0.1.3"
```

# Type of Anything 

You can get the type of anything with the `type` function. Because Clojure
is built on Java, many of these types may be types you recognize form Java.

Boot up the repl to try some of these out:

```clojure
> (type 5)
; java.lang.Long
> (type 5.2)
; java.lang.Double
> (type 5/4)
; clojure.lang.Ratio
> (type (int 2))
; java.lang.Integer
> (type "hello, world!")
; java.lang.String
> (type [1 2 3])
; clojure.lang.PersistentVector
> (type '(1 2 3))
; clojure.lang.PersistentList
```

[source](https://aphyr.com/posts/302-clojure-from-the-ground-up-basic-types)

# When Overflow Is Desired 

If you try to add two `MAX_VALUE` Longs, Clojure is kind enough to warn you.

```clojure
> (+ Long/MAX_VALUE Long/MAX_VALUE)
ArithmeticException integer overflow  clojure.lang.Numbers.throwIntOverflow (Numbers.java:1424)
```

However, when overflow is desired, you can use the *unchecked* operators
(e.g. `unchecked-add`).

```clojure
> (unchecked-add Long/MAX_VALUE Long/MAX_VALUE)
-2
```

See also `unchecked-subtract` and `unchecked-multiply`.

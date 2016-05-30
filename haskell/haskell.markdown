# Dependency graphs with Stack and Graphviz
We can generate dependency graphs for a package or your own project with any
of the following commands,

* stack dot | dot -Tpng -o wreq.png
* stack dot --external | dot -Tpng -o wreq.png
* stack dot --no-include-base --external --depth 1 | dot -Tpng -o wreq.png
* stack dot --external --prune base,lens,wreq-examples,http-client,aeson,tls,http-client-tls,exceptions | dot -Tpng -o wreq_pruned.png

If graphviz is not available, install with,
```
sudo apt-get install graphviz
```
on ubuntu

# Haskell Mistakes - Jasper Van der Jeugt
This is a summary of pointers presented in the following video
[source](https://www.youtube.com/watch?v=S3WGPuqfBLg)

* Focus on modules as units
* Exposed interface of a module should be clean
* Documentation for the module at the top
* Some tests for the module
* Always compile using -Wall during development. You can use -Werror for Prod.
* Try to avoid wildcard pattern matches like,
  ```
  fireSquad :: IsMurderer -> Bool
  fireSquad = \case
    Innocent -> False
    _        -> True
  ```
* Remember that exceptions in Haskell are different. Reasoning about exceptions
  are complicated by laziness and async code. One way to deal with this is to
  use pure code as much as possible. Instead of exceptions use
  `Control.Monad.Except`
* Use a testing library to test your code
* Use the Text library instead of the String type unless you're writing a very
  simple program.
* Make all your code pure if possible.
* Don't try to prematurely abstract or prematurely optimize. Waiting awhile
  until a pattern emerges is better way to come up with abstractions

# Extensions
* **Extensions to use in every file**
  * `{-# LANGUAGE OverloadedLists #-}`
    > Allows string literals to be inferred as ByteString, Text, String or any
      string like Type
  * `{-# LANGUAGE OverloadedStrings #-}`
    > Allows string literals to be inferred as ByteString, Text, String or any
      string like Type
  * `{-# LANGUAGE ScopedTypeVariables #-}`
    > Allows reusing type variables from the parent signature in the type
      signatures for any local definition or in the annotations of any local
      binding
  * `{-# LANGUAGE GeneralizedNewtypeDeriving #-}`
  * `{-# LANGUAGE NoImplicitPrelude #-}` (Use classy-prelude)

* **Extensions that extend the Type system**
  * `{-# LANGUAGE GADTs #-}`
  * `{-# LANGUAGE DataKinds #-}`
    > GHC automatically promotes every suitable datatype to be a kind, and its
      (value) constructors to be type constructors.
  * `{-# LANGUAGE FlexibleInstances #-}`
    > Without it you cannot do `instance TypeClass (Maybe Int) where`
  * `{-# LANGUAGE EmptyDataDecls #-}`
    > Allows defining Types with no Constructors

* **Extensions that improve readability**
  * `{-# LANGUAGE LambdaCase #-}`
  * `{-# LANGUAGE ViewPatterns #-}`
  * `{-# LANGUAGE PatternGuards #-}`
  * `{-# LANGUAGE MultiWayIf #-}`
  * `{-# LANGUAGE TupleSections #-}`
  * `{-# LANGUAGE BinaryLiterals #-}`
  * `{-# LANGUAGE InstanceSigs #-}`
    > Allows one to write type signatures for the methods in an instance
      definition

* **Extensions that can automatically derive classes**
  * `{-# LANGUAGE GeneralizedNewtypeDeriving #-}`
  * `{-# LANGUAGE DeriveFunctor #-}`
  * `{-# LANGUAGE DeriveFoldable #-}`
  * `{-# LANGUAGE DeriveTraversable #-}`
  * `{-# LANGUAGE RecordWildCards #-}`
  * `{-# LANGUAGE DeriveGeneric #-}`
    > Based on GHC.Generics for deriving classes not in Base. Can be used by
      **import GHC.Generics**
  * `{-# LANGUAGE DeriveAnyClass #-}`
  * `{-# LANGUAGE DeriveDataTypeable #-}`
  * `{-# LANGUAGE StandaloneDeriving #-}`
    > Using this allows one to derive instances for types defined in other
      modules with **instance Show a => Show (List a)**

[Other Extensions](https://www.reddit.com/r/haskell/comments/2z248l/language_extensions/)

[Extensions that are good, bad and ugly](https://stackoverflow.com/questions/10845179/which-haskell-ghc-extensions-should-users-use-avoid?lq=1)

# Learning Titbits and Resources
* Studying other peoples code teaches more than simply reading and practicing
  by yourself.
* A [defacto](https://github.com/chemouna/HaskellResources) collection of
  haskell resources
* Haskell related [news](haskellnews.org/grouped) from various sources like
  github, reddit, stackoverflow and a few others
* The haskell [ecosystem](https://github.com/Gabriel439/post-rfc/blob/master/sotu.md)
  by **Gabrial Gonzalez**. This documents the domains in which haskell is well suited
  and any available libraries for use in that domain. And this is his
  [blog](http://www.haskellforall.com/).
* The one and only [what I wish I knew about haskell](http://dev.stephendiehl.com/hask/#haskell)
  by **Stephen Diehl**
* A set of recommended [libraries](http://haskelliseasy.readthedocs.io/en/latest/)
  by **Chris Allen**

# Best practices
* With Haskell, because of its type system refactoring is a breeze. So when
  writing code, always get a quick, dirty version that simply works. And then
  refactor to hearts content. Quantity triumphs any initial attempts at quality.
* Beware of premature abstraction or optimization
* Prefer to import libraries as qualified. Typically this is just considered
  good practice for business logic libraries, it makes it easier to locate the
  source of symbol definitions. It is recommended to focus on the following two
  forms of import:
  ```
  import qualified Very.Special.Module as VSM
  import Another.Important.Module (printf, (<|>), )
  ```

# Values with Types with Kinds with Sorts
The first thing to remember here is that `*` is its own kind. It does not
mean `any type`. `*` is just another kind like `Symbol` or `Nat`. Although,
the kind `*` is represented with a character instead of a word. So the kind
`*` means the set of `all fully applied types`

In Haskell, all values have a type. For example,
```haskell
intnum :: Int
intnum = 3
```
The `Int` above has **Kind** `*`. In other words it is a fully applied type.

Now consider,
```haskell
data Either a b = Left a | Right b
```
`Either` has kind `Either -> * -> * -> *`. Meaning it takes two more types, to
return a fully applied type. So basically, `Either` is a type constructor that
takes two types as parameters and returns a new type. This is just a function at
the type level.

> With -XDataKinds, GHC automatically promotes every suitable datatype to be a
> kind, and its (value) constructors to be type constructors.

What that means is that, if we have a type like so,
```haskell
data Nat = Ze | Su Nat
```
then `DataKinds` gives us the following.

* `Nat` is promoted to a Kind
* `Ze` and `Su` are promoted to types

In other words, the `Ze` and `Su Nat` that existed at the value level have now
moved to the type level. And `Nat` which existed at the type level has now
moved to the kind level. And `Nat` can be used wherever kinds can be used.

In other words, all the suitable terms have been promoted one level higher.

# Monomorphic Restriction
For example, in the code
```haskell
f xs = (len, len)
  where len = genericLength xs
```
the type of genericLength is `Num a => [b] -> a`. So in the above case, since
`f` has no type signature, it is inferred as,
```haskell
f :: Num t => [b] -> (t, t)
```

But `genericLength` is polymorphic in it's return type. So based on the
implementation of `f`, it is perfectly valid to set the type signature of `f` to

```haskell
f :: (Num t1, Num t2) => [b] -> (t1, t2)
```

But when we do that, `len` will have to be computed twice since Haskell has no
type casting. So without the `NoMonomorphismRestriction`, the compiler will
throw the error,
```
Could not deduce (b ~ c)
from the context (Num b, Num c)
bound by the type signature for
      f :: (Num b, Num c) => [a] -> (b, c)
  at restriction.hs:(4,1)-(5,32)
  `b' is a rigid type variable bound by
      the type signature for f :: (Num b, Num c) => [a] -> (b, c)
      at restriction.hs:4:1
  `c' is a rigid type variable bound by
      the type signature for f :: (Num b, Num c) => [a] -> (b, c)
      at restriction.hs:4:1
```

To suppress the error, the programmer should be okay with the fact that `len`
may be computed twice. If he/she is okay with that, the
`NoMonomorphismRestriction` can be enabled with

```
* `{-# LANGUAGE NoMonomorphismRestriction #-}`
```

# Decoding Haskell Error Messages
When you see this message from the compiler,
```
Could not deduce (b ~ c)
```
it simply means that the compiler thinks that the type of `b` and `c` must be
the same. But from the context they are not.

When you see,
```
`b' is a rigid type variable bound by
      the type signature for f :: (Num b, Num c) => [a] -> (b, c)
```
it means that that the compiler is telling you that a concrete annotation was
provided by the programmer with the given type variable in the signature.

# Normal form (NF) and Weak head normal form (WHNF)

## Normal Form (NF)
If an expression is fully evaluated and contains no unevaluated thunks then it
is said to be in **normal form (NF)**. In other words there are no more sub-
expressions to evaluate

Example of expressions in NF
```haskell
42
(2, "hello")
\x -> (x + 1)
```

These are not in NF
```haskell
1 + 2           -- unevaluated. could be 3
(\x -> x + 1) 2 -- function is not applied
"he" ++ "llo"   -- function (++) not yet applied
(1 + 1, 2 + 2)  -- the tuple has unevaluated thunks
```

## Weak Head Normal Form (WHNF)
If an expression is only evaluated upto its outermost constructor regardless of
whether its sub-expressions are evaluated or not, then its said to be in **weak
 head normal form (WHNF)**. So all expressions in NF are by default also in
WHNF. But all WHNF expressions are not in NF since the constructors
subexpressions may be unevaluated.

Examples of expression in WHNF
```haskell
(1 + 1, 2 + 2)       -- outermost data constructor is (,)
\x -> 2 + 2          -- outermost part is lambda
'h' : ("e" ++ "llo") -- outermost data constructor (:)
```
## Differentiating WHNF and NF
**Only the outermost part of the expression is considered when determining if the
entire expression is in WHNF or NF**.

* if outermost expression is constructor or lambda then it is **WHNF**
* if it is a function application or a fully evaluated expression then it is
**NF**

# Making GHCi scale better and faster
GHCi becomes slow to reload once you hit 20, 30, 50 or 150 modules.

I recommend enabling `-fobject-code`. You can enable this by running

```bash
$ ghci -fobject-code
```
Or by setting it in the REPL:

```
:set -fobject-code
```

If you want it on all the time, you can put the above line in a .ghci file
either in your home directory or in the directory of your project.

This makes GHCi compile everything once and then use incremental recompilation
thereafter. You’ll find that you can load 100-module projects and work with
them just fine in this way.

After that, you may notice that loading some modules gives less type
information and general metadata than before. For that, re-enable
byte-compilation temporarily with

```bash
$ ghci -fbyte-code
```

Or by setting it in the REPL:

```
:set -fbyte-code
```
and `:load` that module again, you now have fast recompilation with complete
information, too.

Another tip is to use `-fno-code` to have really fast compilation. This also
works in combination with -fobject-code. But I’d recommend using this only for
type checking, not for getting useful warnings (like pattern match
inexhaustiveness).

So I would combine it with `-fobject-code` in the same way as above with
`-fbyte-code`, and then once you’re done hacking, re-enable `-fobject-code` and
rebuild everything.

[source](http://chrisdone.com/posts/making-ghci-fast)

# Custom GHCi options
GHCi options can either be set at the GHCi prompt with the `:set` command or can
also be set in a `.ghci` file and place in the project folder.

Using the `:set` command works only for that GHCi session.

Setting the custom GHCi options in the `.ghci` file makes it available whenever
GHCi is launched


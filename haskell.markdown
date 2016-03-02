# Extensions

* `{-# LANGUAGE OverloadedStrings #-}` - Allows string literals to be inferred
as ByteString, Text, String or any string like Type
* `{-# LANGUAGE EmptyDataDecls #-}` - Allows defining Types with no Constructors
* `{-# LANGUAGE ScopedTypeVariables #-}` - Allows reusing type variables from
the parent signature in the type signatures for any local definition or in the
annotations of any local binding
* `{-# LANGUAGE DeriveFunctor #-}` - Derives Functors for types of Kind
`(* -> *)`
* `{-# LANGUAGE DeriveFoldable #-}` - Derives Foldable
* `{-# LANGUAGE DeriveTraversable #-}` - Derives Traversable
* `{-# LANGUAGE DeriveGeneric #-}` - Based on GHC.Generics for deriving classes
 not in Base
* `{-# LANGUAGE FlexibleInstances #-}` - Without it you cannot do `instance
TypeClass (Maybe Int) where`
* `{-# LANGUAGE InstanceSigs #-}` - Allows one to write type signatures for
the methods in an instance definition
* `{-# LANGUAGE NoMonomorphismRestriction #-}`
* `{-# LANGUAGE DeriveAnyClass #-}` -
* `{-# LANGUAGE GADTs #-}` -

[Other Extensions](https://www.reddit.com/r/haskell/comments/2z248l/language_extensions/)

[Extensions that are good, bad and ugly](https://stackoverflow.com/questions/10845179/which-haskell-ghc-extensions-should-users-use-avoid?lq=1)

# Learning Titbits
* Studying other peoples code teaches more than simply reading and practicing
  by yourself.

# Best practices
* With Haskell, because of its type system refactoring is a breeze. So when
  writing code, always get a quick, dirty version that simply works. And then
  refactor to hearts content.
* Prefer to import libraries as qualified. Typically this is just considered
  good practice for business logic libraries, it makes it easier to locate the
  source of symbol definitions.

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


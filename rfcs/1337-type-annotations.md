---
feature: Type Annotations
start-date: 2021-12-28
author: _sivizius
co-authors: …
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Adding type-annotations to nix.

# Motivation
[motivation]: #motivation

Nix has nine basic types (`int`, `bool`, `string`, `path`, `null`, `set`, `list`, `lambda` and `float`).
While these types can be checked with the builtin-functions `typeOf`, `isInt`, … the only way to enforce a type is by using custom functions e.g.:
```
  let expect = type: value: if typeOf value == type then value else abort "Wrong Type!";
  in foo = expect "string" ( someRandomFunction 1337 )
```

But this is quite an overhead and the error-message is just a one-liner.
There is actually a way to check one specific type: the set with `foo = { ... } @ value: …` which generates some nice output.
In [nixpkgs/lib/](https://github.com/NixOS/nixpkgs/blob/master/lib/), some functions have a haskell-like type annotation as comment, e.g.
```
Type:
  collect ::
    (AttrSet -> Bool) -> AttrSet -> [x]
```
But as a comment, there is neither a type-check nor a enforced syntax.
The module-system of nixos uses [nixpkgs/lib/types.nix](https://github.com/NixOS/nixpkgs/blob/master/lib/types.nix),
  but this is for checking user-input (the configuration.nix) in a module,
    not really usefull to check my own code and modules.

Nix is not a compilated language, there is not even a run-time, just evaluation, and therefore no reason to have type-annotations mandatory.
I do not expect any performance boost by adding type-annotations, but it certainly helps debugging errors due to wrong types.
There are already type-checks in the builtin-functions, but catching type-errors only there results in long stack-traces or worse:
  Will actually evaluate but with unexpected results.
Catching type-errors early on might help.

# Detailed design
[design]: #detailed-design

1. I suggest to allow optional type annotations with the following syntax:
```
let
  foo: int = someFunctionICannotTrust 1337;
  numToString: int | float -> string = toString;
in {
  bar: string = intToString foo;
}
```
2. I suggest optional and experimental enforcment later
3. Implement generics:
```
abs: T -> T where T: int | float 
= value: if value < 0 then
    0 - value
  else
    value;
```
This is not the same as `int | float -> int | float`, because the input-type is preserved.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

Just a constant with a value:
```
foo: int = 1337;
```

Multiple allowed types (sum type):
```
foo: int | float = if maybeTrue then 1337 else 42.23;
```

Lambdas:
```
foo: int -> string -> string = len: text: substring 0 len text;
```

Higher Order Lambdas:
```
mapInt2Float: (int -> float) -> list = f: list: map f list;
```

Panic/Abort:
```
expectPositive: int -> int | never = value: if value > 0 then value else abort "Not positive";
```

Any:
```
anyToString: any -> string = toString;
```

Optional:
```
foo: int | null -> string = …;
fuu: int? -> string = …;
```

List of:
```
foo: [ int ] = [ 1337 ];
```

Sets:
```
foo: { a: int, b: string, ... } = { a = 1337; b = "23"; c = null; };
```

# Drawbacks
[drawbacks]: #drawbacks

Complexity and performance: This will add code and checking needs time.

# Alternatives
[alternatives]: #alternatives

Type-checking is possible entirely in nix with builtins, but this will make the code less readable and it has a higher performance-impact.

# Unresolved questions
[unresolved]: #unresolved-questions

Generics.

# Future work
[future]: #future-work

Further syntax-additions?

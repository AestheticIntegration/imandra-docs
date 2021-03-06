---
title: "Extracting OCaml modules with imandra-extract"
description: "How to use your verified code as an OCaml module"
kernel: imandra
slug: extracting-ocaml-modules-with-imandra-extract
---

# Extracting OCaml modules with `imandra-extract`

When developing code with Imandra, we write our programs in `.iml` (Imandra-ml),
or `.ire` (Imandra-reason) for Reason syntax.

Imandra is both a programming language and a logic, and as such it incorporates
arbitrary precision (integer and rational) arithmetic with their standard
mathematical semantics. This deviates from 'normal' OCaml in which `int` is of
fixed precision, for example.

When using Imandra, this arbitrary precision semantics is used both for code
execution and reasoning. For example, during an Imandra session, integer
literals are parsed as values of the arbitrary precision `Z.t` type, resp.,
rather than to standard (OCaml) machine integers.

Imandra also comes with its own prelude of functions that have been
pre-verified, `imandra-prelude`.

As well as reasoning about them, you can execute your `.iml` and `.ire` programs
directly with Imandra itself. However at some point you will probably want to
compile a verified module or group of modules into a larger program using an
OCaml toolchain (either the standard OCaml toolchain or Bucklescript). This is
where `imandra-extract` fits in, allowing you to extract `.ml` and `.re` files.

The `imandra-extract` binary comes bundled with your [Imandra
installation](./Installation.md).

## Example

```ocaml
let int_of_bit (a : bool) : Z.t =
 if a then 1 else 0

let rec int_of_bits (a : bool list) : Z.t =
  match a with
   | [] -> 0
   | a :: a' -> int_of_bit a + 2 * (int_of_bits a')
```

We can then use `imandra-extract` on this file:

```shell
imandra-extract my_module.iml -o my_module.ml
```

This creates an `.ml` file similar to the following:

```ocaml
(* generated by imandra-extract from "my_module.iml" *)

open Imandra_prelude;;

(* start generated code here *)



#1 "test.iml"
let int_of_bit (a : bool) =
                (if a then Z.of_string "1" else Z.of_string "0" : Z.t)

#4 "test.iml"
let rec int_of_bits (a : bool list) =
                (match a with
                 | [] -> Z.of_string "0"
                 | a::a' ->
                     (int_of_bit a) + ((Z.of_string "2") * (int_of_bits a')) :
                Z.t)

```

As you can see, the literals are wrapped with the appropriate `Z` conversion
functions, and the `Imandra_prelude` module has been opened (which is opened
implicitly when running Imandra).

As this is now just a regular `.ml` program, it can be compiled alongside other
OCaml modules - you will however need to install the extracted `imandra-prelude`
as a dependency in your project. View the project and README for
`imandra-prelude` on GitHub: https://github.com/aestheticIntegration/imandra-prelude.

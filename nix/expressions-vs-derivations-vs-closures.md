TODO: Clean up.

      Different  versions of  this  text are  at
      different places, and it is getting out of
      hand. Reconcile them, and  put them on the
      `nix-tome` repo.

      + https://medium.com/scientific-breakthrough-of-the-afternoon/closure-vs-derivation-in-the-nix-package-manager-ec0eccc53407

      + https://stackoverflow.com/questions/58243554/what-is-a-nix-expression-in-regard-to-nix-package-management/58243555#58243555

      + https://stackoverflow.com/questions/31490262/what-is-the-purpose-of-nix-instantiate-what-is-a-store-derivation/58243537#58243537

      + ... and here

> TODO: closure summary
>
> "_. It is a representation of the cryptographic hash
> of all inputs involved in building the component._"
>
> Read  the beginning  of  the  thesis again,  because
> closures  weren't  even  mentioned in  this  section
> (which is 2.1 The Nix store, page 19).

# "expression" vs "derivation" vs "closure" in the Nix package manager

Found the clearest definitions in
[Eelco Dolstra](https://nixos.org/~eelco/)'s
(the creator of Nix and NixOS) PhD thesis. It can be
downloaded from
[his list of publications](https://nixos.org/~eelco/pubs/),
but here's the direct link:
[The Purely Functional Software Deployment Model (2006)](https://nixos.org/~eelco/pubs/phd-thesis.pdf).

Emphases all mine.

## Summary

-------

A  **Nix  expression**  is  a  set  of  instructions
describing  how   to  build  a   software  component
(package, project, application,  etc.) using the Nix
purely functional language.

<sup> <a href="https://github.com/Gabriel439/haskell-nix/pull/39#issuecomment-357790605">To quote Gabriel Gonzalez</a>:
"_You   can    think   of   a   derivation    as   a
language-agnostic recipe for  how to build something
(such as a Haskell package)._"
</sup>

Nix   expressions    are   also    commonly   called
**derivations**     (as    in     _Nix    derivation
expressions_), but

```text
*------------------------------------------------------*
|                                                      |
|       STORE DERIVATION =/= NIX EXPRESSION            |
|                                                      |
*------------------------------------------------------*
|                                                      |
| NIX EXPRESSION == function                           |
|                                                      |
| ( Describes how to build a component. That is, how ) |
| ( to  compose  its  input parameters, which can be ) |
| ( other components as well.                        ) |
|                                                      |
| STORE DERIVATION == function application             |
|                                                      |
| ( Call a  Nix  expression with concrete arguments. ) |
| ( Corollary: a single Nix  expression  can produce ) |
| ( different derivations depending on the inputs.   ) |
|                                                      |
*------------------------------------------------------*
```

The purpose of Nix expressions is to produce a
[**store derivation**](link here)
that  can  be  built into a component  (executable,
library, etc.).

From  section "5.4  Translating  Nix expressions  to
store derivations":

> The normal form [of a Nix expression] should
> be
>
>   + a call to `derivation`, or
>
>   + a nested  structure  of lists and
>     attribute sets that contain calls
>     to `derivation`.
>
> In any case, these derivation  Nix expressions
> are subsequently translated to store derivations.


![Flow diagram of derivation creation](./figure_2-12.png)

> QUESTION 2019-09-29_0800
>
> Are   there   any   Nix   expressions   that   don't
> ultimately  result in  derivations?  There are  ones
> that  only  indirectly  produce  derivations  (e.g.,
> `all-packages.nix`),  but the  outcome is  the same.
> Plus that is the only purpose of Nix expression.

## Nix expressions

-------

A **Nix  expression** is a function  that prescribes
build action(s).

-------

From section "_2.2 Nix expression_":

> Nix components are built from _Nix expressions_. The
> language  of  Nix  expressions is  a  simple  purely
> functional language used  to describe components and
> the compositions thereof.

For example, the Nix expression to build the `Hello`
package in Figure 2.6:

> Figure 2.6
>
> ```nix
> {stdenv, fetchurl, perl}:
>
> stdenv.mkDerivation {
>   name = "hello-2.1.1";
>   builder = ./builder.sh;
>   src = fetchurl {
>     url = http://ftp.gnu.org/pub/gnu/hello/hello-2.1.1.tar.gz;
>     md5 = "70c9ccf9fac07f762c24f2df2290784d";
>   };
>   inherit perl;
> }
> ```

Resources on the Nix language:

  + [Nix Manual, Chapter 15. Nix Expression Language](https://nixos.org/nix/manual/#ch-expression-language)
  + [NixOS Wiki, Nix Expression Language](https://nixos.wiki/wiki/Nix_Expression_Language)
  + [Nix Pills, Chapter 4. The Basics of the Language](https://nixos.org/nixos/nix-pills/basics-of-language.html)
  + [Nix by example, Part 1: The Nix expression language](https://medium.com/@MrJamesFisher/nix-by-example-a0063a1a4c55)
  + [What is the syntax of a valid identifier in the Nix language?](https://stackoverflow.com/questions/56198420/what-is-the-syntax-of-a-valid-identifier-in-the-nix-language) (with link to the Nix language [lexer](https://github.com/NixOS/nix/blob/6a5bf9b1438ed0b721657568fbb3a7c0b829e89e/src/libexpr/lexer.l))

```text
*--------------------------------------------------*
| What is a component?                             |
| ====================                             |
|                                                  |
| A package, application, development environment, |
| software library, etc.                           |
|                                                  |
| More formally from "3.1 What is a component?":   |
|                                                  |
| >  A software component is                       |
| >                                                |
| >         *------------------------------------* |
| >     1.  | a software artifact that is subject| |
| >         | to automatic composition           | |
| >         *------------------------------------* |
| >                                                |
| >         It can require, and be required by,    |
| >         other components.                      |
| >                                                |
| >         *----------------------*               |
| >     2.  | a unit of deployment |               |
| >         *----------------------*               |
|                                                  |
| It's worth to read the entire section.           |
*--------------------------------------------------*
```

## Derivations

-------

A  **store derivation**  is  a  Nix  expression  with  all
variability removed (i.e.,  a function with concrete
inputs) and _tranlated_  into an alternative format.
This   intermediate  representation   "_describes  a
single,  static,  constant build  action_"  (section
5.4).

```text
  NIX EXPRESSION  +  INPUTS  =  DERIVATION
    (function)       (args)
```

Derivations can be _built_ into components, that is, the encoded
function with its arguments is evaluated.

-------


<sup>Nix expressions usually translate to a graph of store derivations. (5.5. Building store derivations)</sup>


![Flow diagram of derivation creation](./figure_2-12.png)

Its name comes from "_2.2 Nix expressions_":

> The result  of the  function [i.e.,  Nix expression]
> is  a  **derivation**.  This   is  Nix-speak  for  a
> **component  build  action**,  which  _derives_  the
> component from its inputs.

On page 26, the thesis describes a Nix expression as
a  _family  of  build  actions_, in  contrast  to  a
derivation that is "exactly one build action".

```text
                              ARG_1, ..., ARG_N

                        | ---(aaa, ...)---> DERIVATION_1
        NIX EXPRESSION  | ---(bbb, ...)---> DERIVATION_2
                        |       :
           function(    |       :
             param_1,   |       :
             ...,       |       :
             param_N    |       :
             )          |       :
                        | ---(zzz, ...)---> DERIVATION_N
```

The derivations  above could  be producing  the same
application but  would build it with different configuration
options  for example.  (See APT  packages `vim-nox`,
`vim-gtk`, `vim-gtk3`, `vim-tiny`, etc.)

```text
*--------------------------------------------------*
| Why are derivations needed?                      |
| ===========================                      |
|                                                  |
| Section  "_2.4 Store  derivations_" has  all the |
| details, but here's the gist:                    |
|                                                  |
| > Nix  expressions are  not built  directly;     |
| > rather,  they are  translated to  the more     |
| > primitive  language of  store derivations,     |
| > which   encode   single  component   build     |
| > actions.  This  is  analogous to  the  way     |
| > that  compilers  generally   do  the  bulk     |
| > of  their  work  on  simpler  intermediate     |
| > representations   of    the   code   being     |
| > compiled,  rather  than   on  a  fullblown     |
| > language with all its complexities.            |
|                                                  |
*--------------------------------------------------*
```

For example, the Nix expression to build the `Hello`
package in Figure 2.6,

> Figure 2.6
>
> ```nix
> {stdenv, fetchurl, perl}:
>
> stdenv.mkDerivation {
>   name = "hello-2.1.1";
>   builder = ./builder.sh;
>   src = fetchurl {
>     url = http://ftp.gnu.org/pub/gnu/hello/hello-2.1.1.tar.gz;
>     md5 = "70c9ccf9fac07f762c24f2df2290784d";
>   };
>   inherit perl;
> }
> ```

will  result in  an  intermediate representation  of
something similar to in Figure 2.13:

> Figure 2.13 Store derivation
>
> ```text
> { output = "/nix/store/bwacc7a5c5n3...-hello-2.1.1" 25
> , inputDrvs = { 26
>     "/nix/store/7mwh9alhscz7...-bash-3.0.drv",
>     "/nix/store/fi8m2vldnrxq...-hello-2.1.1.tar.gz.drv",
>     "/nix/store/khllx1q519r3...-stdenv-linux.drv",
>     "/nix/store/mjdfbi6dcyz7...-perl-5.8.6.drv" 27 }
>   }
> , inputSrcs = {"/nix/store/d74lr8jfsvdh...-builder.sh"} 28
> , system = "i686-linux" 29
> , builder = "/nix/store/3nca8lmpr8gg...-bash-3.0/bin/sh" 30
> , args = ["-e","/nix/store/d74lr8jfsvdh...-builder.sh"] 31
> , envVars = { 32
>     ("builder","/nix/store/3nca8lmpr8gg...-bash-3.0/bin/sh"),
>     ("name","hello-2.1.1"),
>     ("out","/nix/store/bwacc7a5c5n3...-hello-2.1.1"),
>     ("perl","/nix/store/h87pfv8klr4p...-perl-5.8.6"), 33
>     ("src","/nix/store/h6gq0lmj9lkg...-hello-2.1.1.tar.gz"),
>     ("stdenv","/nix/store/hhxbaln5n11c...-stdenv-linux"),
>     ("system","i686-linux"),
>     ("gtk","/store/8yzprq56x5fa...-gtk+-2.6.6"),
>   }
> }
> ```

```text
*--------------------------------------------------*
| From section  "5.4. Translating  Nix expressions |
| to store derivations":                           |
|                                                  |
| > The abstract  syntax of  store derivations     |
| > is shown  in Figure 5.5 in  a Haskell-like     |
| > [135] syntax (see  Section 1.7). The store     |
| > derivation example shown in Figure 2.13 is     |
| > a value of this data type.                     |
| >                                                |
| > Figure  5.5.:  Abstract  syntax  of  store     |
| >                derivations                     |
| >                                                |
| >   data StoreDrv = StoreDrv {                   |
| >     output : Path,                             |
| >     outputHash : String,                       |
| >     outputHashAlgo : String,                   |
| >     inputDrvs : [Path],                        |
| >     inputSrcs : [Path],                        |
| >     system : String,                           |
| >     builder : Path,                            |
| >     args : [String],                           |
| >     envVars : [(String,String)]                |
| >   }                                            |
|                                                  |
*--------------------------------------------------*
```

## (To cull and format)
>
> > We     perform    a     derivation    by     calling
> > `stdenv.mkDerivation`. `mkDerivation`  is a function
> > provided  by stdenv  that  builds  a component  from
> > a  set  of  attributes2.  The  attributes  given  to
> > stdenv.mkDerivation are  the concrete inputs  to the
> > build action.
>
> -------
>
> ### Derivations and attribute sets
>
> From "_2.2 Nix expressions_":
>
> > An attribute set  is just a list  of key/value pairs
> > where  each value  is an  arbitrary Nix  expression.
> > They  take the  general form  {name1 =  expr1 ;  ...
> > namen = exprn ;}.
>
> -------
>
> ```text
>     *-------------------------------------*
>     |  STEP 3. Composition                |
>     |-------------------------------------|
>     |                                     |
>     | (STEP 1 Nix expression) { args } --------O----> derivation_1
>     | (STEP 1 Nix expression) { args } --------U----> derivation_2
>     | (STEP 1 Nix expression) { args } --------T----> derivation_3
>     | (STEP 1 Nix expression) { args } --------P----> derivation_4
>     |        :                            |    U
>     | (STEP 1 Nix expression  { args } --------T----> derivation_N
>     |                                     |
>     *-------------------------------------*
> ```
>
> ## Closures
>
> The goal of complete deployment: safe deployment requires that there are no missing dependencies. This means that we need to deploy closures of components under the "depends-on" relation. That is, when we deploy (i.e., copy) a component X to a client machine, and X depends on Y, then we also need to deploy Y to the client machine.
> Derivations
> Nix expressions are not built directly; rather, they are translated to the more primitive language of store derivations, which encode single component build actions. This is analogous to the way that compilers generally do the bulk of their work on simpler intermediate representations of the code being compiled, rather than on a full-blown language with all its complexities. Store derivations are placed in the Nix store, and as such have a store path too. The advantage of this two-level build process is that the paths of store derivations give us a unique identification of objects of source deployment, just as paths of binary components uniquely identify objects of binary deployment.
> Figure 2.12 outlines this two-stage build process: Nix expressions are first translated to store derivations that live in the Nix store and that each describe a single build action with all variability removed. These store derivations can then be built, which results in derivation outputs that also live in the Nix store.
> The above quote is directly preceded with the reasons on why derivations are necessery in section 2.4 Store derivations.
>
> Jumping back to "_2.2 Nix expression_", it continues
> with an example  on how Nix expressions  are used:
>
> > Generally,  to deploy  a component  [using Nix]  one
> > performs the following three steps:
> >
> >   1. Write  a Nix  expression for  the component  (Figure
> >      2.6),  which describes  all the  inputs involved  in
> >      building the component,  such as dependencies (other
> >      components required by  the component), sources, and
> >      so on.
> >
> >      **The result of this function is a _derivation_.**
> >
> >   2. Write  a builder  (Figure 2.7)  - typically  a shell
> >      script - that actually builds the component from the
> >      inputs.
> >
> >   3. Create   a  composition   (Figure   2.8).  The   Nix
> >      expression written in the  first step is a function:
> >      in order to build  a concrete component, it requires
> >      that the dependencies are  filled in. To compose the
> >      component  with  its  dependencies,  we  must  write
> >      another Nix expression that  calls the function with
> >      appropriate arguments.
>
> The thesis  continues with  a detailed  example, but
> here's a quick overview on how these steps relate to
> each other:
>
> ```text
>     *--------------------------*
>     | STEP 3. Composition      |
>     |--------------------------|
>     |                          |
>     | Calling the functions    |  (Yeah, "explain it to me
>     | defined in Step 1 with   |   like I'm five".)
>     | concrete arguments.      |
>     |                          |
>     | e.g., all-packages.nix   |
>     |                          |
>     *--------------------------*
>                |
>              calls
>                |
>                V
>     *-----------------------------*
>     | STEP 1. Function definition |
>     *-----------------------------*
>                |
>              calls
>                |
>                V
>     *------------------------*
>     | STEP 2. Builder script |
>     *------------------------*
> ```

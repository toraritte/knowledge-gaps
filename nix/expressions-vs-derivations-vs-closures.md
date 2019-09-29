# "expression" vs "derivation" vs "closure" in the Nix package manager

Found the clearest definitions in
[Eelco Dolstra](https://medium.com/r/?url=https%3A%2F%2Fnixos.org%2F~eelco%2F)'s
(the creator of Nix and NixOS) PhD. It can be downloaded from
[his list of publications](https://medium.com/r/?url=https%3A%2F%2Fnixos.org%2F~eelco%2Fpubs%2F),
but here's the direct link:
[The Purely Functional Software Deployment Model (2006)](https://medium.com/r/?url=https%3A%2F%2Fnixos.org%2F~eelco%2Fpubs%2Fphd-thesis.pdf).

Emphases all mine.

## Summary

A **derivation**  is a  set of instructions,  in the
form  of  **Nix  expressions**,  specifying  how  to
build  a **software  component** (package,  project,
application, etc.).
[To quote Gabriel Gonzalez](https://github.com/Gabriel439/haskell-nix/pull/39#issuecomment-357790605):
"_You   can    think   of   a   derivation    as   a
language-agnostic recipe for  how to build something
(such as a Haskell package)._"

> TODO: closure summary

Technically (see next sections),

```text

  DERIVATION =/= NIX EXPRESSION
```

but Nix expressions produce derivations, therefore they are commonly called derivations as well.

> TODO 2019-09-29_0800
>
> Are   there   any   Nix   expressions   that   don't
> ultimately  result in  derivations?  There are  ones
> that  only  indirectly  produce  derivations  (e.g.,
> `all-packages.nix`),  but the  outcome is  the same.
> Plus that is the only purpose of Nix expression.

## Expressions

From section "_2.2 Nix expression_":

> Nix expressions  is  a  simple  **purely  functional
> language** used  to  describe  components   and  the
> compositions thereof.

-------

> ```text
> What are components?
> ====================
>
> The  definition  below  is  from  "_3.1  What  is  a
> component?_",  and it  is worth  to read  the entire
> section.
>
> >  A software component is
> >
> >         *----------------------------------*
> >     1.  | a software artifact that is      |
> >         | subject to automatic composition |
> >         *----------------------------------*
> >
> >         It can require, and be required by,
> >         other components.
> >
> >         *----------------------*
> >     2.  | a unit of deployment |
> >         *----------------------*
> ```
-------

Jumping back to "_2.2 Nix expression_", it continues
with an example  on how Nix expressions  are used by
the Nix package management system.

> Generally,  to deploy  a component  [using Nix]  one
> performs the following three steps:
>
>   1. Write  a Nix  expression for  the component  (Figure
>      2.6),  which describes  all the  inputs involved  in
>      building the component,  such as dependencies (other
>      components required by  the component), sources, and
>      so on.
>
>      **The result of this function is a _derivation_.**
>
>   2. Write  a builder  (Figure 2.7)  - typically  a shell
>      script - that actually builds the component from the
>      inputs.
>
>   3. Create   a  composition   (Figure   2.8).  The   Nix
>      expression written in the  first step is a function:
>      in order to build  a concrete component, it requires
>      that the dependencies are  filled in. To compose the
>      component  with  its  dependencies,  we  must  write
>      another Nix expression that  calls the function with
>      appropriate arguments.

The thesis  continues with  a detailed  example, but
here's a quick overview on how these steps relate to
each other:

```text
    *--------------------------*
    | STEP 3. Composition      |
    |--------------------------|
    |                          |
    | Calling the functions    |  (Yeah, "explain it to me
    | defined in Step 1 with   |   like I'm five".)
    | concrete arguments.      |
    |                          |
    | e.g., all-packages.nix   |
    |                          |
    *--------------------------*
               |
             calls
               |
               V
    *-----------------------------*
    | STEP 1. Function definition |
    *-----------------------------*
               |
             calls
               |
               V
    *------------------------*
    | STEP 2. Builder script |
    *------------------------*
```

## Derivations

From "_2.2 Nix expressions_":

> The  result   of  the   function  [in  STEP   1]  is
> a   **_derivation_**.  This   is  Nix-speak   for  a
> **component  build  action**,  which  _derives_  the
> component from its inputs.
>
> We     perform    a     derivation    by     calling
> `stdenv.mkDerivation`. `mkDerivation`  is a function
> provided  by stdenv  that  builds  a component  from
> a  set  of  attributes2.  The  attributes  given  to
> stdenv.mkDerivation are  the concrete inputs  to the
> build action.

-------

### Attribute sets

From "_2.2 Nix expressions_":

> An attribute set  is just a list  of key/value pairs
> where  each value  is an  arbitrary Nix  expression.
> They  take the  general form  {name1 =  expr1 ;  ...
> namen = exprn ;}.

-------

```text
    *-------------------------------------*
    |  STEP 3. Composition                |
    |-------------------------------------|
    |                                     |
    | (STEP 1 Nix expression) { args } --------O----> derivation_1
    | (STEP 1 Nix expression) { args } --------U----> derivation_2
    | (STEP 1 Nix expression) { args } --------T----> derivation_3
    | (STEP 1 Nix expression) { args } --------P----> derivation_4
    |        :                            |    U
    | (STEP 1 Nix expression  { args } --------T----> derivation_N
    *-------------------------------------*
```

## Closures

The goal of complete deployment: safe deployment requires that there are no missing dependencies. This means that we need to deploy closures of components under the "depends-on" relation. That is, when we deploy (i.e., copy) a component X to a client machine, and X depends on Y, then we also need to deploy Y to the client machine.
Derivations
Nix expressions are not built directly; rather, they are translated to the more primitive language of store derivations, which encode single component build actions. This is analogous to the way that compilers generally do the bulk of their work on simpler intermediate representations of the code being compiled, rather than on a full-blown language with all its complexities. Store derivations are placed in the Nix store, and as such have a store path too. The advantage of this two-level build process is that the paths of store derivations give us a unique identification of objects of source deployment, just as paths of binary components uniquely identify objects of binary deployment.
Figure 2.12 outlines this two-stage build process: Nix expressions are first translated to store derivations that live in the Nix store and that each describe a single build action with all variability removed. These store derivations can then be built, which results in derivation outputs that also live in the Nix store.
The above quote is directly preceded with the reasons on why derivations are necessery in section 2.4 Store derivations.

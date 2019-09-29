# "expression" vs "derivation" vs "closure" in the Nix package manager

Found the clearest definitions in
[Eelco Dolstra](https://medium.com/r/?url=https%3A%2F%2Fnixos.org%2F~eelco%2F)'s
(the creator of Nix and NixOS) PhD. It can be downloaded from
[his list of publications](https://medium.com/r/?url=https%3A%2F%2Fnixos.org%2F~eelco%2Fpubs%2F),
but here's the direct link:
[The Purely Functional Software Deployment Model (2006)](https://medium.com/r/?url=https%3A%2F%2Fnixos.org%2F~eelco%2Fpubs%2Fphd-thesis.pdf).

Emphases all mine.

## Expressions

From section "_2.2 Nix expression_":

> Nix expressions  is  a  simple  **purely  functional
> language** used  to  describe  components   and  the
> compositions thereof.

As  to what  compositions  are, there  is an  entire
section devoted  to it,  and here  is the  gist from
"_3.1 What is a component?_":

>  • A software component is  a software artifact that is
>    subject to automatic composition.
>
>    It can require, and be required by, other components.
>
>   • A software component is a unit of deployment.

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
    | e.g., all-packages.nix   |
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

Closures
The goal of complete deployment: safe deployment requires that there are no missing dependencies. This means that we need to deploy closures of components under the "depends-on" relation. That is, when we deploy (i.e., copy) a component X to a client machine, and X depends on Y, then we also need to deploy Y to the client machine.
Derivations
Nix expressions are not built directly; rather, they are translated to the more primitive language of store derivations, which encode single component build actions. This is analogous to the way that compilers generally do the bulk of their work on simpler intermediate representations of the code being compiled, rather than on a full-blown language with all its complexities. Store derivations are placed in the Nix store, and as such have a store path too. The advantage of this two-level build process is that the paths of store derivations give us a unique identification of objects of source deployment, just as paths of binary components uniquely identify objects of binary deployment.
Figure 2.12 outlines this two-stage build process: Nix expressions are first translated to store derivations that live in the Nix store and that each describe a single build action with all variability removed. These store derivations can then be built, which results in derivation outputs that also live in the Nix store.
The above quote is directly preceded with the reasons on why derivations are necessery in section 2.4 Store derivations.

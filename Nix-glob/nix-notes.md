This is a semi-structured mess of notes, and the clean up is in the plans.

### TODO

#### `nix.conf`, `config.nix`, and `configuration.nix`
The [`nix.conf`](https://nixos.org/nix/manual/#name-11) doc states that "*Nix reads settings from two configuration files*", but it depends also whether or not NixOS is the underlying OS. In that case, the config file is `/etc/nixos/configuration.nix`. The only place where this distinction is clarified is in [14.3.1. Tested using sandboxing](https://nixos.org/nixpkgs/manual/#submitting-changes-tested-with-sandbox) in the Nixpkgs manual. The third player, `config.nix`, is introduced by [6.6. Declarative Package Management](https://nixos.org/nixpkgs/manual/#sec-declarative-package-management). It gets more confusing because the main [Chapter 6. Global configuration](https://nixos.org/nixpkgs/manual/#chap-packageconfig) introduces `~/.config/nixpkgs/config.nix` that doesn't seem to work. Also shows examples using `alloBroken`, `allowUnfree` options, but where do they come from? Just from the NixOS options?

#### `buildEnv` and manifests

`buildEnv` is not documented, only description is in [6.6. Declarative Package Management](https://nixos.org/nixpkgs/manual/#sec-declarative-package-management). Good SO thread is [here](https://stackoverflow.com/questions/49590552/how-buildenv-builtin-function-works), with link to the source. The Nixpkgs source comment states that it was forked from Nix `buildenv`, and all changes should be contributed back. The comment is 12 years old, and the sources have diverged substantially.

Manifests are not covered at all. Maybe in the thesis? (UPDATE: it does cover them.) Hits from the manuals:

 + Nixpkgs manual: 1 occurence, may not even refer to Nix manifests.

 + NixOS manual: 0

 + NixOps manual: 0

 + Nix manual: 20 hits. The only hit not in release notes is "*https://stackoverflow.com/questions/49590552/how-buildenv-builtin-function-works*". The most recent release note occurence suggests that manifests are not used for certain operations, if I read that right. See [C.3. Release 2.0 (2018-02-22)](https://nixos.org/nix/manual/#ssec-relnotes-2.0). There's a link to a specific commit by Eelco Dolstra stating "*Manifests have been superseded by binary caches for years. This also gets rid of nix-pull, nix-generate-patches and bsdiff/bspatch.*" But I can still find `*manifest.nix` files in `/nix/store`. Read the thesis, and get into the source.

#### Basic Unix commands (e.g., `namei`)

Can be found in the `util-linux` package. Not documented anywhere, unless you know your way around linux.

#### `packageOverrides` confusion in Nixpkgs manual

First of all, [6.6. Declarative Package Management](https://nixos.org/nixpkgs/manual/#sec-declarative-package-management) is superseded by [Chapter 12. Overlays](https://nixos.org/nixpkgs/manual/#chap-overlays), see more on it [here](https://discourse.nixos.org/t/declarative-package-management-for-normal-users/1823/9?u=toraritte).

[6.5. Modify packages via packageOverrides](https://nixos.org/nixpkgs/manual/#sec-modify-via-packageOverrides) introduces `packageOverrides` by overriding a package (starting with `packageOverrides = pkgs: rec {`). 6.6 injects the `pkgs` attributes in the local scope (`packageOverrides = pkgs: with pkgs; {`), but still refers to `buildEnv` as `pkgs.buildEnv`. According to the `nix repl`, they are the same (see below), but is there a difference?

```text
nix-repl> pkgs = import <nixpkgs> {}

nix-repl> pkgs.buildEnv {}           
error: anonymous function at /nix/store/ywlfq2ns4a3fzb2ap74lvahmrg1p0lmk-nixos-19.03.172231.7b36963e7a7/nixos/pkgs/build-support/buildenv/default.nix:8:2 called without required argument 'name', at /nix/store/ywlfq2ns4a3fzb2ap74lvahmrg1p0lmk-nixos-19.03.172231.7b36963e7a7/nixos/lib/customisation.nix:69:12

nix-repl> pkgs.pkgs.buildEnv {}
error: anonymous function at /nix/store/ywlfq2ns4a3fzb2ap74lvahmrg1p0lmk-nixos-19.03.172231.7b36963e7a7/nixos/pkgs/build-support/buildenv/default.nix:8:2 called without required argument 'name', at /nix/store/ywlfq2ns4a3fzb2ap74lvahmrg1p0lmk-nixos-19.03.172231.7b36963e7a7/nixos/lib/customisation.nix:69:12
```

The [`buildEnv` source](https://github.com/NixOS/nixpkgs/blob/master/pkgs/build-support/buildenv/default.nix) is good to have around because it is not documented (see above).

 + `pathsToLink`
   Where does it link stuff? At this point find time to actually build a package, to know what the terminology means exactly.

 + `extraOutputsToInstall`
   See `pathsToLink` comment above.

 + `runCommand`
   Completely in the black on this one. The source shows that there is also a `runCommand` input argument, so just need to follow it I guess.
   Where is `"profile"` parameter coming from? Or is it just arbitrary because it gives a name to something?
#### Notes to [Chapter 12. Overlays](https://nixos.org/nixpkgs/manual/#chap-overlays)

> "*Overlays are used to add layers in **the fixed-point used by Nixpkgs** to compose the set of all packages.*"
Fixed-point as in "pinning Nixpkgs"? Don't even really understand what the latter means, hence the quotes.

#### heap

 + **Make references to claims.**
   For  example,  in  the  NixOS  manual  at  "*Chapter
   39.  Container  Management*",  it  reads  "*Warning:
   Currently,  NixOS   containers  are   not  perfectly
   isolated  from the  host  system.*"  There must be a
   Github issue,  a forum  discussion etc. then  to let
   people follow up on it.

 + `binary-caches`  and   `extra-binary-caches`  are  deprecated.
   Replace  this  with  their corresponding  replacements  (e.g.,
   `substituters`, `extra-subsituters` etc.)
   See [13.1. Serving a Nix store via HTTP](https://nixos.org/nix/manual/#ssec-binary-cache-substituter)

   Also introduce the concept of "substituter" in 13.1 because it
   is already in use in **13.3. Serving a Nix store via SSH**.

 + `mkIf` never mentioned in Nixpkgs manual even though it is used extensively
   E.g.,
   https://github.com/NixOS/nixpkgs/blob/release-19.03/nixos/modules/services/amqp/rabbitmq.nix
   https://github.com/NixOS/nixpkgs/blob/release-19.03/nixos/modules/services/misc/autorandr.nix

 + Why are vim plugins and the vim package at different pages in Nixpkgs?
   Related: `nixpkgs/pkgs/misc/vim-plugins/vim-utils.nix` has way
             better  description of  how  to  configure vim  with
             plugins in  NixOS than the official  manual... Found
             it by accident, and it would be nice to have it with
             the vim package (or in the manual).

 + NIXPKGS doc:

   Section  9  and 11  is  completely  messed up.  The
   former is called  "*Support for specific programming
   languages and  frameworks*", and has Vim  in it, and
   the title  of the  latter is "*Package  Notes*", but
   has Emacs  in it,  and Elm, which  should go  to the
   former.  The distinction  between  9 and  11 is  bad
   anyways, because  these are all  describing specific
   package.

   There  are sections,  such as  "*12. Overlays*"  and
   "*13. Coding Conventions*", that are intermingled in
   the package  specific chapters,  but provide  a more
   general  information. In  9, plenty  of descriptions
   mention overlays for example.

   **Overlays**
   *What problems does it solve?*
   *What is it exactly?* Couldn't  figure it out from the
   summary, and  it doesn't logically follow  any other
   sections that would give this away.

   *13. Coding Conventions* is a style guide.

 + NixOS manual:

   The  install section  is  good, but  then it  starts
   to  trail   off  in  "*II.   Configuration*"  toward
   (random|opinionated)  packages.  For example,  after
   the linux  kernel section,  there is  "Matrix" (what
   is  that?), Emacs  (why  not Vim,  or a  generalized
   "Editors"  section?).  It  finally  steers  back  to
   useful topics with "*III. Administration*".
   -> I  think   it  would  be  useful   to  continue  the
      installation  section with  the most  general setup,
      i.e. basic networking, user setup (because initially
      there's   just  root),   etc.   and  have   detailed
      descriptions later.
         |
         V
      Just got to "*Chapter 31. Profiles*". WHY ISN'T THIS
      RIGHT AFTER INSTALLATION (or somewhere near, such as
      "*Changing configuration*")?

   "*IV. Developmnet*"  should go to the  Nix manual in
   its entirety.  This is  a recurring theme  that many
   concepts are  introduced at  one place, but  are way
   better explained  with detailed examples  at another
   (almost random) location.

   "*Chapter  38. Cleaning  the  Nix  Store*" is  again
   hidden someplace  random. Why  not somewhere  in the
   vicinity of "*6. Package Management*"?

   The ordering (again) is weird in "*42. Writing NixOS
   Modules*": starts with "*42.1. Option Declarations*"
   and  "*42.1.1. Extensible  Option Types*",  and then
   goes into  "*42.2 Option  Types*" and  "*42.3 Option
   Definitions*". Why is 42.1.1 even nested under 42.1?

 * SYSTEMD

   Far from being clear, and there was this twitter thread too:
   https://twitter.com/toraritte/status/1118667135297802240
   (with a Stackoverflow question embedded in it)

   "*Chapter  33. Service  Management*" gives  the bare
   minimum  about  the topic,  but  it  still does  not
   explain how  to use packages that  also have systemd
   services written.  The closest that I  could find is
   in "*Chapter 18.  WeeChat*", explicitly showing what
   to do, and  "*Chapter 21. Emacs*". All  this is only
   in  the NixOS  manual,  nothing  in Nixpkgs  manual.
   (This  kinda  makes  sense  as  the  latter  is  not
   specifically tied to the  NixOS distro, but it would
   be  nice to  at least  have a  mention, because  the
   Nixpkgs repo has a  specific directory structure for
   systemd service files.

   Check out  "*Chapter 43. Building Specific  Parts of
   NixOS*",  there is  a good  description of  manually
   building systemd services.

   + The `/home/toraritte/clones/nixpkgs/nixos/modules/system/boot` folder in the nixpkgs repo has all the Nix lambdas that produce a systemd unit file. `systemd.nix` and `systemd-lib.nix` seem to be the workhorses, and `systemd-unit-options.nix` is testing options, and also sort of the only documentation there is.

   + in the NixOS config file, "services"`services.<pkg>.enable` lines refers to systemd service module packages (or nix-files) that belong to a specific package and they basically expose options for the given package. Whereas `systemd.services` requires one to fully flesh out a systemd target (?, i.e., service files etc., need to read up on systemd). Look for "systemd.services" in [NixOS options page](https://nixos.org/nixos/options.html).

   + in `nixos/modules/services/amqp/rabbitmq.nix:144` there is the line
     ```text
     142     # This is needed so we will have 'rabbitmqctl' in our PATH
     143     environment.systemPackages = [ cfg.package ];
     ```
     and `cfg.package` confused me. I assumed that it refers to `rabbitmq-server` itself, and as it turns out, it does:
     ```text
      18     services.rabbitmq = {
      (...)
      27       package = mkOption {
      28         default = pkgs.rabbitmq-server;
      29         type = types.package;
      30         defaultText = "pkgs.rabbitmq-server";
      31         description = ''
      32           Which rabbitmq package to use.
      33         '';
      34       };
      ```
      So at one point, this option will evaluate to `pkgs.rabbitmq-server`.

      To double-check:
      ```
      0 [12:34:10] nix repl
      Welcome to Nix version 2.2.2. Type :? for help.

      nix-repl> pkgs = import <nixpkgs> {}

      nix-repl> nixos = import <nixpkgs/nixos> {}

      nix-repl> pkgs.rabbitmq-server
      «derivation /nix/store/p3cdpk00zd4wvzwcz1ppgyqf8l52icbn-rabbitmq-server-3.7.11.drv»

      nix-repl> nixos.config.services.rabbitmq.package
      «derivation /nix/store/p3cdpk00zd4wvzwcz1ppgyqf8l52icbn-rabbitmq-server-3.7.11.drv»
      ```

### Ideas to improve the docs

 + **Show related commits to each manual.**

   It would be useful when (and what) the last change was.

   Some resources:
   [How to show git commits to a specific directory?](https://stackoverflow.com/questions/11950037/view-git-history-for-folder/19984452)

### Learned

 + Try out packages without installing anything:
   `nix-shell -p pkgname`
   Pulls in dependencies, drops into `nix-shell` prompt.

 + Working with **symlinks**

   `namei` (see output below)
   `readlink -f | -e`

### For [Chapter 1. About Nix](https://nixos.org/nix/manual/#ch-about-nix)

#### "You can have multiple versions or variants of a package installed at the same time."

How?

#### "Managing build environments" AKA `nix-shell`

Using `nix-shell`. Read `man nix-shell` and more on this topic.

#### `nix-shell '<nixpkgs>' -A pan`

QUESTION: Where does `'<nixpkgs>'` come from?

ANSWER: From [15.1. Values, subsection "Simple Values"](https://nixos.org/nix/manual/#idm140737317975776): "*Paths can also be specified between angle brackets, e.g. <nixpkgs>. This means that the directories listed in the environment variable NIX_PATH will be searched for the given file or directory name.*"

### For [Chapter 10. Profiles](https://nixos.org/nix/manual/#sec-profiles)

#### What is `/nix/var/nix/profiles/default` is for?

The command `nix-env --list-generations` lists the `per-user`
generations, and no clue what the `default` symlinks are for.
Cf. `namei ~/.nix-profile` and `namei /nix/var/nix/profiles/per-user/toraritte/profile`,
they both point to the same  location. I think the image is an
old one from Dolstra's thesis.

```text
 toraritte@lenixo  [~]
0 [14:09:31] namei /nix/var/nix/profiles/per-user/toraritte/profile
f: /nix/var/nix/profiles/per-user/toraritte/profile
 d /
 d nix
 d var
 d nix
 d profiles
 d per-user
 d toraritte
 l profile -> profile-14-link
   l profile-14-link -> /nix/store/kyq445ghy0l87sr2zym218s8x8icxaip-user-environment
     d /
     d nix
     d store
     d kyq445ghy0l87sr2zym218s8x8icxaip-user-environment

 toraritte@lenixo  [~]
0 [14:19:44] namei /nix/var/nix/profiles/default
f: /nix/var/nix/profiles/default
 d /
 d nix
 d var
 d nix
 d profiles
 l default -> default-16-link
   l default-16-link -> /nix/store/3vcwrnz51dv45wgjb1zfghpn7bhxm1d4-user-environment
     d /
     d nix
     d store
     d 3vcwrnz51dv45wgjb1zfghpn7bhxm1d4-user-environment

 toraritte@lenixo  [~]
0 [14:19:50] namei .nix-profile
f: .nix-profile
 l .nix-profile -> /nix/var/nix/profiles/per-user/toraritte/profile
   d /
   d nix
   d var
   d nix
   d profiles
   d per-user
   d toraritte
   l profile -> profile-14-link
     l profile-14-link -> /nix/store/kyq445ghy0l87sr2zym218s8x8icxaip-user-environment
       d /
       d nix
       d store
       d kyq445ghy0l87sr2zym218s8x8icxaip-user-environment
```

### [Chapter 11. Garbage Collection](https://nixos.org/nix/manual/#sec-garbage-collection)

#### "The defaults will ensure that all derivations that are not build-time dependencies of garbage collector roots will be collected but that all output paths that are not runtime dependencies will be collected."

"derivations that are not build-time dependencies of garbage collector roots"
-> derivations that are not dependencies to any profile-linked top-derivation?

"output paths that are not runtime dependencies"
-> artifacts after compilation that are needed during application runtime?

### [14.1. Expression Syntax](https://nixos.org/nix/manual/#sec-expression-syntax)

#### `stdenvNoCC`

`fetchurl/default.nix` starts with that. I assume that it is defined in `pkgs/stdenv`. See SO thread [How do I select GCC version in nix-shell?](https://stackoverflow.com/questions/50277775/how-do-i-select-gcc-version-in-nix-shell).

#### `mkDerivation`

Defined in `generic/make-derivation.nix`, as a wrapper around "_the builtin `derivation` function to produce derivations that use this stdenv and its shell._"

#### `derivation` inputs

 + `system`
 + `name`
 + `builder`
 + (optional) `args`
 + (optional) `outputs`
 + rest of the attributes are passed as environment variables according to the rules in [15.4. Derivations](https://nixos.org/nix/manual/#ssec-derivation), item 4 (the one with subitems).

### [15.4.1. Advanced Attributes](https://nixos.org/nix/manual/#sec-advanced-attributes)

QUESTIONs:
 + `allowedReferences`
 + well, almost all options

-------

QUESTIONS:

  + What is the "active Nix expression"?

    For most  of the  `nix-*` commands either  specify a
    Nix  expression to  use  with `-f`  or  the ones  in
    `~/.nix-defexpr/` are used by default.

  + When is the Nix store path not a derivation? What is it otherwise, a binary?

  + build, host, and target platforms? (in [3.3. Specifying dependencies](https://nixos.org/nixpkgs/manual/#ssec-stdenv-dependencies))

    3.3 also links to  [Chapter 5. Cross-compilation](https://nixos.org/nixpkgs/manual/#chap-cross) that
    tries to further elucidate the meaning of the terms,
    but the distinction between host and target is still
    not  clear  for me.  Chapter  5  also links  to  GNU
    Autoconf docs,  but that wasn't clear  until I found
    the SO thread and a book snippet below.

    Correction:  Chapter 5  is pretty  good. Leaving
                 the sources below though, just in case.

    From GNU Autoconf's [6.1 Configure Terms and History](https://gcc.gnu.org/onlinedocs/gccint/Configure-Terms.html):
    > There are three system names that the build knows about:
    > + the machine you are building on (_build_),
    > + the machine that you are building for (_host_), and
    > + the machine that GCC will produce code for (_target_).

    This from [this SO thread](https://stackoverflow.com/questions/5139403/whats-the-difference-of-configure-option-build-host-and-target):
    > Argument `--target`  makes sense only  when building
    > compiler (e.g. GCC).

    with [this](http://mdcc.cx/pub/autobook/autobook-latest/html/autobook_259.html) (even though it is just rephrasing the official docs) helped:
    > When building cross compilation tools, there are two
    > different systems involved: the  system on which the
    > tools will run,  and the system for  which the tools
    > will generate  code. The  system on which  the tools
    > will run is  called the host system.  The system for
    > which the  tools generate code is  called the target
    > system.

  + **Sliding Window Principle**

    > A build  time dependency,  however, implies  a shift
    > in  platforms  between  the  depending  package  and
    > the  depended-on package.  The  meaning  of a  build
    > time  dependency  is  that to  build  the  depending
    > package we need to be  able to run the depended-on's
    > package. The  depending package's build  platform is
    > therefore  equal to  the depended-on  package's host
    > platform. Analogously, the  depending package's host
    > platform  is  equal  to  the  depended-on  package's
    > target platform.

    For example, paraphrasing the official example,

    ```text
                        (BUILD, HOST, TARGET)
    DEPENDING PACKAGE   (foo,   bar,   bar  )
                             \      \
                              \      \
                               V      V
    DEPENDED-ON PKGS    (foo,   foo,   bar  )
                             \      \
                              \      \
                               V      V
    DEPENDED-ON'S       (foo,   foo,   foo   )
    DEPENDENCIES
    ```

    That  is,  the  depending package's  build  platform
    is  determined  by  the depended-on  packages'  host
    platform (e.g., because the depended-on package is a
    compiler that  runs on that platform),  and its host
    platform is  determined by the  depended-on packages
    target  platform  (e.g.,   because  the  depended-on
    package  is a  compiler only  being able  to produce
    code running in that platform).

  + "Variables specifying dependencies" in
    [3.3. Specifying dependencies](https://nixos.org/nixpkgs/manual/#ssec-stdenv-dependencies)

    **depsBuildBuild**
      > A  list  of  dependencies   whose  host  and  target
      > platforms are  the new derivation's  build platform.
      > This means a  -1 host and -1 target  offset from the
      > new derivation's  platforms. These are  programs and
      > libraries used  at build time that  produce programs
      > and libraries also used at build time.

      Is the offset (-1, -1) because of this?

      ```text
                          (BUILD, HOST, TARGET)
      DEPENDING PACKAGE   (foo,   bar,   bar  )
                               \   :  \   :
                                \  :   \  :
                                 V :    V :
      DEPENDED-ON PKGS    (foo,   f:o,   b:r  )
                               \   :  \   :
                                \  :   \  :
                                 V V    V V
      DEPENDED-ON'S       (foo,   foo,   foo   )
      DEPENDENCIES
      ```

    **nativeBuildInputs**

      Offset (-1, 0)?

      ```text
                          (BUILD, HOST, TARGET)
      DEPENDING PACKAGE   (foo,   bar,   bar  )
                               \   :  \   :
                                \  :   \  :
                                 V V    V V
      DEPENDED-ON PKGS    (foo,   foo,   bar  )
                               \      \
                                \      \
                                 V      V
      DEPENDED-ON'S       (foo,   foo,   foo   )
      DEPENDENCIES
      ```

    **depsBuildTarget**

      Offset (-1, 0)?

      ```text
                          (BUILD, HOST, TARGET)
      DEPENDING PACKAGE   (foo,   bar,   bar  )
                               \   :  \   :
                                \  :   \  :
                                 V V    V V
      DEPENDED-ON PKGS    (foo,   foo,   bar  )
                               \      \
                                \      \
                                 V      V
      DEPENDED-ON'S       (foo,   foo,   foo   )
      DEPENDENCIES
      ```

  + What is **dependency propagation** exactly? "_The extension of PATH with dependencies_"?

    [3.3. Specifying dependencies](https://nixos.org/nixpkgs/manual/#ssec-stdenv-dependencies)
    quickly gets out of hand, even after reading
    [Chapter 5. Cross-compilation](https://nixos.org/nixpkgs/manual/#chap-cross).

  + **Dependency propagation offsets**

    Kind of in the middle of
    [3.3. Specifying dependencies](https://nixos.org/nixpkgs/manual/#ssec-stdenv-dependencies),
    after introducing(?) dependency propagation.

    > The exact  rules for  dependency propagation  can be
    > given by  assigning to each dependency  two integers
    > based  one how  its  host and  target platforms  are
    > offset from the depending derivation's platforms.

---

This is not a project to belittle efforts of others who contribute to the official docs, or who have created their own resources.

---

FQDN issue (https://github.com/NixOS/nixpkgs/issues/10183)
==========================================================

Things tried to resolve this on a fresh NixOS 19.03 install.

1. Add

  ```nix
    networking.hostName = "pat";
    networking.domain = "test.tld";
  ```
  then
  ```text
  # nixos-rebuild switch
  # reboot
  ```

  After reboot:
  ```text
  # hostname --fqdn
  pat
  # hostname --domain

  # domainname -v
  getdomainname()='test.tld'
  test.tld
  # dnsdomainname -v
  gethostname()='pat'
  Resolving 'pat' ...
  Result: h_name='pat'
  Result: h_addr_list='fe80::....::....::....::....'
  # cat /etc/hosts
  127.0.0.1 localhost
  127.0.1.1 pat
  ::1 localhost
  ```

2. Changes according to issue comment https://github.com/NixOS/nixpkgs/issues/10183#issue-109493770 , i.e., adding

  ```nix
  networking.search = [ "test.tld" ];
  ```
  then
  ```text
  # nixos-rebuild switch
  # reboot
  ```

  After reboot: everything the same as in 1. above.

### Odd, not documented (or too obvious) stuff found

Working inside company network with its own DNS and own domain. Without `networking.search = []`, the only entry in `/etc/resolv.conf` is `options edns0`. The system FQDN resolves nicely though, even pinging with just the host part (i.e., `pat`). With the option there, it nicely puts in the local domain and miscellaneous info.

# [1.4.  Being purely functional](https://nixos.org/nixos/nix-pills/why-you-should-give-it-a-try.html#idm140737316696912)

Links to understand the `ldd` code sample (the first three are the more introductory pieces):
+ http://man7.org/linux/man-pages/man1/ldd.1.html
+ https://superuser.com/questions/71404/what-is-an-so-file
+ http://www.yolinux.com/TUTORIALS/LibraryArchives-StaticAndDynamic.html
+ https://unix.stackexchange.com/questions/190719/how-to-use-libraries-installed-by-nix-at-run-time
+ https://nixos.org/patchelf.html
+ https://www.reddit.com/r/NixOS/comments/4e9r8x/what_goes_in_lb_library_path/

# [2.1. Installation](https://nixos.org/nixos/nix-pills/install-on-your-running-system.html#idm140737316668000)

The install fails on Debian 9 Stretch, but issuing `sudo sysctl kernel.unprivileged_userns_clone=1` according to [`NixOS/nix` issue #2633](https://github.com/NixOS/nix/issues/2633), helped.

# [2.2. The beginnings of the Nix store](https://nixos.org/nixos/nix-pills/install-on-your-running-system.html#idm140737316661696)

"*You may have noticed that /nix/store can contain not only directories, but also files, still always in the form hash-name.*"

# [2.4. The first profile](https://nixos.org/nixos/nix-pills/install-on-your-running-system.html#idm140737316649984)

The `ls -l ~/.nix-profile/` code snippet shows:
```
toraritte@tr2:~$ ls -l ~/.nix-profile
lrwxrwxrwx 1 toraritte toraritte 48 May 17 22:42 /home/toraritte/.nix-profile -> /nix/var/nix/profiles/per-user/toraritte/profile
```

but in reality, it is a symlink  so `readlink` is needed (which is later explained though, but still confusing):
```text
toraritte@tr2:~$ ls -l  $(readlink -f  ~/.nix-profile)
total 12
lrwxrwxrwx 1 toraritte toraritte 57 Jan  1  1970 bin -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin
lrwxrwxrwx 1 toraritte toraritte 57 Jan  1  1970 etc -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/etc
lrwxrwxrwx 1 toraritte toraritte 61 Jan  1  1970 include -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/include
lrwxrwxrwx 1 toraritte toraritte 57 Jan  1  1970 lib -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/lib
lrwxrwxrwx 1 toraritte toraritte 61 Jan  1  1970 libexec -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/libexec
lrwxrwxrwx 1 toraritte toraritte 60 Jan  1  1970 manifest.nix -> /nix/store/b5impca1zc7v4jpbih3zzl92ij4c8w16-env-manifest.nix
lrwxrwxrwx 1 toraritte toraritte 59 Jan  1  1970 share -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/share
```

Following the symlink and showing each intermediate steps (a good [SO thread](https://stackoverflow.com/questions/33255460/how-to-list-all-members-of-a-symlink-chain) on the topic):
```text
$ namei ~/.nix-profile
f: /home/toraritte/.nix-profile
 d /
 d home
 d toraritte
 l .nix-profile -> /nix/var/nix/profiles/per-user/toraritte/profile
   d /
   d nix
   d var
   d nix
   d profiles
   d per-user
   d toraritte
   l profile -> profile-27-link
     l profile-27-link -> /nix/store/8bxb0b37v3vjdklfskaxg7q84k7jy20p-user-environment
       d /
       d nix
       d store
       d 8bxb0b37v3vjdklfskaxg7q84k7jy20p-user-environment
```
With `--long`, `namei` will also print out permissions like `ls -l`.

The above is right after a fresh Nix install on Debian Stretch, and on the same, after installing `tree`:
```text
toraritte@tr2:~$ ls -l  $(readlink -f  ~/.nix-profile)
total 20
dr-xr-xr-x 2 toraritte toraritte 4096 Jan  1  1970 bin
lrwxrwxrwx 1 toraritte toraritte   57 Jan  1  1970 etc -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/etc
lrwxrwxrwx 1 toraritte toraritte   61 Jan  1  1970 include -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/include
lrwxrwxrwx 1 toraritte toraritte   57 Jan  1  1970 lib -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/lib
lrwxrwxrwx 1 toraritte toraritte   61 Jan  1  1970 libexec -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/libexec
lrwxrwxrwx 1 toraritte toraritte   60 Jan  1  1970 manifest.nix -> /nix/store/jcfwrdpgvpb2m043yd0z6394hcw06smw-env-manifest.nix
dr-xr-xr-x 3 toraritte toraritte 4096 Jan  1  1970 share

toraritte@tr2:~$ ls -l  $(readlink -f  ~/.nix-profile/bin)
total 52
lrwxrwxrwx 1 toraritte toraritte 61 Jan  1  1970 nix -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix
lrwxrwxrwx 1 toraritte toraritte 67 Jan  1  1970 nix-build -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-build
lrwxrwxrwx 1 toraritte toraritte 69 Jan  1  1970 nix-channel -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-channel
lrwxrwxrwx 1 toraritte toraritte 77 Jan  1  1970 nix-collect-garbage -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-collect-
garbage
lrwxrwxrwx 1 toraritte toraritte 74 Jan  1  1970 nix-copy-closure -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-copy-closur
e
lrwxrwxrwx 1 toraritte toraritte 68 Jan  1  1970 nix-daemon -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-daemon
lrwxrwxrwx 1 toraritte toraritte 65 Jan  1  1970 nix-env -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-env
lrwxrwxrwx 1 toraritte toraritte 66 Jan  1  1970 nix-hash -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-hash
lrwxrwxrwx 1 toraritte toraritte 73 Jan  1  1970 nix-instantiate -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-instantiate
lrwxrwxrwx 1 toraritte toraritte 74 Jan  1  1970 nix-prefetch-url -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-prefetch-ur
l
lrwxrwxrwx 1 toraritte toraritte 67 Jan  1  1970 nix-shell -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-shell
lrwxrwxrwx 1 toraritte toraritte 67 Jan  1  1970 nix-store -> /nix/store/hbhdjn5ik3byg642d1m11k3k3s0kn3py-nix-2.2.2/bin/nix-store
lrwxrwxrwx 1 toraritte toraritte 63 Jan  1  1970 tree -> /nix/store/xzmw05vn6n17n3nblkzqyr44hsgk9p2p-tree-1.8.0/bin/tree
```

here is one on a NixOS machine in active use:
```text
$ ll $(readlink -f ~/.nix-profile)
total 1524
dr-xr-xr-x    8 root root      4096 Dec 31  1969 ./
drwxrwxr-t 2485 root nixbld 1511424 May 17 10:33 ../
dr-xr-xr-x    2 root root      4096 Dec 31  1969 bin/
dr-xr-xr-x    3 root root      4096 Dec 31  1969 etc/
dr-xr-xr-x    2 root root      4096 Dec 31  1969 include/
dr-xr-xr-x    5 root root      4096 Dec 31  1969 lib/
dr-xr-xr-x    2 root root      4096 Dec 31  1969 libexec/
dr-xr-xr-x   12 root root      4096 Dec 31  1969 share/
lrwxrwxrwx    1 root root        85 Dec 31  1969 google-cloud-sdk -> /nix/store/rh1784zqd1bqpkvmk6b808l73r8f2kqy-google-cloud-sdk-222.0.0/google-cloud-sdk/
lrwxrwxrwx    1 root root        60 Dec 31  1969 manifest.nix -> /nix/store/h44rpzh89chr7l45xg5r150szimi9nl2-env-manifest.nix
lrwxrwxrwx    1 root root        71 Dec 31  1969 sbin -> /nix/store/j9mf82znx5ld38mfpxbjb153v84j0dja-network-manager-1.10.6/sbin/
lrwxrwxrwx    1 root root        70 Dec 31  1969 var -> /nix/store/j9mf82znx5ld38mfpxbjb153v84j0dja-network-manager-1.10.6/var/
```

# [2.5. Nixpkgs expressions](https://nixos.org/nixos/nix-pills/install-on-your-running-system.html#idm140737316634384)

## "*Read `nix.sh`, it's short.*"

`nix.sh` is the one that needs to be run after a fresh install (or restart the shell, because `nix.sh` is now sourced form `.bashrc`). See it in this repo, here's the link: [`nix.sh`](./nix-profile_etc_profile.d_nix.sh).

NixOS doesn't have one, probably because it is baked in from the start. Figure it out.

# [3.1. Enter the environment](https://nixos.org/nixos/nix-pills/enter-environment.html#idm140737316600912)

## "*let's start by switching to it with `su - nix`*"

Why? Isn't the point of showing that a normal user can install packages? Later it explains, so I guess this was for everyone to have a uniform experience.

# [3.2. Install something](https://nixos.org/nixos/nix-pills/enter-environment.html#install-something)

With `nix-env -i hello`, "*We installed hello by derivation name minus the version. I repeat: we specified the **derivation name** (minus the version) to install it.*"

"Generations" are kind of explained in [2.4. The first profile](https://nixos.org/nixos/nix-pills/install-on-your-running-system.html#idm140737316649984): "*Not only that, but profiles are made up of multiple "generations": they are versioned. Whenever you change a profile, a new generation is created.*"

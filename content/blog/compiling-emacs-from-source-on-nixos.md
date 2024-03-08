---
title: "Compiling Emacs From Source on NixOS"
date: 2024-03-08T21:54:19+01:00
tags: ['Emacs']
---
I'm writing this mainly for myself so I don't have to Google it again in the
future. Arguably the easiest way to compile Emacs from source on NixOS is to
enter the interactive shell and override any build inputs that we're
interested in. For example, if we want to build Emacs with ImageMagick,
Tree-Sitter, and native compilation support, we'd run:

```
nix-shell -E '(import <nixpkgs> {}).emacs.override { srcRepo = true; withImageMagick = true; withTreeSitter = true; withNativeCompilation = true; }'
```

This ensures that Nix collects all the dependencies we need. Now it's time to
cook up some Emacs goodness:

```
cd emacs-VERSION
./configure --with-imagemagick --with-tree-sitter --with-native-compilation
make
```

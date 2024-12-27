+++
title = "Life on NixOs"
date = "2024-12-27T22:03:07+01:00"
author = "FKouhai"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "nixOS"]
keywords = ["", ""]
description = "An artictle reflecting on nixOS"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Imagine a world where you can have a system thats reproducible, where you dont need to re-write your configs on every re-install<br>.
A world where you dont need to write weird scripts that may not assure portability after re-installing your system. <br>
Well fear not, the day has come, the day we the nerds were so anxiously waiting for. Nix and NixOS are starting to come into the mainstream.<br>
At least whatever mainstream means for us nerds and tinkerers.


## A little intro

So my journey with nix started some months ago when I was geeking out about it with a coworker, although in the past I was trashing the system a little bit.<br>
I remember a friend of mine started to talk me into it, I watched a couple of talks about it and personally I didn't like the way it was promoted, by just trashing ansible.<br>
Don't get me wrong ansible has a couple of short commings, but overall is an amazing configuration management system and in fact nix configs and ansible dont cancel each other out. <br>
In fact I personally use ansible to push changes to my raspberry pi k3s cluster, its built on nixOS, and for what I've seen ansible is still superior to push changes directly to different hosts. <br>
So going back to the point, after a coworker and I started to geek out about it, we tried it on our laptops, did some quick config on it, and it felt like magic.<br>

At first I was a bit weird out about it, specially due to the fact that the language feelt like nothing I've ever seen before, it ressembled a little bit of Functional Programming languages such as elixir or gleam.<br>
Nix also introduced some new concepts that are strange to understand, for example the flakes which is another way of declaring your configs, adding a bit more of modularity? at least thats my understanding of it.<br>

## Some short commings
I'm not going to pretend that nix/nixOS is perfect, although for now I find it amazing and in fact I have been daily driving nixOS on my desktop and personal laptop for less than a month, it has some stuff that is a bit undesirable, specially coming from arch, the documentation is all over the place<br>
I know engineers are not known for their great ability at documentation, but it definitely helps having a nice wiki like the arch wiki does, so im gonna leave a couple of resources here:
* [mynixos](https://mynixos.com/) -> It's a really great site that allows you to search different configs available for the applications/programs and services you might use
* [nixpkgs](https://search.nixos.org/packages) -> It's the site that allows you to search for the packages you can install.
* [vimjoyer](https://www.youtube.com/@vimjoyer) -> His channel covers a miriad of topics from nix, I have watched a lot of his videos to understand nix a little bit better, and it was a nice resource.
* [my nix dotfiles](https://github.com/FKouhai/nix-dots/tree/desktop) -> This is my personal repo that hosts the dotfiles for both my laptop and desktop(yes I know using branches for it isnt the best and that you can use flakes to unify your conifg).

So after digging around for several hours I managed to have a nice config for both machines that has nixOS, so you may ask but what's the point, since I only re-install so often my systems.<br>
Well dear reader, thats an amazing question, and yes it is true that a full system install doesn't happen very often, but when it does, it could be painful and it could take you some time,if you had some scripts here and there you might spend several hours debugging those because you might be missing a dependency somewhere<br>
It definitely saves up some time, but some other extraordinary features nix and nixOS have are, the ability to go back into different versions of your system, sort of similar to the btrfs snapshots of your drive.
Another great feature is the ability to use nix-shells or [devboxes](https://www.jetify.com/devbox), these allows you to have small,portable and reproducible environments making them a nice replacement of the infamous devcontainers.<br>
and that's because everything in nix is or tries to be reproducible, so it may seem like a hussle to get started on nix, but the pros outweigh the cons on this one.<br>

## The next big stop, the editor
With that said, my next stop after having nixOS installed was to configure my editor of choice(i use nvim btw) the nix way, and that is not easy task, so digging around I found about [nixvim](https://nix-community.github.io/nixvim/index.html)
So, okay this interesting, some of the neovim plugins are packaged in the nix repositories and nixvim allows you some flexibility on declaring the configuration, so I jumped straight into it, and once again once I was done with the config it felt like magic.
Having a more painless way of setting up neovim specially the lsp and completion it made the experience so much better, I still remember how challenging it was at first to have a proper nvim config, and with nixvim it wasn't as challenging.

## Gaming time
Okay so you have an editor, you have your completely reproducible system, but about some fun, well actually installing steam and setting the udev rules for your controller of choice, was a pretty seamless experience, I remember spending hours in arch installing several drivers, not wanting to touch udev rules cause they sound like a PITA to manage<br>
but after some minutes and a reboot to load the udev rules I was able to use my controller to play some games like the latest indiana jones.

## Closing thoughts
Despite the fact that the learning curve of nix and nixOS is a bit steep, it has been a really fun experience I truly enjoy daily driving nix, to either add some plugins to my neovim config, install some binaries or add some configurations.<br>
some things I still want to learn is how to make proper builds for my projects and even contribute upstream with some packaging of either a neovim plugin or a binary I use or may use, my initial thoughts before using nixOS has changed so wildly<br>
That it made it an experience I actually recommend to anyone who is willing to learn it and has the time to do so

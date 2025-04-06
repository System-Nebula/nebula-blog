+++
date = '2025-04-06T21:07:49+02:00'
title = 'Life on Nixos Revisitted'
author = "FKouhai"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "nixOS"]
keywords = ["nix", ""]
description = "A little update on 'life on NixOS'"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++
After more or less 5 months daily driving NixOS and heavily using nix I got some updates and a better formed opinion about it <br>

## What are flakes good for?

A lot of new people getting into nix may see flakes in several places but never really understand why they even exist or what are they useful for.<br>
A short definition of a flake that may make it clearer is, another way to write your nix expressions which in my opinion they make distributing your projects easier, <br>
specially for the enterprise where you don't really have a place to leave your projects/tools so your other teammates could easily use them.<br>
A recent scenario that happened to me was related to terraform-cdk failing to build using the nix packaged version for macOS(aarch64-darwin), <br>
So we had a couple of choices, either drop the usage of our nix shell for that particular project and stick to devcontainers and their fuzzy magic or make the build oruselves<br>
I personally never had doubts on which option I would pick, which was making a nix expression to build it ourselves from source, doing a little research I ended up using a flake<br>
which was extremely useful first to determine that javascript indeed is not a real language and second to keep feeding my curiosity towards nix.
After some time of reading the build process terraform-cdk uses and understanding how javascript "builds" work (yes this process was long and a bit of a PITA, please don't write CLI tools in JS).<br>
I finished up writting up my flake to easily distribute the package and avoid using a devcontainer :). <br>

Another cool thing about flakes is that not only you can distribute and build packages with them, but also your development environment and even make a container (without needing docker).<br>
An example of that kind of flake looks like this:
```(nix)
{
  description = "nix flake that builds url-shortner and provides the development environment used";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs?ref=nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
    gomod2nix = {
      url = "github:tweag/gomod2nix";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = {
    self,
    nixpkgs,
    flake-utils,
    gomod2nix,
  }:
    flake-utils.lib.eachDefaultSystem (
      system: let
        pkgs = import nixpkgs {
          inherit system;
          overlays = [gomod2nix.overlays.default];
          config.allowUnfree = true;
        };
        shorturl = pkgs.buildGoModule {
          name = "shorturl";
          src = ./.;
          vendorHash = "sha256-BJtyFeAL1WK8Ai965iT6t+8rWHNVNrTjVz3NbWhtwi8=";
          proxyVendor = true;
          doCheck = false;
          postInstall = ''
            mv $out/bin/url_shortener $out/bin/shorturl
          '';
        };
        dockerImage = pkgs.dockerTools.buildImage {
          name = "shorturl";
          tag = "latest";
          created = "now";
          copyToRoot = [shorturl];
          config = {
            Cmd = ["${shorturl}/bin/shorturl"];
            Env = [
              "OTEL_EP=$OTEL_EP"
              "VALKEY_SERVER=$VALKEY_SERVER"
            ];
          };
        };
      in
        with pkgs; {
          inherit shorturl dockerImage;
          defaultPackage = shorturl;
          devShells.default = mkShell {
            buildInputs = [
              go
              air
              sqlite
              sqlc
            ];
          };
        }
    );
}
```
So the flake above has as outputs, a docker image, a development shell and the binary itself, these could be consumed like this:
```(bash)
nix develop # this would spawn you inside of a shell with the needed binaries for your project to run
nix build .#dockerImage.x86_64-linux # this would build a docker image in the result directory that you can just load in the docker daemon
nix build .#defaultPackage.x86_64-linux # this would build the binary leaving it under result/bin/binary-name
```
And you may ask, but isn't this just like a fancy Makefile, and to that my answer is yes but also no<br>
take into account that Makefiles run scripts that either install your dependencies, builds your project or even ship it.<br>
But the key difference here is that you can take a flake as an input in another flake and consume any output you need, plus they provide reproducibility by being deterministic.<br>
On another note, flakes are a really good mechanism to unify a little bit more the different steps for your project, instead of using a shell.nix, default.nix and maybe a Makefile.

### A downside on flakes
One of the downsides flakes have is specially for a mono repo where you just wan't to have flakes, it makes importing a specific flake a little bit more complicated<br>
Since you end up needing to either push each flake as a tarball to an artifact registry or a bucket (if your git server relies under ssh), or playing around with the attributes you can pass to your inputs.

## nixpkgs
Now I wanted to talk about nixpkgs, during this period of time I started to be a pkg maintainer, nix has been the only open source software that trully made me wan't to contribute back to it <br>
So the first thing about nixpkgs is taking into account the style guide for your packages, you need to use nixfmt instead of alejandra which is my prefferred formatter,<br>
there's some nitpicks around naming conventions, using `tag` instead `rev` or `hash` instead of `sha256`, but regardless of that it's a really fun experience. <br>
It has been an experience that made me so much better at using git, and appreciating it a lot more, so if you want to contribute to it feel free to do so, you are going to learn more than you think.<br>

## Conclussion
During this period of time, I feel like nix is becoming second nature to me although im sure there's a lot more to learn about this amazing software.<br>
Flakes are awesome, NixOs is one of the better distros and javascript is not a real language. _peace_


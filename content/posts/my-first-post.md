+++
title = "My First Post"
date = "2024-12-27T20:58:44+01:00"
author = "FKouhai"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "devops"]
keywords = ["", ""]
description = "A blog about how to make a blog"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## Introduction
In this first blog post, im gonna write about how to make a hugo site and why we decided to make one in the first place. <br>
### How to create a blog
First and foremost Im using nixOS to write this blog. I didnt want to install hugo locally so here is what I did: 
```(bash)
nix-shell -p hugo
```
I created a nix shell with hugo installed to not over populate my current build with stuff im not sure I will use.<br>
After that I run the following command:
```(bash)
hugo new site nebula-blog
```
Then I stumbled into some issues with the gh actions, I just cant read :) <br>
So I decided to learn how to actually read and run the following commands: 
```(bash)
hugo mod init system-nebula.github.io
hugo mod get github.com/mirus-ua/hugo-theme-re-terminal
# I edited my hugo config to include the module theme
```
```(toml)
[module]
[[module.imports]]
    path = 'github.com/mirus-ua/hugo-theme-re-terminal'
```
After that the gh action run succesfully and we got a site :)

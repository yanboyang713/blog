---
title: "Using Multiple Emacs Configurations with Chemacs2"
draft: false
---

## The challenge of multiple Emacs configurations {#the-challenge-of-multiple-emacs-configurations}

When you start using [Emacs]({{< relref "20230107214049-emacs.md" >}}) , you typically either build up your own configuration from scratch little by little or adopt one of the many pre-built configurations like **Doom Emacs**, **Spacemacs**, or **Prelude**.

But what if you want to experiment with other Emacs configurations or even rebuild your existing config from scratch?

Emacs doesn't make it very easy to use multiple Emacs configurations at the same time. Since there is a well-defined set of locations where Emacs looks for your configuration, your only option to try a new configuration is to move the old one out of the way, move the new one into its place, and restart Emacs.

This is a pain to deal with, especially if you want to use multiple configurations at the same time in separate Emacs windows. I have this problem all the time when running my own configuration and showing off the Emacs From Scratch configuration or someone else's config at the same time!

There must be a better way to solve this problem...


## Enter Chemacs2 {#enter-chemacs2}

[Chemacs2](https://github.com/plexus/chemacs2) is a very small Emacs configuration "shim" which allows you to set up multiple Emacs configuration profiles so that you can easily switch between them or use them all simultaneously!

With Chemacs, you can:

-   Use both Doom Emacs and Spacemacs simultaneously
-   Use Doom or Spacemacs while building your own Emacs config from scratch
-   Try out the Emacs configurations of others very easily
-   Redesign your Emacs configuration while still using your current one for productivity
-   Run the same Emacs configuration with different settings for different purposes

It's called Chemacs2 because it's the successor to the original Chemacs, both written by Arne Brasseur. The main benefit of Chemacs2 over Chemacs is that it supports the new early-init.el config file from Emacs 27 (used by Doom Emacs and others).

Windows users, I've personally verified that Chemacs2 works perfectly there too! It actually made my personal Emacs configuration easier to manage because I no longer needed to create a symbolic link from my dotfiles repo to the ~/.emacs.d path!


## Setting it up {#setting-it-up}

Clone the Chemacs 2 repository as $HOME/.emacs.d. Note that if you already have an Emacs setup in ~/.emacs.d you need to move it out of the way first. If you have an ~/.emacs startup script then move that out of the way as well.

```bash
[ -f ~/.emacs ] && mv ~/.emacs ~/.emacs.bak
[ -d ~/.emacs.d ] && mv ~/.emacs.d ~/.emacs.default

git clone https://github.com/plexus/chemacs2.git ~/.emacs.d
```

Note that this is different from Chemacs 1. Before Chemacs installed itself as ~/.emacs and you could have your own default setup in ~/.emacs.d. This approach no longer works because of ~/.emacs.d/early-init.el, so Chemacs 2 needs to be installed as ~/.emacs.d.


## Managing your Emacs profiles {#managing-your-emacs-profiles}

To tell Chemacs2 where to get your Emacs configuration, create a file in your home directory called ~/.emacs-profiles.el and populate it with this line:

```emacs-lisp
(("default" . ((user-emacs-directory . "~/.emacs.default"))))
```

Now that we have this file in place, let's start Emacs to verify that it works.


### Adding a profile {#adding-a-profile}

Let's imagine that we wanted to create a new configuration to run alongside the current one. This is just as easy as adding a new entry to the ~/.emacs-profiles.el file:

```emacs-lisp
(("myconfig" .  ((user-emacs-directory . "~/.my-emacs")))
 ("newconfig" . ((user-emacs-directory . "~/.my-new-emacs"))))
```


### Setting the default profile {#setting-the-default-profile}

When you have multiple profiles set up in this way, you can easily specify which one gets chosen as the default by creating the file ~/.emacs-profile and writing the name of the default profile there:

```bash
echo 'myconfig' > ~/.emacs-profile
```


## A real example! {#a-real-example}

Here's an example of using Chemacs2 to use 4 different Emacs configurations:

```emacs-lisp
(("default"   . ((user-emacs-directory   . "~/.dotfiles/.emacs.d")))
 ("doom"      . ((user-emacs-directory   . "~/emacs-configs/doom-emacs")
                 (env . (("DOOMDIR"      . "~/.dotfiles/.doom.d")))))
 ("spacemacs" . ((user-emacs-directory   . "~/emacs-configs/spacemacs")
                 (env . (("SPACEMACSDIR" . "~/.dotfiles/.spacemacs.d")))))
 ("efs"       . ((user-emacs-directory   . "~/emacs-configs/emacs-from-scratch"))))
```

**NOTE**: The env lines aren't necessary if your Doom or Spacemacs personal config folders are in their default locations!

We can now launch Emacs with any of these configurattions at the same time using the following command:

```bash
emacs --with-profile=profilename
```

So if we wanted to run both Doom and Spacemacs at the same time, we can run:

```bash
emacs --with-profile=doom &
emacs --with-profile=spacemacs &
```

Also note that we have a a profile named default, this will automatically make that profile the default when you run Emacs without specifying a profile, but you can still override the default by setting up .emacs-profile!


### Doom Emacs Tip {#doom-emacs-tip}

If you originally cloned Doom Emacs to the ~/.emacs.d folder and you now had to move it to make room for Chemacs, you might need to do one of two things to ensure that the configuration works!

The first thing to try is to run doom sync in the new folder location:

```console
$ <new doom location>/bin/doom sync
```

If that doesn't work, you will need to delete the Doom config repo folder and re-clone it into the new location! This will also require running bin/doom install again.


## Using Chemacs2 in your dotfiles repository {#using-chemacs2-in-your-dotfiles-repository}

If you have a dotfiles repository (you should!), you can make the Chemacs2 repo a submodule of your dotfiles repository so that you can replicate your multi-Emacs configuration setup on any of your machines.

The first step is to initialize the Chemacs2 repository as a submodule wherever you previously kept your .emacs.d folder in your dotfiles repo:

```bash
cd ~/.dotfiles
git submodule add https://github.com/plexus/chemacs2 .emacs.d
```

If you are using a tool like GNU Stow or dotcrafter.el to automatically link files from your dotfiles folder to your home folder, make sure to clone the submodule into the folder that you create your symbolic links from.

Keep in mind that you will need to clone your dotfiles repo with the --recursive option from now on:

```bash
git clone <URL-of-dotfiles-repo> --recursive
```

This will both your dotfiles repository and also any submodule repositories that are needed.


## Setting the eln-cache path correctly {#setting-the-eln-cache-path-correctly}

One thing I noticed while using Chemacs2 with Emacs 28 and native compilation functionality turned on: the compiler output files were being sent to Chemacs' config folder instead of the real folder where my personal configuration lives. To send those files to the right location, add this line to your configuration:

```emacs-lisp
(add-to-list 'comp-eln-load-path (expand-file-name "eln-cache/" user-emacs-directory))
```

Doom Emacs seems to have adjusted for this but I can't see it done in Spacemacs, might be needed there too!

IMPORTANT NOTE: As of this commit to Emacs 28 master branch, this variable has been renamed to native-comp-eln-load-path!


## What will you use Chemacs2 for? {#what-will-you-use-chemacs2-for}

Hopefully this gives you some interesing ideas for how you can use Chemacs2 for managing your Emacs configurations!

I'm curious to hear how you can imagine yourself using it, so please leave a comment on the video and let me know!


## Reference List {#reference-list}

1.  <https://systemcrafters.cc/emacs-tips/multiple-configurations-with-chemacs2>
2.  <https://emacs.stackexchange.com/questions/36380/use-several-emacs-configurations-versions-simultaneously>

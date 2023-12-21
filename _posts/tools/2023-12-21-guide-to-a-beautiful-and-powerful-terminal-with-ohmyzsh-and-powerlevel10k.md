---
title: "Guide To A Beautiful And Powerful Terminal With OhMyZsh and Powerlevel10k"
date: 2023-12-21 17:29
categories: [Tools]
tags: [ohmyzsh, terminal, zsh]
img_path: /tools/powerlevel10k
---

Recently, I learned how to configure the look of the zsh terminal on macOS and was stunned with how many options there are. Since the Powerlevel10k theme is my favorite so far, I will show you how to set it up so you can brag about your new terminal to your peers.

![Powerlevel10k Terminal](powerlevel10k-terminal.png)

> This guide is intended for macOS with zsh as default terminal. However, there are alternatives for [Bash](https://github.com/ohmybash/oh-my-bash) and [Powershell](https://ohmyposh.dev/) that work similarly.
{: .prompt-info}

## Install OhMyZsh

First, we are going to install [OhMyZsh](https://ohmyz.sh/) which is a framework for handling your zsh configuration. 

```console
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Running the above command backs up your current .zshrc file and creates a new one based on the OhMyZsh template. It also downloads all plugins, themes, and some other files and puts them into the */Users/{username}/.oh-my-zsh* directory.

There are also over 300 plugins and 150 themes that you can activate. For example, the git plugin is activated by default and includes several shortcuts for git commands which can make your life as a developer a bit easier once you get to know them. This is just a side note, though, as the main part of this guide are the themes. A complete list of all plugins and themes can be found [here](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins) and [here](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes), respectively.

## Set Up Fonts

When you restart the terminal after installing OhMyZsh, the _robbyrussel_ theme will be used.

However, you might notice that the icons are not shown correctly. This is a font issue and can be solved by installing the following ttf files as mentioned by [powerlevel10k's instructions](https://github.com/romkatv/powerlevel10k#manual-font-installation):
- [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
- [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
- [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
- [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

After installing them, you need to configure your terminal to use these fonts by going to _Terminal → Preferences → Profiles → Text_, click _Change_ under _Font_ and select `MesloLGS NF` family.

## Install & Configure Powerlevel10k

Let's get to installing the Powerlevel10k theme now. This is the theme you can see in picture at the beginning of this post. It can be found on [romkatv's GitHub page](https://github.com/romkatv/powerlevel10k). 

Since we have already installed OhMyZsh, we can use the following command to clone the repositories into OhMyZsh's theme folder:

```console
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

After it has finished downloading, set the `ZSH_THEME` in your new `~/.zshrc` file to `powerlevel10k/powerlevel10k` and restart your shell.

After restarting, a configuration wizard should open automatically. If it doesn't, type `p10k configure` to trigger it manually. Choose the options that suit you most.

Enjoy your new shiny shell!

> If you don't care about OhMyZsh's plugins and only want the Powerlevel10k theme, you can also install it via a different method. You can find all installation methods [here](https://github.com/romkatv/powerlevel10k#installation).
{: .prompt-info}

## Configure VS Code

If you use VS Code, you may realize that fonts don't work correctly on their built-in terminal. In order to solve that, do the following:[^1]
1. Press `Command + Shift + P` in VSC to open 
2. Type `settings.json`
3. Select `Preferences: Open Settings (JSON)`
4. Add `"terminal.integrated.fontFamily": "MesloLGS NF"` to the JSON root
5. Save and restart VSC

[^1]: Source: [Medium](https://youngstone89.medium.com/how-to-change-font-for-terminal-in-visual-studio-code-c3305fe6d4c2)





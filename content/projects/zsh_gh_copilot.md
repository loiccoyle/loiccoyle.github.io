---
author:
  - Loïc Coyle
title: "Copilot powered command suggestions and explanations in the terminal"
date: "2024-12-14"
description: A zsh plugin for Github Copilot
summary: A zsh plugin for Github Copilot
categories:
  - Projects
tags:
  - zsh
  - Copilot
  - shell
ShowToc: true
TocOpen: false
cover:
  image: /images/projects/zsh_gh_copilot/usage.gif
  alt: Example usage of getting command suggestions and explanations in the terminal
---

A quick step by step guide to get AI powered suggestions and explanations, using [Github Copilot](https://github.com/features/copilot), in your `zsh` powered terminal.

> This will require access to Github Copilot, it is available for [free for students](https://education.github.com/pack). This guide also assumes you're using `zsh`.

## Step 1 - Installing `gh` & `gh-copilot`

As you may know, Github maintains a pretty nice CLI [{{< ico bootstrap github >}} cli/cli](https://github.com/cli/cli), they also maintain an extension, [{{< ico bootstrap github >}} github/gh-copilot](https://github.com/github/copilot), which let's you interact with the Copilot LLM using terminal commands.

- On macOS:

```console
brew install gh
```

- On Arch:

```console
pacman -S github-cli
```

> For other operating systems, follow the install instructions in the [{{< ico bootstrap github >}} cli/cli](https://github.com/cli/cli) repo.

Once installed, you'll have to authenticate:

```console
gh auth login --web
```

Next, to install the `gh-copilot` extension:

```console
gh extension install github/gh-copilot --force
```

You can try it out with:

```console
gh copilot suggest ...
gh copilot explain ...
```

> Don't forget to update the `gh-copilot` extension with `gh extension upgrade --all` from time to time.

## Step 2 - Installing the `zsh-gh-copilot` plugin

To be able to easily bind the `suggest` and `explain` subcommands to a key combination, you'll need to install the [{{< ico bootstrap github >}} loiccoyle/zsh-gh-copilot](https://github.com/loiccoyle/zsh-gh-copilot) plugin.

This process will depend on the `zsh` plugin manager you use. For example, for [`zplug`](https://github.com/zplug/zplug), add the following to your `.zshrc` file:

```zsh
zplug "loiccoyle/zsh-github-copilot"
```

> I'll encourage you to check out the [README](https://github.com/loiccoyle/zsh-github-copilot?tab=readme-ov-file#-installation) instructions for your plugin manager of choice.

This plugin will register two new `zle` keymaps, `zsh_gh_copilot_suggest` and `zsh_gh_copilot_explain`.

## Step 3 - Binding `explain` & `suggest`

The final step is to bind the newly registered keymaps to keys. Simply add the following to your `.zshrc`:

- On macOS:

```zsh
bindkey '¿' zsh_gh_copilot_explain  # bind Option+shift+\ to explain
bindkey '÷' zsh_gh_copilot_suggest  # bind Option+\ to suggest
```

- On Linux:

```zsh
bindkey '^[|' zsh_gh_copilot_explain  # bind Alt+shift+\ to explain
bindkey '^[\' zsh_gh_copilot_suggest  # bind Alt+\ to suggest
```

And you're all done, just open up a new terminal, type out a query or a command and hit the appropriate key bind.

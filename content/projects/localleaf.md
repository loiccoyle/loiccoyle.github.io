---
author:
  - Loïc Coyle
title: "localleaf - An Overleaf like experience, but locally!"
date: "2025-01-13"
description: "Headache free LaTeX"
summary: "Headache free LaTeX"
categories:
  - Posts
tags:
  - Workflow
  - LaTeX
  - Docker
ShowToc: true
TocOpen: false
cover:
  image: /images/projects/localleaf/cover.png
  alt: Image of the Overleaf logo with an arrow poiting to a desktop computer
---

## Why is LaTeX still the standard?

It is [generally accepted](https://www.google.com/search?q=why+is+latex+so+terrible%3F) that LaTeX is a terrible language, why it is still the standard for scientific writing is beyond me.
Impressively, LaTeX has managed to achieve the trifecta of terribleness, it is a convoluted mess of a language, it is terrible to install and configure and is woefully under-documented.

## Overleaf

To try to mitigate its suckiness, [Overleaf](https://overleaf.com) stepped in to at least provide a reasonable writing experience by abstracting away the pain of installing and setting up LaTeX behind a web based editor.

> Also, Overleaf is open-source [{{< ico bootstrap github >}} overleaf/overleaf](https://github.com/overleaf/overleaf) 🎉

On top of that, they have done a lot of work to [document and provide recipes for common tasks](https://www.overleaf.com/learn), which is awesome.

> Overleaf has basically become de facto way of writing LaTeX documents, especially for collaborative work.

The downside is that Overleaf is web based, which means you can't bring your editor and you need to have an internet connection to write. It also runs on their back-end which can be sluggish, depending on your subscription plan.

{{< figure src="/images/projects/localleaf/overleaf.png" align="center" width="700px" title="Overleaf pricing at the time of writing">}}

**Most importantly they have an infuriating compile-time timeout for non paying users which makes writing big documents next to impossible for free users.**

## localleaf - A local alternative

Instead of using a web-app to abstract away the pains of LaTeX, why don't we install and run the LaTeX engine in an isolated environment like [`docker`](https://docker.com). This is the basic idea behind [{{< ico bootstrap github >}} loiccoyle/localleaf](https://github.com/loiccoyle/localleaf).

`localleaf` is just a [~150 LOC bash script](https://github.com/loiccoyle/localleaf/blob/main/localleaf) which spins up a [textlive](https://tug.org/texlive/) [`docker` image](https://github.com/loiccoyle/localleaf/blob/main/Dockerfile) and mounts the project directory into the container. It then runs the `latexmk` command in the container and copies the output back to the host.
By default it will monitor the LaTeX files for changes, using [`entr`](https://github.com/eradman/entr), and builds when a change is detected.

There are a few more bells and whistles, but that's the gist of it.

```console
$ localleaf -h
Easy breezy latex.

Spins up a latex docker image, monitors .tex files and builds on change.

Usage: localleaf [OPTIONS] [PROJECT_DIR] -- [EXTRA_ARGS]
  -h                          Show this message and exit.
  -m MAIN_DOCUMENT            The main document of the latex project.
  -e ENGINE                   Latex engine. [pdflatex] {latex,pdflatex,xelatex,lualatex}
  -i IMAGE                    Docker image. [loiccoyle/localleaf]
  -c                          Commit changes on exit.
  -1                          Don't monitor, build once and exit.
  PROJECT_DIR                 Root directory of the latex project. ['.']
  EXTRA_ARGS                  Extra arguments to pass to latexmk, e.g. --outdir=build/
```

Care was taken to use the same build command and LaTeX environment as Overleaf, by looking at their [source code](https://github.com/overleaf/overleaf/blob/main/services/clsi/app/js/LatexRunner.js). So the resulting pdfs should be identical.

> Pro-tip: pass `latexmk` the `--outdir` and `--auxdir` args by running `localleaf {your args} -- --outdir=build/ --auxdir=aux/` to not clutter the root directory of the project with the build and auxiliary files.

By running everything locally, you can host the project on github and benefit from the collaborative features of git.

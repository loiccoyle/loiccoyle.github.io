---
author:
  - Loïc Coyle
title: "Strandify: no strings attached string art"
date: "2024-12-10"
description: A rust library, binary, wasm package and web app
summary: A rust library, binary, wasm package and web app
categories:
  - Projects
tags:
  - Rust
  - Art
  - String Art
  - Image processing
  - Wasm
ShowToc: false
TocOpen: false
cover:
  image: /images/projects/strandify/cover.png
  alt: Screenshot of the strandify web app
---

In the spirit of my previous `rust`/`WASM` [experiments](https://loiccoyle/projects/phomo_webapp.md), I've developed [`strandify`](https://loiccoyle.com/strandify), a [computational string art](https://en.wikipedia.org/wiki/String_art) web app, for all your string art needs.

<h3>
  <a href="https://loiccoyle.com/strandify" style="display: flex; align-items: center; justify-content: center; gap: 0.5em; background-color: cornflowerblue; border-radius: 1em; margin:auto; max-width: 10em;">
    <img src="/images/projects/strandify/logo.png" alt="Strandify logo" width="64px" />
    Strandify
  </a>
</h3>

Just like my [photo mosaic web app](https://loiccoyle.com/projects/phomo_webapp), the front-end uses `React` and `vite`. The computation is done in web workers, via Wasm bindings to a `rust` crate.

All source code is open source:

- `rust` crates and Wasm bindings: [{{< ico bootstrap github >}} loiccoyle/phomo](https://github.com/loiccoyle/strandify)
- Web app: [{{< ico bootstrap github >}} loiccoyle/phomo (gh-pages)](https://github.com/loiccoyle/strandify/tree/gh-pages)

## Features

First and foremost, there are not strings attached (pun intended), it is free and **all processing is done on the client device, the images never leave your device.**

Once an image is selected, it is loaded into the canvas and the user is able to place "pegs". These pegs define the connection points of the string.

{{< figure src="/images/projects/strandify/peg_tools.gif" align="center" width="500px" title="Using the brush tools to place the pegs">}}

The configuration of the string art pathing algorithm is left up to the user. By default it will use a purely greedy algorithm. By setting a beam search width greater than one, it will transition to a [beam search](https://en.wikipedia.org/wiki/Beam_search).

The rendering of the string art can be configured by setting the width, opacity, color of the yarn as well as the background color. These will update the generated string art without having to re-generate it.

{{<figure src="/images/projects/strandify/config_options.png" width="500px" align="center" title="String art pather configuration options">}}

Once the string art is generated, a player is displayed which lets you play through the string art generation process. The string art can also be downloaded as a `svg` or a `png` file.

{{<figure src="/images/projects/strandify/player.gif" width="500px" align="center" title="Playing through the generated string art">}}

The `Strandify` web app can also be used on mobile devices and includes a [PWA](https://en.wikipedia.org/wiki/Progressive_web_app) manifest so that it can be added to the home page and used like any other app.

---
author:
  - Loïc Coyle
title: "Phomo: photo mosaic web app"
date: "2024-10-22"
description: A rust library, binary, wasm package and web app
summary: A rust library, binary, wasm package and web app
categories:
  - Projects
tags:
  - Rust
  - Art
  - Photo mosaic
  - Image processing
  - Wasm
ShowToc: true
TocOpen: false
cover:
  image: /images/projects/phomo_webapp/cover.png
  alt: Screenshot of the phomo web app
---

> 🚀 **[Check it out!](https://loiccoyle.com/phomo-rs)** 🚀

Previously, I've built [{{< ico bootstrap github >}} loiccoyle/phomo](https://github.com/loiccoyle/phomo) a python package to construct [photo mosaics](https://en.wikipedia.org/wiki/Photographic_mosaic), complete with GPU acceleration.

> I've written about it [here](https://loiccoyle.com/projects/phomo).

As an educational project, to improve my rust skills and learn how to use [WebAssembly](https://webassembly.org/), I've built [{{< ico bootstrap github >}} loiccoyle/phomo-rs](https://github.com/loiccoyle/phomo-rs). It's a rust port of `phomo` complete with a binary crate and Wasm bindings. As well as an accompanying [web app](https://loiccoyle.com/phomo-rs).

## The rusty part

There isn't much point delving into the inner workings of the rust library itself, as it is essentially a direct port of the `phomo` python package I've previously built. **If you're curious about it, you can [read about it](https://loiccoyle.com/projects/phomo-rs).**

## The web app

What is more interesting is the web app part, and how to use go from Rust, to Wasm, to using it in the browser.

### Wasm is it?

WebAssembly or Wasm is a low-level, portable binary instruction format designed to run code efficiently across different platforms. It enables high-performance execution of code in web browsers and other environments by allowing developers to compile code written in languages like C, C++, or Rust into a compact binary that can run natively on the browser's virtual machine. **Achieving near native performance with the flexibility of the web.**

> Wasm speed, engage!

Unlike JavaScript, which is parsed and JIT compiled at runtime by the [js engine](https://en.wikipedia.org/wiki/JavaScript_engine), Wasm code is compiled ahead of time into low-level machine instructions, allowing the compilation process to do more intensive optimizations, reducing execution overhead and increased performance. It is executed by the [Wasm virtual machine](https://en.wikipedia.org/wiki/WebAssembly#Virtual_machine) built into most modern browsers, following the [Wasm spec](https://webassembly.github.io/spec/). Wasm also offers more efficient memory management, avoiding the performance costs associated with JavaScript's dynamic typing and garbage collection systems.

### Rust to Wasm

There are number of great resources, explaining how to compile rust to Wasm.

> The [Rust and WebAssembly](https://rustwasm.github.io/docs/book/) book is invaluable.

So I'll just give an exceedingly brief explainer of the steps required to add Wasm bindings to an existing rust library.

First initialize a new crate:

```Sh
cargo init --lib mycrate-wasm
```

Open up its `Cargo.toml` and add the following:

```toml
...
[lib]
crate-type = ["cdylib"]
```

And add the `wasm-bindgen` dependency with `cargo add wasm-bindgen`

> This tells the compiler to produce a dynamic system library, which can be loaded from another language and built into a Wasm binary.

Next you'll need to install [`wasm-pack`](https://webassembly.github.io/spec/), its the go to for building Wasm binaries and packaging them into Node.js packages.

> Check out the `wasm-pack` [docs](https://rustwasm.github.io/docs/wasm-pack/)!

To install it, just run `cargo install wasm-pack`.

So all the tooling is installed, next you just need to write your bindings. I'll use my [`phomo-wasm`](https://github.com/loiccoyle/phomo-rs/tree/main/phomo-wasm) package as an example. I'll walk through the [`lib.rs`](https://github.com/loiccoyle/phomo-rs/blob/main/phomo-wasm/src/lib.rs) file.

First, there is some boilerplate, to enable the [`log`](https://docs.rs/log/latest/log/) crate to log to the web console and have better stack traces on panics in Rust land, courtesy of [`console_error_panic_hook`](https://docs.rs/console_error_panic_hook/latest/console_error_panic_hook/).

```rust
// ...
use wasm_bindgen::prelude::*;
extern crate wasm_logger;

#[wasm_bindgen(start)]
pub fn init_panic_hook() {
    wasm_logger::init(wasm_logger::Config::default());
    console_error_panic_hook::set_once();
}

```

To add bindings to a `struct`, I like to create a wrapper `struct` with the `wasm_bindgen` derive macro which wraps the rust `struct` contained in its `inner` field. Here I'm wrapping the `phomo::Mosaic` `struct` imported as `MosaicRs`:

```rust
// ...
#[wasm_bindgen]
pub struct Mosaic {
    inner: MosaicRs,
}

#[wasm_bindgen]
impl Mosaic {
    #[wasm_bindgen(constructor)]
    pub fn new(
        master_img_data: &[u8],
        tile_imgs_data: js_sys::Array,
        grid_width: u32,
        grid_height: u32,
        tile_resize: Option<ResizeType>,
    ) -> Result<Mosaic, JsValue> {
// ...
```

Notice the `Err` in the `Result` nomad, is this mysterious `JsValue`, which serves as an intermediary between Rust and JavaScript owned values. `wasm-bindgen` requires the `Err` type to implement `Into<JsValue>`. So all `Err` values need a bit of help to play well with `JsValue`, for example:

```rust {hl_lines=[7]}
#[wasm_bindgen(js_name = transferTilesToMaster)]
pub fn transfer_tiles_to_master(&mut self) -> Result<(), JsValue> {
    self.inner.master = MasterRs::from_image(
        self.inner.master.img.match_palette(&self.inner.tiles),
        self.inner.grid_size,
    )
    .map_err(|err| JsValue::from(err.to_string()))?;
    Ok(())
```

> The rest is relatively self-explanatory.

To build the Wasm binary and construct the Node.js packages, just run `wasm-pack build`. The Node.js package will be built to the `pkg/` directory and can be published to the [npm registry](https://www.npmjs.com/package/phomo-wasm) with `npm publish pkg/`.

### Wasm in Web App

For the `phomo` web app, **source code available [here](https://github.com/loiccoyle/phomo-rs/tree/gh-pages)**, I'm using the [`Vite`](https://vite.dev) build tool. Which requires some extra configuration to use Wasm. I'm assuming a basic `VIte` project is already set up.

Let's go through it, first let's install our Wasm package:

```sh
npm add phomo-wasm
```

Then we need to add the Wasm and top level await plugins:

```sh
npm add vite-plugin-wasm vite-plugin-top-level-await
```

> In my case, I'm using `react` so I also need the `@vitejs/plugin-react` package.

And configure `Vite` to use it, edit the `vite.config.js` file:

```ts {hl_lines=[9,10]}
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import wasm from "vite-plugin-wasm";
import topLevelAwait from "vite-plugin-top-level-await";

// https://vitejs.dev/config/
export default defineConfig({
  // ...
  plugins: [react(), wasm(), topLevelAwait()],
  worker: { plugins: () => [wasm(), topLevelAwait()], format: "es" },
});
```

> With that done we can seamlessly use our Wasm package in the web app and even in [web workers](https://web.dev/learn/performance/web-worker-overview).

<!-- ### A couple gotchas -->
<!---->
<!-- - [`rayon`](https://docs.rs/rayon) [doesn't play too well with Wasm](https://github.com/rayon-rs/rayon/tree/97c1133c2366a301a2d4ab35cf686bca7f74830f?tab=readme-ov-file#usage-with-webassembly), there are ways to get it to work, using [`wasm-bindgen-rayon`](https://github.com/RReverser/wasm-bindgen-rayon), but it requires a bit of setup and I haven't had much success. -->

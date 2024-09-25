---
author:
  - Loïc Coyle
title: "thqm: dynamic menu over the network"
date: "2024-09-23"
description: A command line utility to generate and serve a dynamic menu web page.
summary: A Commandl ine utility to generate and serve a dynamic menu web page.
categories:
  - Projects
tags:
  - Rust
  - Scripting
ShowToc: true
TocOpen: false
cover:
  image: /images/projects/thqm/cover.png
  alt: Image with a thqm script on the left and a menu web page on the right
---

Let's take a look at [{{< ico bootstrap github >}} loiccoyle/thqm-rs](https://github.com/loiccoyle/thqm-rs).

Its a command line utility written in `rust` which generates and serves a menu web page on the local network.

## Inspiration

`thqm` takes a lot of ideas from dynamic menu software like [`dmenu`](https://tools.suckless.org/dmenu/) and [{{< ico bootstrap github >}} davatorium/rofi](https://github.com/davatorium/rofi).
The basic idea is to read a list of menu entries from `stdin`, generate a menu, and forward a user's selection to `stdout`.

> For the interested, I've made my fair share of script which rely on `rofi`'s `dmenu` to interface with the user:
>
> - [{{< ico bootstrap github >}} loiccoyle/rofi-ttv](https://github.com/loiccoyle/rofi-ttv): Dynamic menu interface for twitch.
> - [{{< ico bootstrap github >}} loiccoyle/rofi-nordvpn](https://github.com/loiccoyle/rofi-nordvpn): Dynamic menu interface for nordvpn.
> - [{{< ico bootstrap github >}} loiccoyle/rofi-prowlet](https://github.com/loiccoyle/rofi-prowlet): Rofi wrapper for prowlet. Use the Prowlarr search API to find torrents.

Where `thqm` comes in is in allowing the user to interact with the menu and script over the network.
This let's the user create simple scripts which can be used to control or interact with the host computer from any device on the local network.

## The building blocks

`thqm` relies on two main components to function, the http server which serves the menu on the network and the templating engine which generates the menu.

### Web server

`thqm` doesn't need to a high performance or featureful web server. So I decided to use the small and flexible micro-framework [{{< ico bootstrap github >}} tomaka/rouille](https://github.com/tomaka/rouille).

Here is roughly how `thqm`'s' the web server is configured:

```rust
use rouille::{match_assets, router, Response, Server};

// ...
let server = Server::new(address, move |request| {
    router!(request,
    (GET) (/) => {
        // show the generated menu
        Response::html(&rendered_template)
    },
    (GET) (/select/{entry: String}) => {
        // handle printing the user's selection to stdout
        handle_select(entry)
    },
    _ => {
        // provide static resources (css, js, images, etc)
        let response = match_assets(request, &style_base_path);
        if response.is_success() {
            response
        } else {
            Response::empty_404()
        }
    }
    )
})
// ...
```

> For the full source code of the server, see the [source](https://github.com/loiccoyle/thqm-rs/blob/main/src/server.rs).

### Templating engine

`thqm` uses the [{{< ico bootstrap github >}} keats/tera](https://github.com/keats/tera) templating engine. It takes inspiration from the popular [`jinja2`](https://github.com/pallets/jinja) templating engine.

To generate a web page containing the list of entries, the `index.html` contains something like:

```html
<!-- ... -->
{%- for e in entries %}
<div>
  <button onclick="fetch('./select/' + '{{ e }}')">
    <pre>{{ e | safe }}</pre>
  </button>
</div>
{%- endfor %}
<!-- ... -->
```

`thqm` provides the template with the `entries` variable which is read from `stdin`:

```rust
use tera::{Context, Tera};

// ...
let template_contents = fs::read_to_string(template_path)?;
let mut context = Context::new();

context.insert("entries", &template_options.entries);
let result = Tera::one_off(&template_contents, &context, true)?;
// ...
```

> `thqm` comes with a few menu template styles which live at [{{< ico bootstrap github >}} loiccoyle/thqm-styles](https://github.com/loiccoyle/thqm-styles).

## Example scripts

### Basic

Most scripts will look something like this:

```bash
#!/bin/sh

# define the handler function, i.e. what each option should do.
handler() {
  while IFS= read -r event; do
    case "$event" in
    "Option 1")
      # handle Option 1
      ;;
    "Option 2")
      # handle Option 2
      ;;
    *)
      # pass through thqm's output
      echo "$event"
      ;;
    esac
  done
}

printf "Option 1\nOption 2" | thqm "$@" | handler
# ^                                 ^     ^ Pass user selections to the handler
# │                                 └ Forward script's options to thqm
# └ Provide the options to thqm through stdin
```

The menu entries are piped to `thqm` which then pipes the user's selections to a `handler` function.

### Media control

Controlling media playback and volume on the host is a great use case for `thqm`, in fact an [example](https://github.com/loiccoyle/thqm-rs/blob/main/examples/thqm-media-fa) is included in the repo for exactly this. This script uses the [`fa-grid`](https://github.com/loiccoyle/thqm-styles/tree/main/fa-grid) style which interprets the provided input as `fontawesome` icons and displays them in a grid optimized for mobile devices.

```bash
#!/usr/bin/env bash

# This script uses thqm to create a dashboard to control the playback and volume
# of media playing on the host, using fontawesome icons.
# Requires xdotool, pactl

media_control() {
    while IFS= read -r event; do
        case "$event" in
        "fas fa-volume-up")
            pactl set-sink-volume @DEFAULT_SINK@ +10%
            ;;
        "fas fa-volume-down")
            pactl set-sink-volume @DEFAULT_SINK@ -10%
            ;;
        "fas fa-volume-mute")
            pactl set-sink-mute @DEFAULT_SINK@ toggle
            ;;
        "fas fa-play")
            xdotool key --clearmodifiers XF86AudioPlay
            ;;
        "fas fa-step-backward")
            xdotool key --clearmodifiers XF86AudioPrev
            ;;
        "fas fa-step-forward")
            xdotool key --clearmodifiers XF86AudioNext
            ;;
        "fas fa-arrow-right")
            xdotool key --clearmodifiers Right
            ;;
        "fas fa-arrow-left")
            xdotool key --clearmodifiers Left
            ;;
        "fas fa-play-circle")
            xdotool key --clearmodifiers space
            ;;
        *)
            # pass through
            echo "$event"
            ;;
        esac
    done
}

printf "fas fa-volume-mute\nfas fa-volume-down\nfas fa-volume-up\nfas fa-step-backward\nfas fa-play\nfas fa-step-forward\nfas fa-arrow-left\nfas fa-play-circle\nfas fa-arrow-right" |
    thqm --title="media" --style fa-grid "$@" |
    media_control
```

This will generate the following web page which can be used to control media on the host.

{{< figure src="/images/projects/thqm/thqm-media.png" align=center width="300px" >}}

> Check out the [examples folder](https://github.com/loiccoyle/thqm-rs/tree/main/examples) for more scripts.

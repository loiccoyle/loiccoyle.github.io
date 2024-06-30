---
author:
  - Loïc Coyle
title: '--help in the README'
date: '2024-06-30'
description: How to keep your CLI's help message up to date in the README
summary: How to keep your CLI's help message up to date in the README
categories:
  - Posts
tags:
  - Workflow
ShowToc: true
TocOpen: false
---
# Why?

When developing a CLI, it is helpful to show the `--help` message in your project's `README.md` to quickly show its features and options to a prospective user.

{{< figure src="/images/posts/cli_help_readme/phomo_help.png" title="Screenshot of phomo's help message in the project's readme" align=center link="https://github.com/loiccoyle/phomo" target="_blank" >}}

But it gets very annoying to have to paste in the readme every time you add an option to the CLI or tweak the help message in any way.

# How

One nifty solution is to automatically insert the help message using a workflow. I'll be showing how I set it up for my [{{< ico bootstrap github >}} loiccoyle/phomo](https://github.com/loiccoyle/phomo) repo, it'll need to be adapted for use in other projects.

## Markers

To achieve this, we need to leave markers within the `README.md` file to indicate where we want the help message to go.

We can just use `markdown` comments:

```md
...

<!-- help start -->
<!-- help end -->

...
```

> We need a start and end marker because the help message will be inserted in between. When we next update the readme, we need to override the content between the start and end markers.

## Inserting the help message

So we want to insert the help message in between the markers. This can be done any number of ways. I just use a small `awk` command.

````sh {hl_lines=[7,8,9,10]}
awk -i inplace '
  BEGIN { in_section = 0 }
  /^<!-- help start -->/ {
    in_section = 1
    print
    print ""
    print "```console"
    print "$ phomo -h"
    system("poetry run phomo -h")
    print "```"
    print ""
  }
  /^<!-- help end -->/ { in_section = 0 }
  !in_section
' README.md
````

With this scripts we:

- insert the code starting block fence with the `console` syntax highlighting
- add a line with `$ phomo -h` to show how to get the help message from the command line
- insert the actual help message by running `poetry run phomo -h`
- close the code block

> Make sure to use the same start and end markers in the `awk` command and the `README.md` file.

## Setup the workflow

I like to move this into a `Makefile` just to be able to run it manually.

````makefile
.PHONY: readme
readme:
  @awk -i inplace 'BEGIN { in_section = 0 } \
  /^<!-- help start -->/ { \
    in_section = 1; \
    print; \
    print ""; \
    print "```console"; \
    print "$$ phomo -h"; \
    system("poetry run phomo -h"); \
    print "```"; \
    print ""; \
  } \
  /^<!-- help end -->/ { in_section = 0 } \
  !in_section' README.md
````

To add it as a step in my CI/CD pipeline:

```yaml {hl_lines=[15]}
readme:
  runs-on: ubuntu-latest
  needs: ci
  steps:
    - uses: actions/checkout@v4
      with:
        token: ${{ secrets.BOT_ACCESS_TOKEN }}
    - name: Insstall poetry
      run: pipx install poetry
    - uses: actions/setup-python@v5
      with:
        python-version: "3.10"
        cache: "poetry"
    - run: poetry install
    - run: make readme
    - name: Commit changes
      uses: stefanzweifel/git-auto-commit-action@v5
      with:
        commit_message: "ci: Update readme"
        branch: ${{ github.head_ref }}
        commit_user_name: github-actions[bot]
        commit_user_email: github-actions[bot]@users.noreply.github.com
        commit_author: github-actions[bot] <github-actions[bot]@users.noreply.github.com>
```

In this workflow, we setup `python` and `poetry`, install the `phomo` package, then run the `make readme` recipe to insert the help message into the `README.md`. Finally, we push the changes, if there are any, back to the repo.

> You'll need to configure an access token as a secret to the repository. To do this, create a new token at <https://github.com/settings/tokens> with the `repo` scope. Add the token to the repo's secrets at `https://github.com/{user}/{repo}/settings/secrets/actions`.


This document is used to record the notes of reading the article[1].

- Environment: `busybox:stable`

    `docker run -it busybox:stable`

    > Cause I'm using Macos which uses BSD userland, so I'm using docker to get a busybox userland for having an environment aligned with the article.

- `(>&2 echo 'Already a shiv repository.')` looks like a nice way to put out warning information into the sterr.

- Use internals

    ``` bash
    #!/bin/sh

    set -ex

    shiv_bin="$(realpath "$0")"

    # Init a new repository
    if [ "$1" = '_init' ]; then
        if [ -d '.shiv' ]; then
            (>&2 echo 'Already a shiv repository.')
            exit 1
        fi

        mkdir '.shiv'

        # TODO: Form a tree of folders...

        exit 0
    fi

    # CLI parsing
    if [ "$1" = 'init' ]; then
        "$shiv_bin" '_init'
        exit 0
    else
        (>&2 echo 'Unknown command.')
        exit 1
    fi

    ```

    > `init` is by design exposed to user, `_init` is a service to `init`

    > It's interesting the whole script is organized around getting `shiv_bin` and pass parameter into it.

- Find **.shiv** directory

    ``` bash
    # Find the shiv repository
    if [ "$1" = '_find_shiv' ]; then
        # Search upward to sysroot.
        dir="$(pwd)"
        while [ ! -d "$dir"/.shiv ] && [ "$dir" != '/' ]; do
            dir="$(dirname "$dir")"
        done

        # If we found something, emit it.
        if [ -d "$dir"/.shiv ]; then
            echo "$dir"
            exit 0
        else
            exit 1
        fi
    fi
    ```

    > This logic is reused many many times.

- if condition based on script exit code


    ``` bash
    # Init a new repository
    if [ "$1" = '_init' ]; then
        if "$shiv_bin" '_find_shiv' >/dev/null; then
            (>&2 echo 'Already a shiv repository.')
            exit 1
        fi

        mkdir '.shiv'

        # TODO: Form a tree of folders...

        exit 0
    fi
    ```

# Reference

1. [Shiv - Let's Build a Version Control System!](https://shatterealm.netlify.app/programming/2021_01_02_shiv_lets_build_a_vcs)

    > This article describes the process of an engineer try to build a custom version control system.
# Argc

[![CI](https://github.com/sigoden/argc/actions/workflows/ci.yaml/badge.svg)](https://github.com/sigoden/argc/actions/workflows/ci.yaml)
[![Crates](https://img.shields.io/crates/v/argc.svg)](https://crates.io/crates/argc)

Easily create feature-rich CLIs in bash.

![demo](https://user-images.githubusercontent.com/4012553/228990851-fee5649f-aa24-4297-a924-0d392e0a7400.gif)

Argc lets you define your CLI through comments and focus on your specific code, without worrying about command line argument parsing, usage texts, error messages and other functions that are usually handled by a framework in any other programming language.

## Features

- Parsing user's command line and extracting:
  - Positional arguments (optional, required, default value, choices, repeated, multiple values),
  - Option arguments (optional, required, default value, choices, multiple values),
  - Flag arguments (repeated),
  - Comma-separated list of option and positional arguments,
  - Sub-commands (nesting).
- Rendering usage texts,  showing flags, options, positional arguments and sub-commands.
- Validating the arguments, printing error messages if the command line is invalid.
- Generating a single, standalone bash script without argc dependency.
- Generating man pages.
- Generating multi-shell completion scripts (require argc as completer).

## Install

### With cargo

```
cargo install argc
```

### Binaries on macOS, Linux, Windows

Download from [Github Releases](https://github.com/sigoden/argc/releases), unzip and add argc to your $PATH.

### GitHub Actions

[extractions/setup-crate](https://github.com/marketplace/actions/setup-crate) can be used to install just in a GitHub Actions workflow.

```yaml
- uses: extractions/setup-crate@v1
  with:
    owner: sigoden
    name: argc
```

## Usage

To write a command-line program with argc, we only need to do two things:

1. Describe options, flags, positional parameters and subcommands in comments.
2. Insert `eval "$(argc --argc-eval "$0" "$@")"` into script to let argc to parse command line arguments.

Write `example.sh`

```sh
# @flag --foo     Flag value
# @option --bar   Option value
# @arg baz*       Positional values

eval "$(argc --argc-eval "$0" "$@")"
echo foo: $argc_foo
echo bar: $argc_bar
echo baz: ${argc_baz[@]}
```

Run `./example.sh --foo --bar=xyz a b c`, you can see argc successfully parses arguments and generate variables with `argc_` prefix.

```
foo: 1
bar: xyz
baz: a b c
```

Run `./example.sh -h`, argc will print help information for you.

```
USAGE: example.sh [OPTIONS] [BAZ]...

ARGS:
  [BAZ]...  Positional values

OPTIONS:
      --foo        Flag value
      --bar <BAR>  Option value
  -h, --help       Print help
```

## Comment Decorator

Argc uses comments with a `JsDoc` inspired syntax to add functionality to the scripts at runtime.

This [grammar](./docs/grammar.md), known as a `comment decorator`, is a normal Bash comment followed by an `@` sign and a tag.

It's how the argc parser identifies configuration.

### @cmd

Define a subcommand.

```sh
# @cmd Upload a file
upload() {
  echo Run upload
}

# @cmd Download a file
download() {
  echo Run download
}
```

```
USAGE: prog <COMMAND>

COMMANDS:
  upload    Upload a file
  download  Download a file
```

### @alias

Add aliases for the subcommand.

```sh
# @cmd Run tests
# @alias t,tst
test() {
  echo Run test
}
```

```
USAGE: prog <COMMAND>

COMMANDS:
  test  Run tests [aliases: t, tst]
```

### @arg

Define a positional argument.

```sh
# @arg va
# @arg vb!                 required
# @arg vc*                 multi-values
# @arg vd+                 multi-values + required
# @arg vna <PATH>          value notation
# @arg vda=a               default
# @arg vdb=`_default_fn`   default from fn
# @arg vca[a|b]            choices
# @arg vcb[=a|b]           choices + default
# @arg vcc[`_choice_fn`]   choices from fn
# @arg vx~                 capture all remaining args
```

### @option

Define a option argument.

```sh
# @option    --oa                   
# @option -b --ob                   short
# @option -c                        short only
# @option    --oc!                  required
# @option    --od*                  multi-occurs
# @option    --oe+                  multi-occurs + required
# @option    --ona <PATH>           value notation
# @option    --onb <FILE> <FILE>    two-args value notations
# @option    --oda=a                default
# @option    --odb=`_default_fn`    default from fn
# @option    --oca[a|b]             choices
# @option    --ocb[=a|b]            choices + default
# @option    --occ[`_choice_fn`]    choices from fn
# @option    --oxa~                 capture all remaining args
```

### @flag

Define a flag argument.


```sh
# @flag     --fa 
# @flag  -b --fb         short
# @flag  -c              short only
# @flag     --fd*        multi-occurs
```

### @env

Define an environment variable.

```sh
# @env EA                 optional
# @env EB!                required
# @env EC=true            default
# @env EDA[dev|prod]      choices
# @env EDB[=dev|prod]     choices + default
```

### @meta

Add a metadata.

```sh
# @meta key [value]
```

| usage                        | scope  | description                                                            |
| :--------------------------- | ------ | :--------------------------------------------------------------------- |
| `@meta dotenv [<path>]`      | root   | Load a `.env` file from a custom path, if persent.                     |
| `@meta default-subcommand`   | subcmd | Set the current subcommand as the default.                             |
| `@meta inherit-flag-options` | root   | Subcommands will inherit the flags/options from their parent.          |
| `@meta no-inherit-env`       | root   | Subcommands won't inherit the environment variables from their parent. |
| `@meta symbol <param>`       | anycmd | Define a symbolic parameter, e.g. `+toolchain`, `@argument-file`.      |
| `@meta combine-shorts`       | root   | Short flags/options can be combined, e.g. `prog -xf => prog -x -f `.   |
| `@meta man-section <1-8>`    | root   | Override the default section the man page.                             |

### @describe / @version / @author

```sh
# @describe A demo cli
# @version 2.17.1 
# @author nobody <nobody@example.com>
```

```
prog 2.17.1
nobody <nobody@example.com>
A demo cli

USAGE: prog
```

<details>
<summary>

### Value Notation
</summary>

Value notation is used to describe value type of options and positional parameters.

```
# @option --target <FILE>
# @arg target <FILE>
```

Here are some value notation that will affect the shell completion.

- `FILE`/`PATH`: complete files
- `DIR`: complete directories

</details>

## Build

Build a single standalone bash script without argc dependency.

```
argc --argc-build <SCRIPT> [OUTPATH]
```

## Manpage

Generate man pages for your script.

```
argc --argc-mangen <SCRIPT> [OUTDIR]
```

## Completions

Argc provides shell completion for argc command and all the bash scripts powered by argc.

```
argc --argc-completions <SHELL> [CMDS]...
```

```
# bash (~/.bashrc)
source <(argc --argc-completions bash mycmd1 mycmd2)

# elvish (~/.config/elvish/rc.elv)
eval (argc --argc-completions elvish mycmd1 mycmd2 | slurp)

# fish (~/.config/fish/config.fish)
argc --argc-completions fish mycmd1 mycmd2 | source

# nushell (~/.config/nushell/config.nu)
argc --argc-completions nushell mycmd1 mycmd2 # update config.nu manually according to output

# powershell ($PROFILE)
Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
argc --argc-completions powershell mycmd1 mycmd2 | Out-String | Invoke-Expression

# xonsh (~/.config/xonsh/rc.xsh)
exec($(argc --argc-completions xonsh mycmd1 mycmd2))

# zsh (~/.zshrc)
source <(argc --argc-completions zsh mycmd1 mycmd2)

# tcsh (~/.tcshrc)
eval `argc --argc-completions tcsh mycmd1 mycmd2`
```

**Replace `mycmd1 mycmd2` with your argc scripts**.

Argc can be used as multiple shell completion engine. see [argc-completions](https://github.com/sigoden/argc-completions)

## Argcscript

Argc will automatically find and run `Argcfile.sh` unless `--argc-*` options are used to change this behavior.

What is the benefit?

- Can enjoy a handy shell completion.
- Can be invoked in arbitrarily sub-directory, no need to locate script file each time.
- As a centralized entrypoint/document for executing the project's bash scripts.
- Serves as [task runner](./docs/task-runner.md). Argcfile is to argc what Makefile is to make. 

You can use `argc --argc-create` to quickly create a boilerplate argcscript.

```
argc --argc-create [TASKS]...
```

![argcscript](https://github.com/sigoden/argc/assets/4012553/5130d9c5-90ff-478e-8404-3db6f55ba1d0)

## Parallel

argc provides features for running commands/functions in parallel.

```sh
argc --argc-parallel "$0" cmd1 arg1 arg2 ::: cmd2
```

The above command will run `cmd1 arg1 arg2` and `cmd2` in parallel. Functions running in parallel mode can still access the `argc_*` variable.

<details>
<summary>

# Windows

The only dependency of argc is bash. Developers under windows OS usually have [git](https://gitforwindows.org/) installed, and git has built-in bash. So you can safely use argc and GNU tools (grep, sed, awk...) under windows OS.

</summary>

## Make `.sh` file executable

If you want to run a `.sh` script file directly like a `.cmd` or `.exe` file, execute the following code in PowerShell.

```ps1
# Add .sh to PATHEXT
[Environment]::SetEnvironmentVariable("PATHEXT", [Environment]::GetEnvironmentVariable("PATHEXT", "Machine") + ";.SH", "Machine")

# Associate the .sh file extension with Git Bash
New-Item -LiteralPath Registry::HKEY_CLASSES_ROOT\.sh -Force
New-ItemProperty -LiteralPath Registry::HKEY_CLASSES_ROOT\.sh -Name "(Default)" -Value "sh_auto_file" -PropertyType String -Force
New-ItemProperty -LiteralPath 'HKLM:\SOFTWARE\Classes\sh_auto_file\shell\open\command' `
  -Name '(default)' -Value '"C:\Program Files\Git\bin\bash.exe" "%1" %*' -PropertyType String -Force
```

![image](https://github.com/sigoden/argc/assets/4012553/16af2b13-8c20-4954-bf58-ccdf1bbe23ef)

</details>

## License

Copyright (c) 2023-2024 argc developers.

argc is made available under the terms of either the MIT License or the Apache License 2.0, at your option.

See the LICENSE-APACHE and LICENSE-MIT files for license details.

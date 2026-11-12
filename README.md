# CTM Command Line Tool

A commandline tool for maintaining CTM objects very useful when building with CTM lambda functions

# Usage

```
ctm verify # will validate the files in your current directory
```

```
ctm deploy # will deploy your files assuming they are correct to upstream ctm servers
```

```
ctm create:lambda -h # will create a new directory and initialize a new git repo for you
usage: ctm create [-h] -n NAME [-p PATH]

options:
  -h, --help            show this help message and exit
  -n NAME, --name NAME  Project name
  -p PATH, --path PATH  Project path
```

```
ctm reset:credentials -n my-project -h # will reset the credentials stored for the current project
usage: ctm reset:credentials [-h] [-n NAME]

options:
  -h, --help            show this help message and exit
  -n NAME, --name NAME  Project name
```

example usage
```
ctm create:lambda -n get-email -p `pwd`
cd get-email
  # edit code.js

ctm verify
ctm deploy
```


# Development

Use an already-installed Python with `venv` and `pip`. The pinned
[PyInstaller 5.10.1](https://pypi.org/project/pyinstaller/5.10.1/) requires
Python >=3.7,<3.12; newer Python releases are not supported by this dependency
set. No exact Python version is recorded in this repository, CI, or version
files, so no speculative `.tool-versions` pin is supplied. Select a compatible
interpreter explicitly (for example `PYTHON=/absolute/path/to/python3 bin/setup`).
If using mise/asdf shims, disable automatic runtime installation before running
these commands, or use an absolute path to an installed interpreter.

| Command | Purpose |
| --- | --- |
| `bin/setup` | Create the existing `env/` virtualenv and install pinned requirements. |
| `bin/update` | Repeat dependency sync after pulling; creates `env/` if absent. |
| `bin/lint` | Check Python and developer shell script syntax without executing the CLI. |
| `bin/doctor` | Read-only Python/dependency diagnostics; nonzero for missing or mismatched dependencies. |
| `bin/build [PyInstaller options]` | Build local artifacts in `build/` and `dist/` using `ctm.spec`. |

Scripts locate this checkout independently of the caller's working directory.
`PYTHON` selects the bootstrap interpreter only when `env/` does not exist;
`bin/lint` also accepts it for dependency-free syntax checking. Setup/update
install the versions in `requirements.txt` without upgrading pins, pulling Git,
accessing CTM, or running migrations. They accept no arguments. Build forwards
options to PyInstaller; it does not sign, upload, or publish artifacts. The
existing spec describes the macOS build with `entitlements.plist`; building on
other platforms has not been validated.

There is no automated test suite or established formatter in this repository,
so no `bin/test` is provided. Syntax checks do not replace functional tests.
This repository owns a CLI, not a local app server or a release deployment
workflow, so no `bin/run.sh` or `bin/deploy.sh` is added. The existing `ctm`
interface is unchanged. To run it from source, activate this checkout's `env/`
and invoke the absolute path to `ctm` from the project you intend to work on.
`ctm verify` additionally requires Node.js on PATH for JavaScript projects and
uses the installed `yamllint`; no tested Node.js version is recorded here.
Commands that use CTM require network access and OS keyring support. Doctor
never reads credentials or contacts CTM, and does not verify remote access.

## Manual macOS signing

After `bin/build`, the existing optional signing procedure is:

```sh
security find-identity -p basic -v
codesign --deep --force --options=runtime --entitlements ./entitlements.plist --sign YOUR_SIGNING_IDENTITY --timestamp ./dist/ctm/ctm
```

Signing requires your own identity and is deliberately separate from the build.

# CopyCat CLI Reference

_Generated from `copycat --help`. See [COMMANDS.md](COMMANDS.md) for an example-driven guide._

```
CopyCat copies files and directories locally or between machines (SFTP and more), with retries, exclude rules, conflict handling, and post-transfer validation. The CLI is the default interface; an optional Tauri GUI is available via `copycat install-gui`.

Usage: copycat.exe [OPTIONS] <COMMAND>

Commands:
  copy         Copy a file or directory (local or remote)
  paste        Paste from the system clipboard or an internal buffer
  remote       Manage and perform remote transfers
  config       View and edit configuration
  api          Run or manage the gRPC API server
  install-gui  Install the optional CopyCat GUI (may require elevated privileges)
  completions  Generate a shell completion script (write to stdout)
  man          Generate the man page (write to stdout)
  help         Print this message or the help of the given subcommand(s)

Options:
  -r, --retry <N>
          Number of retries for fallible operations (default: from config, 2)

  -e, --exclude <PATTERN>
          Exclude patterns/paths (comma-separated, repeatable)

      --exclude-from <FILE>
          Read additional exclude patterns from a file (one per line)

      --replace
          Overwrite existing destinations instead of renaming

      --confirm
          Prompt before overwriting an existing destination

      --dry-run
          Plan the operation without modifying anything

      --resume
          Resume partially-transferred destination files instead of recopying

  -j, --jobs <N>
          Maximum concurrent file transfers (default 1 = sequential)

      --limit-rate <BYTES>
          Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)

      --identity <FILE>
          Private key file for remote (SFTP) authentication

      --agent
          Use the SSH agent for remote authentication

  -v, --verbose
          Emit verbose, debug-level output

  -h, --help
          Print help (see a summary with '-h')

  -V, --version
          Print version
```

## `copycat copy`

```
Copy a file or directory (local or remote)

Usage: copycat.exe copy [OPTIONS] <SOURCE> <DESTINATION>

Arguments:
  <SOURCE>       Source path or remote URI (e.g. `sftp://user@host/path`)
  <DESTINATION>  Destination path or remote URI

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat paste`

```
Paste from the system clipboard or an internal buffer

Usage: copycat.exe paste [OPTIONS] [DESTINATION]

Arguments:
  [DESTINATION]  Destination path to paste into (defaults to current directory) [default: .]

Options:
      --buffer <NAME>        Read from a named internal buffer instead of the system clipboard
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat remote`

```
Manage and perform remote transfers

Usage: copycat.exe remote [OPTIONS] <COMMAND>

Commands:
  push  Copy from a local path to a remote URI
  pull  Copy from a remote URI to a local path
  help  Print this message or the help of the given subcommand(s)

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat config`

```
View and edit configuration

Usage: copycat.exe config [OPTIONS] <COMMAND>

Commands:
  path  Print the resolved config file path
  init  Write a default config file (fails if one exists unless `--force`)
  get   Print a single config value by key (e.g. `retries`, `api.port`)
  set   Set a single config value by key
  help  Print this message or the help of the given subcommand(s)

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat api`

```
Run or manage the gRPC API server

Usage: copycat.exe api [OPTIONS] <COMMAND>

Commands:
  serve  Start the gRPC API server (local-only by default)
  help   Print this message or the help of the given subcommand(s)

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat completions`

```
Generate a shell completion script (write to stdout)

Usage: copycat.exe completions [OPTIONS] <SHELL>

Arguments:
  <SHELL>  Target shell [possible values: bash, elvish, fish, powershell, zsh]

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat man`

```
Generate the man page (write to stdout)

Usage: copycat.exe man [OPTIONS]

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

## `copycat install-gui`

```
Install the optional CopyCat GUI (may require elevated privileges)

Usage: copycat.exe install-gui [OPTIONS]

Options:
  -r, --retry <N>            Number of retries for fallible operations (default: from config, 2)
  -e, --exclude <PATTERN>    Exclude patterns/paths (comma-separated, repeatable)
      --exclude-from <FILE>  Read additional exclude patterns from a file (one per line)
      --replace              Overwrite existing destinations instead of renaming
      --confirm              Prompt before overwriting an existing destination
      --dry-run              Plan the operation without modifying anything
      --resume               Resume partially-transferred destination files instead of recopying
  -j, --jobs <N>             Maximum concurrent file transfers (default 1 = sequential)
      --limit-rate <BYTES>   Limit transfer bandwidth in bytes/second (e.g. 1048576 for 1 MiB/s)
      --identity <FILE>      Private key file for remote (SFTP) authentication
      --agent                Use the SSH agent for remote authentication
  -v, --verbose              Emit verbose, debug-level output
  -h, --help                 Print help
  -V, --version              Print version
```

# fpdc
**TL;DR**
`fpdc` is a convenience wrapper for `docker compose` written as a shell script

**fpdc** stands for _Free-Port Docker Compose_ promising a simple functionality: not having to worry about what ports are open and which ones are taken on the host when spinning up services for trail and testing purposes.

The current wrapper is in an early alpha stage where I am just experimenting with _comfort features_ that I personally like to have. Depending on what the future holds, the wrapper can be entirely re-written or changed based on the reaction from the community.

## Introduction

In principle, a wrapper is just an _alias_ with added features. Namely, you should be able to use `fpdc` in the same way you would use `docker compose`.

For example, one can replace `docker compose -f compose.yaml -f compose.traefik.yaml up -d` with `fpdc -f compose.yaml -f compose.traefik.yaml up -d` and things should ideally, just work.

How much this rule holds, is just a question of how far I get along growing this tool for it to become something that people other than just myself would want to use.

### Added cli options

The wrapper supports two non-standard docker compose "commands". These commands can only be mentioned prior to other (standard) docker compose arguments and cli options:
```bash
fpdc [interactive] [dry-run] ...
```

#### `interactive`

The interactive command will discard any assumptions about where to find the docker compose files, and instead, launch an editor interactively.

> [!NOTE]  
> The preferred editor can be specified in a manner similar to many other cli tools using the `$EDITOR` env variable. In the absence of such variable the tool will look for common editors in the following order: vscode (using `code`), then `nano`, then `vim`. This order of precedence is dictated by favoring more novice users, but can be challenged if more people decide to use the wrapper.

#### `dry-run`

This command will replace the command requested with `docker compose config`, resulting in the parsing and the _fpdc transformations_ being applied to a temporary file, for debugging or troubelshooting purposes.

### Features

#### Port-replacement

The main feature that this wrapper offers is embedded in its name, which is the _Free-Port transformation_.

This transformation will peak into the parsed docker compose config and replace destination ports with alternative free ports if the original specified ports were currently busy on the host. The algorithm for choosing the new port is sequential. It will continue to increment the port number until a free port is found.

> [!TIP]
> Maybe overkill, but the wrapper uses sorting and binary search over the entire set of used ports on the host to ensure efficiency when looking for available ones. This means that theoretically, you could have all the ports from 80 to 65534 occupied, and as you request port 80 via `fpdc`, the script should be able to identify 65535 as the right port to use relatively efficiently.

> [!IMPORTANT]  
> Currently the `fpdc` wrapper only supports a traditional port range between 1-65535 (meaning that requesting a port of 65535 with this particular port being occupied on the host will lead to a failure).

#### Secret generation using `pwgen`

If you supply a docker compose config that contains _external secrets_, `fpdc` leverages this unsupported aspect of the specification to generate secrets on the fly using `pwgen`, interactively, while storing them in a folder `secrets` in the current working directory.

On the first time a secret is generated, the `fpdc` wrapper will ask for a choice between three types of secrets: a secret username, a secret password, and a secret long password. These are just translated to `pwgen` running options and parameters.

> [!TIP]
> Currently, secret usernames are 8-12 characters long with only letters and numbers, whereas passwords use symbols. Regular passwords are 25 characters long, and long passwords are 50 characters long, but these things are easy to change. I just started with something that made sense to me as the sole user of the wrapper in the moment.


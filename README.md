# workon

`workon` launches Codex or Claude Code inside a persistent tmux session on a remote VPS.

- `codexl` means **Codex launch**.
- `claudel` means **Claude launch**.

```sh
codexl myproject
claudel myproject
```

The scripts discover running EC2 instances tagged `Workon=true`, find the requested project under `~/apps` on those servers, and connect to the best match over SSH. The coding client runs on the VPS, not on your local computer.

## Requirements

On your local computer:

- Bash, SSH, and the AWS CLI
- AWS CLI credentials that can call `ec2:DescribeInstances`
- SSH configuration for each VPS

On each VPS:

- Bash and tmux
- Codex CLI and/or Claude Code
- Projects stored as directories under `~/apps`

Each running EC2 instance must have a `Workon=true` tag and a `Name` tag whose value is an SSH host configured in `~/.ssh/config`. For example:

```sshconfig
Host devbox
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/devbox
```

The corresponding instance would have `Name=devbox` and `Workon=true` tags.

## Install

Clone the repository somewhere permanent, then link the launchers into a directory on your `PATH`:

```sh
git clone https://github.com/QualityCopperShovel/workon.git ~/.local/share/workon
mkdir -p ~/.local/bin
ln -s ~/.local/share/workon/codexl ~/.local/bin/codexl
ln -s ~/.local/share/workon/claudel ~/.local/bin/claudel
```

Make sure `~/.local/bin` is on your `PATH`. To update later:

```sh
git -C ~/.local/share/workon pull --ff-only
```

## Authenticate on the VPS

Authentication belongs on every VPS where the client will run. SSH to each VPS once and authenticate the installed clients there.

For Codex, run:

```sh
codex login
```

Choose ChatGPT sign-in to use access included with an eligible ChatGPT subscription. On a headless VPS, device-code sign-in is usually the easiest option:

```sh
codex login --device-auth
```

For Claude Code, run:

```sh
claude
```

Choose Claude.ai sign-in to use a Pro or Max subscription.

Both tools also support API-based authentication, but API usage is billed separately from ChatGPT or Claude subscriptions. Do not put API keys in this repository. Prefer the clients' interactive login and credential storage; if you use environment variables instead, configure them securely for the VPS account that runs tmux.

## Usage

Launch the best matching project with Codex or Claude:

```sh
codexl myproject
claudel myproject
```

Names are matched case-insensitively and fuzzily across all discovered servers. To restrict the search to one server, prefix the query with that server's name or an unambiguous prefix:

```sh
codexl devbox/myproject
claudel dev/myproject
```

Use a number to open a separate tmux session slot for the same project:

```sh
codexl myproject 2
claudel myproject 2
```

Re-running the same command reconnects to its existing tmux session. Codex sessions are also associated with their project and slot so they can be resumed after the client exits.

## Host discovery cache

EC2 host discovery is cached for five minutes in `~/.cache/workon/hosts`. If AWS is temporarily unavailable, a valid older cache is used as a fallback. Set `WORKON_CACHE_TTL` to change the cache lifetime in seconds.

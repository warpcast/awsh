# `awsh`

`awsh` is `ssh` for [AWS](https://aws.amazon.com/) — authenticate and open a secure shell on your instances without SSH keys.

`awsh` behaves like `ssh` but uses AWS IAM for authentication.

## Installation

### Requirements

`awsh` is a simple Bash script that can run anywhere, but it expects the following executables in your `PATH`:

- `aws`
- `fzf`
- `ssh`

If you install via Homebrew, these dependencies are taken care of.

### via Homebrew/Linuxbrew
```
brew install warpcast/formulae/awsh
```

Otherwise, you can run:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/warpcast/awsh/refs/heads/main/install.sh)"
```

## Usage

### Connect to any instance

If you specify no arguments, `awsh` will allow you to select from a list of accessible instances:

![awsh demo](demo.gif)

`awsh` assumes you are attempting to SSH as `ec2-user` in this mode. If you want to connect as a specific user, see below, or set the `SSH_USER` environment variable.

### Connect to a specific instance

If you know the username + hostname you want to connect with, specify the same command line options just like `ssh`:

```
awsh [options] user@host [command [argument ...]]
```

## Caveats

`awsh` assumes that you trust AWS. If you don't, don't use it.

## How it works

At a high level:

1. Generates a temporary (one-time-use) SSH key.
2. Uses `ec2-instance-connect:SendSSHPublicKey` to add the key to the host's `~/.ssh/authorized_keys` for the specified user.
3. Opens a tunnel with the AWS CLI and runs a `ssh` command via that tunnel.

## Development

### Pushing a new release

1. Once ready, simple push a tag in the format of `vX.Y.Z` and a release will automatically be created and an artifact uploaded.
   ```
   version=1.2.3; git tag -s -a v$version -m v$version
   git push --tags
   ```
   This will trigger the automated [release workflow](https://github.com/warpcast/awsh/actions/workflows/release.yml).

2. You should then update the [`awsh` Homebrew formula](https://github.com/warpcast/homebrew-formulae/blob/main/awsh.rb) to point at the latest release.

## License

This project is released under the [MIT license](LICENSE).

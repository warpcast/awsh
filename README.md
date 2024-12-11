# `awsh`

`awsh` is `ssh` for [AWS](https://aws.amazon.com/) — authenticate and open a secure shell on your instances without SSH keys.

`awsh` behaves like `ssh` but uses AWS IAM for authentication.

## Requirements

`awsh` is a simple Bash script that can run anywhere, but it expects the following executables in your `PATH`:

- `bash` — to run the script itself. Should already be on the vast majority of systems
- `aws` — `awsh` assumes that `aws` will be able to authenticate with your current shell environment
- `fzf` —  only if you don't specify an explict username and host to connect to
- `ssh`

If you install via Homebrew, dependencies are installed automatically.

### Authentication

Your local environment needs to be set up so that you can be authenticated in IAM with the following permissions:

Your permissions must include a policy similar to the example below:

**WARNING: Below example grants access to all instances. You likely want to introduce conditions (see second example below).**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2InstanceConnect",
      "Action": [
        "ec2:DescribeInstances",
        "ec2-instance-connect:SendSSHPublicKey"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

If you want to constrain which instances the user can connect to, add a condition.
The following example grants EC2 Connect Access to all instances with a tag of `ssh-access = true`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DescribeInstances",
      "Action": [
        "ec2:DescribeInstances"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "ConnectToInstancesWithTag",
      "Effect": "Allow",
      "Action": "ec2:SendSSHPublicKey",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/ssh-access": "true"
        }
      }
    }
  ]
}
```

## Installation

### Homebrew/Linuxbrew
```
brew install warpcast/formulae/awsh
```

### Generic *nix

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

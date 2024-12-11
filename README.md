# `awsh`

`awsh` is `ssh` for [AWS](https://aws.amazon.com/) – authenticate and open a secure shell on your instances without SSH keys.

`awsh` behaves like `ssh` but uses AWS IAM for authentication.

## Installation

### Requirements

`awsh` is a simple Bash script that can run anywhere, but it expects the following executables in your `PATH`:

- `aws`
- `fzf`
- `ssh`

If you install via Homebrew, these dependencies are taken care of.

**via Homebrew/Linuxbrew**
```
brew install warpcast/formulae/awsh
```

Otherwise just copy `awsh` into your PATH (`/usr/local/bin` / etc.) and make sure it's executable.

## Usage

### Connect to any instance

If you specify no arguments, `awsh` will prompt with a fuzzy search for an instance:

```
$ awsh
No user@hostname specified, fetching available instances...
i-00c3b7b05ca2906fe     10.0.5.51       job-processor                                                                                                                    │
i-05a13440331eeb2df     10.0.5.84       app                                                                                                                              │
i-0856ed186c2f0416d     10.0.5.96       ws                                                                                                                               │
3/3 ────────────────────────────────────────────────
```

`awsh` assumes you are attempting to SSH as `ec2-user` in this mode. If you want to connect as a specific user, see below, or set the `SSH_USER` environment variable.

### Connect to a specific instance

If you know the username + hostname you want to connect with, specify the same command line options just like `ssh`:

```
awsh [options] user@host [command [argument ...]]
```

## Caveats

A core assumption with using AWS EC2 Instance Connect (and AWS in general) is that traffic within their private network is **not** susceptible to IP spoofing.
Therefore host verification isn't practically useful–and in an environment with EC2 instances constantly cycling through a pool of reused IPs–it is annoying to maintain a `~/.ssh/known_hosts` file.
For this reason, `awsh` takes an opinionated approach by disabling host key verification.
You will see a warning emitted every time you run `awsh` saying the host key was permanently added, but it can be ignored since the host key wasn't actually saved–it was written to `/dev/null`).

## How it works

At a high level:

1. Generates a temporary (one-time-use) SSH key.
2. Uses `ec2-instance-connect:SendSSHPublicKey` to add the key to the host's `~/.ssh/authorized_keys` for the specified user.
3. Opens a tunnel with the AWS CLI and runs a `ssh` command via that tunnel.

## License

This project is released under the [MIT license](LICENSE).
# Porta dev-tools
These are dev tools to play with [3scale/porta](https://github.com/3scale/porta) in local environment. They include:

1. a CLI to ease commands such as starting the Rails server, launch dependencies, deploy to OpenShift, etc (see the [full list](#supported-commands) of commands)
2. [Porxy](porxy/README.md), a porksy reverse proxy to porta. Because [APIcast](https://github.com/3scale/apicast) has it's own DNS resolution that we can't override with `/etc/hosts`, we need this reverse proxy.

> :warning: Porta dev-tools are for development purposes and cannot in any way be considered a replacement for Red Hat's official recommendations to work with 3scale.

> :warning: Porta dev-tools commands and settings were tested under Mac OS X with zsh shell. You may need adapt them for your own environment.

> :warning: Porta dev-tools use [Podman](https://podman.io) as the container tool by default, but it can also be used with Docker.

## Requirements
Porta dev-tools assume you can already run 3scale/porta locally with whatever DBMS currently supported (MySQL, PostgreSQL, Oracle). See [installation instructions](https://github.com/3scale/porta/blob/master/INSTALL.md) for help.

A running instance of Redis is expected to be attending to port 6379, as well as a DNS resolver capable of handling wildcard domains, such as dnsmasq. These are usual requirements of 3scale/porta, but may be used as well by other components triggered with some of Porta dev-tools commands.

`redis-cli` is needed for `porta reset`. If you are running Redis in a container, you may create a script with that name in PATH with a content like `podman exec redis redis-cli "$@"`.

For Zync, please see [Quickstart guide](https://github.com/3scale/zync/blob/master/doc/Quickstart.md) for PostgreSQL setup.

If you are using dnsmasq or another DNS server locally, make sure to include the following two DNS records. They will be particularly useful to run Porta along with [3scale/APIcast](https://github.com/3scale/apicast).

```conf
# address=/example.com/127.0.0.1 # for some old tests
address=/.localhost/127.0.0.1
address=/.apicast.dev/127.0.0.1 # default porta API product
```

For [3scale/apisonator](https://github.com/3scale/apisonator), default settings should be alright but you can modify some variables with `settings.yml` below.

Apart from the aforementioned requirements, specific commands of Porta dev-tools may additionally require:

- [Podman](https://podman.io) (or another container tool)
- [OpenShift CLI Tools](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_cli/getting-started-cli.html)
- A clone of the [3scale/3scale-operator](https://github.com/3scale/3scale-operator) repo
- A public OpenShift cluster where to deploy 3scale

## Install

```shell
export PORTA_DEV_TOOLS_PATH=/usr/local/porta-dev-tools
git clone git@github.com:guicassolato/porta-dev-tools.git $PORTA_DEV_TOOLS_PATH
echo "export PATH=$PORTA_DEV_TOOLS_PATH/bin:\$PATH">>~/.zshrc
```

### Settings/defaults
You may want to copy and edit the `settings.yml` file. This file holds default values to options of the Porta dev-tools commands, such as the paths to the Porta and other repos locally, secret keys, etc.

```shell
cp $PORTA_DEV_TOOLS_PATH/config/examples/settings.yml $PORTA_DEV_TOOLS_PATH/config/
```

## Usage

General syntax:

```shell
porta CMD [options]
```

### Supported commands

| Command  | Description                                                                                                       |
| ---------|-------------------------------------------------------------------------------------------------------------------|
| server   | Starts the Rails server locally                                                                                   |
| sidekiq  | Starts a Sidekiq worker locally                                                                                   |
| reset    | Resets Porta's databases (Redis and DBMS)                                                                         |
| data     | Generates fake data (uses the Admin API, server must be running)                                                  |
| assets   | Removes node_modules and precompile assets again                                                                  |
| test     | Bundle execs a Porta's Rails test file                                                                            |
| cuke     | Bundle execs a Porta's Cucumber test file                                                                         |
| deps     | Runs components that Porta depends upon – (in podman) Apisonator, APIcast and porxy; (daemonized) Zync and Sphinx |
| resync   | Resyncs Porta with Apisonator (Sidekiq and Apisonator must both be running)                                       |
| build    | Builds Porta for OpenShift                                                                                        |
| push     | Pushes latest `system-os` podman image to quay.io                                                                 |
| deploy   | Deploys 3scale to an OpenShift devel cluster, fetching images from quay.io                                        |
| help     | Prints the list of available commands                                                                             |

You can get a full list of the supported commands by running:

```shell
porta help
```

### Common options

The following options are available with all commands:

| Option      | Description                                                                              |
| ------------|------------------------------------------------------------------------------------------|
| `--help`    | Prints the specific help fo rthe command, which includes a list of the supported options |
| `--dry-run` | Prints the commands to STDOUT instead of executing them                                  |
| `--verbose` | Prints every command executed                                                            |

## Examples

### Workflow to run porta and dependencies locally

```shell
porta reset
porta deps
porta sidekiq  # hijacks the shell
porta resync
porta server   # hijacks the shell
```

### Workflow to build and deploy porta to OCP

```shell
porta build
porta push
porta deploy --watch
```

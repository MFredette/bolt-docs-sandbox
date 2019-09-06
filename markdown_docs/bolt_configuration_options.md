---
author: Michelle Fredette <michelle.fredette@puppet.com\>
---

# Bolt configuration options

Your Bolt configuration file can contain global and transport options.

## Sample Bolt configuration file

```
modulepath: "~/.puppetlabs/bolt-code/modules:~/.puppetlabs/bolt-code/site-modules"
inventoryfile: "~/.puppetlabs/bolt/inventory.yaml"
concurrency: 10
format: human
ssh:
  host-key-check: false
  private-key: ~/.ssh/bolt_id
  user: foo 
  interpreters:
     rb: /home/foo/.rbenv/versions/2.5.1/bin/ruby
```

## Global configuration options

-   `color`: Whether to use colored output when printing messages to the console.

-   `concurrency`: The number of threads to use when executing on remote nodes. Default is `100`.

-   `format`: The format to use when printing results. Options are `human` and `json`. Default is `human`.

-   `hiera-config`: Specify the path to your Hiera config. The default path is `hiera.yaml` inside the [Bolt project directory](bolt_project_directories.md#).

-   `interpreters`: A map of an extension name to the absolute path of an executable, enabling you to override the shebang defined in a task executable. The extension can optionally be specified with the `.` character \(`.py` and `py` both map to a task executable `task.py`\) and the extension is case sensitive. The transports that support interpreter configuration are `docker`, `local`, `ssh`, and `winrm`. When a node's name is `localhost`, Ruby tasks run with the Bolt Ruby interpreter by default. This example demonstrates configuring Python tasks to run with a `python3` interpreter: :

```
interpreters:
  py: /usr/bin/python3
```

-   `inventoryfile`: The path to a structured data inventory file used to refer to groups of nodes on the command line and from plans. The default path for the inventory file is `inventory.yaml` inside the [Bolt project directory](bolt_project_directories.md#).

-   `modulepath`: The module path for loading tasks and plan code. This is either an array of directories or a string containing a list of directories separated by the OS-specific `PATH` separator. The default path for modules is `modules:site-modules:site` inside the [Bolt project directory](bolt_project_directories.md#).

-   `puppetfile`: A map containing options for the `bolt puppetfile install` command.

-   `save-rerun`: Specify whether to update `.rerun.json` in the [Bolt project directory](bolt_project_directories.md#). If your target names include passwords, set this value to false to avoid writing passwords to disk.

-   `transport`: Specify the default transport to use when the transport for a target is not specified in the URL or inventory. The valid options for transport are `docker`, `local`, `pcp`, `ssh`, and `winrm`.


## SSH transport configuration options

-   `connect-timeout`: How long Bolt should wait when establishing connections.

-   `host-key-check`: Whether to perform host key validation when connecting over SSH. Default is `true`.

-   `password`: Login password.

-   `port`: Connection port. Default is `22`.

-   `private-key`: The path to the private key file to use for SSH authentication.

-   `proxyjump`: A jump host to proxy SSH connections through, and an optional user to connect with, for example: jump.example.com or user1@jump.example.com.

-   `run-as`: A different user to run commands as after login.

-   `run-as-command`: The command to elevate permissions. Bolt appends the user and command strings to the configured run as a command before running it on the target. This command must not require an interactive password prompt, and the `sudo-password` option is ignored when `run-as-command` is specified. The run-as command must be specified as an array.

-   `sudo-password`: Password to use when changing users via `run-as`.

-   `tmpdir`: The directory to upload and execute temporary files on the target.

-   `user`: Login user. Default is `root`.


## OpenSSH configuration options

In addition to the SSH transport options defined in Bolt configuration files, some additional SSH options are read from OpenSSH configuration files, including `~/.ssh/config`, `/etc/ssh_config`, and `/etc/ssh/ssh_config`. Not all OpenSSH configuration values have equivalents in Bolt.

These are the options configurable in OpenSSH files:

-   `User`

-   `Port`

-   `UserKnownHostsFile`

-   `Ciphers`: Ciphers allowed in order of preference. Multiple ciphers must be comma-separated.

-   `Compression`: Whether to use compression.

-   `CompressionLevel`: Compression level to use if compression is enabled.

-   `GlobalKnownHostsFile`: Path to global host key database.

-   `HostKeyAlgorithms`: Host key algorithms that the client wants to use in order of preference.

-   `HostKeyAlias`: Use alias instead of real hostname when looking up or saving the host key in the host key database file.

-   `IdentitiesOnly`: Use only identity key in SSH config even if ssh-agent offers others.

-   `HostName`: Host name to log.

-   `IdentityFile`: File in which user's identity key is stored.

-   `Port`: SSH port.

-   `UserKnownHostsFile`: Path to local user's host key database.


**Note:** For OpenSSH configuration options with direct equivalents in Bolt, such as user and port, the settings in Bolt config take precedence.

To illustrate, consider this example:

`inventory.yaml`

```
nodes:
  - name: host1.example.net
    config:
      transport: ssh
      ssh:
        host-key-check: true
        port: 22
        private-key: /.ssh/id_rsa-example
```

`ssh.config`

```
Host *.example.net
  UserKnownHostsFile=~/.ssh/known_hosts
  User root
  Port 444
```

In this example, the SSH connection is configured to use the user and known hosts file defined in OpenSSH config and the port defined inBolt config.

**Note:** The `host-key-check` option must be set in Bolt config because the `StrictHostKeyChecking` OpenSSH configuration value is ignored.

When using the SSH transport, Bolt also interacts with the ssh-agent for SSH key management. The most common interaction is to handle password protected private keys. When a private key is password protected it must be added to the ssh-agent in order to be used to authenticate Bolt SSH connections.

## WinRM transport configuration options

-   `cacert`: The path to the CA certificate.

-   `connect-timeout`: How long Bolt should wait when establishing connections.

-   `extensions`: List of file extensions that are accepted for scripts or tasks. Scripts with these file extensions rely on the target node's file type association to run. For example, if Python is installed on the system, a `.py` script should run with `python.exe`. The extensions .`ps1`, `.rb`, and `.pp` are always allowed and run via hard-coded executables.

-   `file-protocol`: Which file transfer protocol to use. Either `winrm` or `smb`. Using `smb` is recommended for large file transfers. Default is `winrm`.

**Note:** The SMB file protocol is experimental and is currently unsupported in conjunction with SSL, given that only SMB2 is currently implented.

-   `password`: Login password. Required.

-   `port`: Connection port. Default is `5986`, or `5985` if `ssl: false`.

-   `smb-port`: With `file-protocol` set to `smb`, this is the port to establish a connection on. Default is `445`.

-   `ssl`: When `true`, Bolt uses secure https connections for WinRM. Default is `true`.

-   `ssl-verify`: When true, verifies the targets certificate matches the `cacert`. Default is `true`.

-   `tmpdir`: The directory to upload and execute temporary files on the target.

-   `user`: Login user. Required.


## PCP transport configuration options

-   `cacert`: The path to the CA certificate.

-   `service-url`: The URL of the orchestrator API.

-   `task-environment`: The environment orchestrator should load task code from.

-   `token-file`: The path to the token file.

    `job-poll-interval`: Set interval to poll orchestrator for job status.

    `job-poll-timeout`: Set time to wait for orchestrator job status.


## Local transport configuration options

-   `run-as`: A different user to run commands as after login.

-   `run-as-command`: The command to elevate permissions. Bolt appends the user and command strings to the configured run as a command before running it on the target. This command must not require an interactive password prompt, and the `sudo-password` option is ignored when `run-as-command` is specified. The run-as command must be specified as an array.

-   `sudo-password`: Password to use when changing users via `run-as`.

-   `tmpdir`: The directory to copy and execute temporary files.


## Docker transport configuration options

**Note:** The Docker transport is experimental because the capabilities and role of the Docker API might change.

-   `service-options`: A hash of options to configure the Docker connection. This option is necessary only if you're using a non-default URL. See [https://github.com/swipely/docker-ap](https://github.com/swipely/docker-api)i for supported options.

-   `service-url`: URL of the Docker host used for API requests. Defaults to local via a Unix socket at `unix:///var/docker.sock`.

-   `shell-command`: A shell command any Docker exec commands should be wrapped in, for example, `bash -lc`.

-   `tmpdir`: The directory to upload and execute temporary files on the target.

-   `tty`: When `true`, enable tty on Docker exec commands. Default is `false`.


## Remote transport configuration options

**Note:** The remote transport is experimental. Its configuration options and behavior might change between Y releases.

The remote transport can accept arbitrary options depending on the underlying remote target, for example `api-token`.

-   `run-on`: The proxy target that the task should execute on. Default is `localhost`.


## Log file configuration options

Capture the results of your plan runs in a log file.

-   `log`: the configuration of the log file output. This option includes the following properties:

-   `append` add output to an existing log file. Available for only for logs output to a filepath. Your options are `true` \(default\) and `false`.
-   `console` or `path/to.log`: the location of the log output.
-   `level`: the type of information in the log. Your options are `debug`, `info`, `notice`, `warn`, and `error`.


```
log:
  console:
    level: info
  ~/.bolt/debug.log:
    level: debug
    append: false

```

## Puppetfile configuration options

The `puppetfile` section configures how modules are retrieved when running `bolt puppetfile install`.

-   `proxy`: The HTTP proxy to use for Git and Forge operations.

-   `forge`: A subsection that can have its own `proxy` setting to set an HTTP proxy for Forge operations only, and a `baseurl` setting to specify a different Forge host.


**Parent topic:**[Configuring Bolt](configuring_bolt.md)

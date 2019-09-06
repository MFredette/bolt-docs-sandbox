---
author: Jean Bond <jean@puppet.com\>
---

# Known issues

Known issues for the Bolt 1.x release series.

## JSON strings as command arguments might require additional escaping in PowerShell

When passing complex arguments to tasks with `--params`, JSON strings \(typically created with the `ConvertTo-Json` cmdlet\) might require additional escaping. In some cases, you can use the PowerShell stop parsing symbol `--%` as a workaround. \([BOLT-1130](https://tickets.puppetlabs.com/browse/BOLT-1130)\)

## SSH keys generated with ssh-keygen from OpenSSH 7.8+ fail

OpenSSH 7.8 switched to generating private keys with its own format rather than the OpenSSL PEM format. The Bolt SSH implementation assumes any key using the OpenSSH format uses ed25519, resulting in false errors such as:

```
 OpenSSH keys only supported if ED25519 is available net-ssh requires the following gems for ed25519 support: * ed25519 (>= 1.2, < 2.0) * bcrypt_pbkdf (>= 1.0, < 2.0) See https://github.com/net-ssh/net-ssh/issues/565 for more information Gem::LoadError : "ed25519 is not part of the bundle. Add it to your Gemfile."
```

or

```
Failed to connect to HOST: expected 64-byte String, got NUM
```

As a workaround, you can generate new keys with the ssh-keygen `-m PEM` flag. For existing keys, you can try exporting keys from the OpenSSH format using the `-e` option, although export is not implemented for all private key types. \([BOLT-920](https://tickets.puppetlabs.com/browse/BOLT-920)\)

## Commands fail in remote Windows sessions

Interactive tools fail when run in a remote PowerShell session. For example, using `--password` to prompt for a password when running Bolt triggers an error. As a workaround, consider putting the password in `bolt.yaml` or an inventory file, or passing the password on the command line. \([BOLT-1075](https://tickets.puppetlabs.com/browse/BOLT-1075)\)

## No Kerberos support

A license incompatibility with other components distributed with Bolt prevents authenticating with Kerberos over SSH using the net-ssh-krb gem. \([BOLT-980](https://tickets.puppetlabs.com/browse/BOLT-980)\)

Kerberos over WinRM, both from Windows and non- Windows hosts, is also not supported. \([BOLT-126](https://tickets.puppetlabs.com/browse/BOLT-126)\)

**Parent topic:**[Bolt release notes](bolt_release_notes.md)


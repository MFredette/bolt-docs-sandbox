---
author: Kate Lopresti <kate.lopresti@puppet.com\>
---

# Deprecations and removals

A list of the features and functions deprecated or removed from Bolt 1.x.

## `lookups` removed from `target_lookups` \(0.25.0\)

We have deprecated the `target-lookups` key in the experimental inventory file v2. To address this change, migrate any `target-lookups` entries to `targets` and move the `plugin` key in each entry to `_plugin`.

## Configuration location ~/.puppetlab/bolt.yaml \(0.21.0\)

When the directory Boltdir was added as the local default configuration directory, the previous directory, `~/.puppetlab/bolt.yaml`, was deprecated in favor of `~/.puppetlabs/bolt/bolt.yaml`. For more information on the current default directory for configfile, inventoryfile and modules, see [Configuring Bolt](configuring_bolt.md). \([BOLT-503](https://tickets.puppetlabs.com/browse/BOLT-503)\)

**Parent topic:**[Bolt release notes](bolt_release_notes.md)


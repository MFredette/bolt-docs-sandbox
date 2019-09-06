<?xml version="1.0" encoding="UTF-8"?><?path2rootmap-uri ./?>
<!DOCTYPE topic
  PUBLIC "-//OASIS//DTD DITA Topic//EN" "topic.dtd">
<topic id="inventory-file-version-2"><title>Inventory file version 2</title><prolog><author>Melissa Amos &lt;melissa.amos@puppet.com\&gt;</author></prolog><body><p>Version 2 of the inventory file is experimental and might experience breaking changes in future releases.</p><p><b>Parent topic:</b><xref href="inventory_file.md" format="dita" type="topic">Inventory file</xref></p></body><topic id="migrating-to-version-2"><title>Migrating to version 2</title><body><p>Version 2 of the inventory file changes some terms and syntax. To convert to version 2, you must make these changes.</p><p><b><codeph>version: 2</codeph></b></p><p>The default version for inventory files is version 1. In order to have Bolt treat your inventory file as a version 2 inventory, specify <codeph>version: 2</codeph> at the top level.</p><p><b><codeph>nodes</codeph> =\&gt; <codeph>targets</codeph></b></p><p>In order to standardize terminology across Bolt and capture the breadth of possible targets, such as web services, version 2 of the inventory file uses the <codeph>targets</codeph> section of a group to specify its members instead of <codeph>nodes</codeph>.</p><p><b><codeph>name</codeph> =\&gt; <codeph>uri</codeph></b></p><p>Changing the <codeph>name</codeph> key to <codeph>uri</codeph> results in an inventory file that matches the behavior of version 1.</p><p>In version 1 of the inventory file, Bolt treated the <codeph>name</codeph> field of a node as its URI. This made it impossible to specify a <codeph>name</codeph> that did not include the hostname of a target, which proved limiting for remote targets. In version 2, the optional <codeph>uri</codeph> field sets the URI for a target. Any connection information from the URI, such as a user specified by user@uri can't be overridden with other configuration methods. If the <codeph>uri</codeph> is set, it's used as the default value for the <codeph>name</codeph> key. Every target requires a <codeph>name</codeph>, so either the <codeph>name</codeph> or <codeph>uri</codeph> field must be set.</p><p>If there is a bare string in the target's array, Bolt tries to resolve the string to a target defined elsewhere in the inventory. If no target has a name or alias matching the string, Bolt creates a new target with the string as its URI.</p></body></topic><topic id="version-2-features"><title>Version 2 features</title><body><p>These features are exclusive to version 2 of the inventory file.</p></body><topic id="creating-a-node-with-a-human-readable-name-and-ip-address"><title>Creating a node with a human readable name and IP address</title><body><p>With version 2 of the inventory file, you can create a node with a human readable name even when an IP address is used for connecting.</p><p>You do so by setting both a <codeph>uri</codeph> and <codeph>name</codeph>, or by setting <codeph>host</codeph> in the transport config in addition to the <codeph>name</codeph>.</p><codeblock xml:space="preserve">targets:
  - name: my_device
    config:
      transport: remote
      remote:
        host: 192.168.100.179
  - name: my_device2
    uri: 192.168.100.179</codeblock></body></topic><topic id="plugins-and-dynamic-inventory"><title>Plugins and dynamic inventory</title><body><p>Inventory plugins can be used to dynamically load information into the inventroy file.</p><p>To use a plugin, replace a static value in the inventory file with an Object containing a <codeph>_plugin</codeph> key and any required plugin- specific options.The location where you do this replacement determines how the plugin behaves. Currently, plugins are only supported for <codeph>inventory_targets</codeph>, in the <codeph>targets</codeph> section of inventory, and <codeph>inventory_config</codeph>, in the config section of inventory. Most plugins only work in one location or the other.</p></body><topic id="target-plugins"><title>Target plugins</title><body><p>To use an<codeph>inventory_target</codeph> plugin, replace an item in the <codeph>targets</codeph> array with a plugin object.</p><codeblock xml:space="preserve">targets:
 - _plugin: my_plugin
   plugin_specific_option: exampleoption
            </codeblock><p>Use the following optional plugins for targets:</p><ul><li><p><codeph>config</codeph> - use it to look up a value.</p></li><li><p><codeph>puppetdb</codeph> - query PuppetDB to populate the targets.</p></li><li><p><codeph>terraform</codeph> - load a Terraform state file to populate the targets.</p></li></ul><p><b>Config plugins</b></p><p>Config plugins can be used inside the <codeph>config</codeph> section of a target or group to look up a value. Config lookup plugins that return a value with a <codeph>_plugin</codeph> are not reevaluated.</p><codeblock xml:space="preserve">config: 
  transport:
   _plugin: my_plugin
   plugin_specific_option: exampleoption
            </codeblock><p>These plugins can be used for <codeph>config</codeph>:</p><ul><li><p><codeph>prompt</codeph> - prompts a user for a configuration value.</p></li><li><p><codeph>pkcs7</codeph> - decrypts a pkcs7 encrypted value from the inventory file.</p></li></ul><p><b>Task</b></p><p>The Task plugin runs a task on <codeph>localhost</codeph> to look up configuration information or target lists for the inventory.</p><p>For both use cases, the plugin accepts two keys:</p><ul><li><p><codeph>task</codeph> - the task to run.</p></li><li><p><codeph>parameters</codeph> - the parameters to pass to the task.</p></li></ul><p>For example the following runs the <codeph>my_json_file::targets</codeph> task to look up targets and the <codeph>my_db::secret_lookup</codeph> task to look up the SSH password.</p><codeblock xml:space="preserve">version: 2
targets:
  - _plugin: task
    task: my_json_file::targets
    parameters:
      # These parameters are specific to the task
      file: /etc/targets/data.json
      environment: production
      app: my_app
config:
  ssh:
    password:
      _plugin: task
      task: my_db::secret_lookup
      parameters:
        # These parameters are task specific
        key: ssh_password</codeblock><p><b>Inventory config tasks</b></p><p>To look up configuration information for the inventory return a <codeph>config</codeph> key in the task result containing the data that will be used in place where the plugin entry is. The value of config can be any type of data that is appropriate for the specific location in config. For example <codeph>host-key-check</codeph> for SSH must be a Boolean, <codeph>password</codeph> must be a string, and <codeph>run-as-command</codeph> must be an array of strings. This result is appropriate for an entire <codeph>ssh</codeph> section of config.</p><codeblock xml:space="preserve">{
  "config": {
    "host-key-check": true,
    "password": "hunter2",
    "run-as-command": [ "sudo", "-k", "-S", "-E", "-u", "user", "-p", "password"]
  }
}</codeblock><p>This task looks up a password value from a secret database and returns it.</p><codeblock xml:space="preserve">#!/usr/bin/env python
import json, sys
from my_secret import Client

params = json.load(sys.stdin)

client = Client
secret = client.get_secret(data['key'])
# secret can be any value that can be dumped to json.
json.dump({'config': secret}, sys.stdout)</codeblock><p><b>Inventory target tasks</b></p><p>To look up a list of targets for inventory, return a hash or JSON object that includes a <codeph>targets</codeph> key with an <codeph>array</codeph> value from task. Each item in this list should be a Target JSON object matching the <codeph>Object</codeph> format you use in a <codeph>targets</codeph> section of the inventory file. Bare string targets are not valid.</p><p>This task reads in a JSON file and looks up a value.</p><codeblock xml:space="preserve">#!/usr/bin/env python
import json, sys

params = json.load(sys.stdin)
with open(params['file']) as fh:
  data = json.load(fh)
targets = data[params['environment']][params['app']]
json.dump({'targets': targets}, sys.stdout)
</codeblock><p><b>PuppetDB</b></p><p>The PuppetDB plugin supports looking up target objects from PuppetDB. It takes a <codeph>query</codeph> field, which is either a string containing a <xref href="https://puppet.com/docs/puppetdb/latest/api/query/v4/ast.html" format="html" scope="external">PuppetDB AST</xref> query. The query is used to determine which targets should be included in the group. If <codeph>name</codeph> or <codeph>uri</codeph> is not specified with a fact lookup, then the <xref href="https://puppet.com/docs/puppet/latest/lang_facts_and_builtin_vars.html#trusted-facts" format="html" scope="external">certname</xref> for each target in the query result will be used as the <codeph>uri</codeph> for the new target.</p><codeblock xml:space="preserve">groups:
  - name: windows
    targets:
     - _plugin: puppetdb
        query: "inventory[certname] { facts.osfamily = 'windows' }"
    config:
      transport: winrm
  - name: redhat
    targets:
      - _plugin: puppetdb
        query: "inventory[certname] { facts.osfamily = 'RedHat' }"
    config:
      transport: ssh</codeblock><p>Make sure you've <xref href="bolt_connect_puppetdb.md" format="dita" type="topic">configured PuppetDB</xref>.</p><p>If target-specific configuration is required, the PuppetDB plugin can be used to look up configuration values for the <codeph>name</codeph>, <codeph>uri</codeph>, and <codeph>config</codeph> inventory options for each target. The fact lookup values can be either <codeph>certname</codeph>, to reference the <xref href="https://puppet.com/docs/puppet/latest/lang_facts_and_builtin_vars.html#trusted-facts" format="html" scope="external">certname</xref> of the target, or a <xref href="https://puppet.com/docs/puppetdb/latest/api/query/v4/ast.html#dot-notation" format="html" scope="external">PQL dot notation</xref> facts string, such as <codeph>facts.os.family</codeph>, to reference fact value. Dot notation is required for both structured and unstructured facts.</p><p><b>Note:</b> If the <codeph>name</codeph> or <codeph>uri</codeph> values are set to a lookup, the PuppetDB plugin will <b>not</b> set the <codeph>uri</codeph> to the certname of the target.</p><p>For example, to set the user to be the user from the <xref href="https://puppet.com/docs/facter/latest/core_facts.html#identity" format="html" scope="external">identity fact</xref>:</p><codeblock xml:space="preserve">version: 2
groups:
  - name: dynamic_config
    targets:
      - _plugin: puppetdb
        query: "inventory[certname] { facts.osfamily = 'RedHat' }"
        config:
          ssh:
            # Lookup config from PuppetDB facts
            user: facts.identity.user
    # And include static config
    config:
      ssh:
        tmpdir: /tmp/mytmp</codeblock><p>And to use the certname of a target as the <codeph>name</codeph>:</p><codeblock xml:space="preserve">version: 2
groups:
  - name: dynamic_config
    targets:
      - _plugin: puppetdb
        query: "inventory[certname] { facts.osfamily = 'RedHat' }"
        name: certname
        config:
          ssh:
            # Lookup config from PuppetDB facts
            hostname: facts.networking.interfaces.en0.ipaddress</codeblock><p><b>Terraform</b></p><p>The Terraform plugin supports looking up target objects from a Terraform state file. It accepts several fields:</p><ul><li><p><codeph>dir</codeph>: The directory from which to load Terraform state.</p></li><li><p><codeph>resource_type</codeph>: The Terraform resources to match, as a regular expression.</p></li><li><p><codeph>uri</codeph>: \(Optional\) The property of the Terraform resource to use as the target URI.</p></li><li><p><codeph>statefile</codeph>: \(Optional\) The name of the Terraform state file to load within <codeph>dir</codeph>. Defaults to <codeph>terraform.tfstate</codeph>.</p></li><li><p><codeph>name</codeph>: \(Optional\) The property of the Terraform resource to use as the target name.</p></li><li><p><codeph>config</codeph>: A Bolt config map where each value is the Terraform property to use for that config setting.</p></li></ul><p>Either<codeph>uri</codeph> or <codeph>name</codeph> is required. If only <codeph>uri</codeph> is set, then the value of <codeph>uri</codeph> is used as the <codeph>name</codeph>.</p><codeblock xml:space="preserve">groups:
  - name: cloud-webs
    targets:
      - _plugin: terraform
        dir: /path/to/terraform/project1
        resource_type: google_compute_instance.web
        uri: network_interface.0.access_config.0.nat_ip
      - _plugin: terraform
        dir: /path/to/terraform/project2
        resource_type: aws_instance.web
        uri: public_ip</codeblock><p>Multiple resources with the same name are identified as &lt;resource\&gt;.0, &lt;resource\&gt;.1, etc.</p><p>The path to nested properties must be separated with <codeph>.</codeph>: for example, <codeph>network_interface.0.access_config.0.nat_ip</codeph>.</p><p>For example, the following truncated output creates two targets, named <codeph>34.83.150.52</codeph> and <codeph>34.83.16.240</codeph>. These targets are created by matching the resources <codeph>google_compute_instance.web.0</codeph> and <codeph>google_compute_instance.web.1</codeph>. The <codeph>uri</codeph> for each target is the value of their <codeph>network_interface.0.access_config.0.nat_ip</codeph> property, which corresponds to the externally routable IP address in Google Cloud.</p><codeblock xml:space="preserve">google_compute_instance.web.0:
  id = web-0
  cpu_platform = Intel Broadwell
  machine_type = f1-micro
  name = web-0
  network_interface.# = 1
  network_interface.0.access_config.# = 1
  network_interface.0.access_config.0.assigned_nat_ip = 
  network_interface.0.access_config.0.nat_ip = 34.83.150.52
  network_interface.0.address = 
  network_interface.0.name = nic0
  network_interface.0.network = https://www.googleapis.com/compute/v1/projects/cloud-app1/global/networks/default
  network_interface.0.network_ip = 10.138.0.22
  project = cloud-app1
  self_link = https://www.googleapis.com/compute/v1/projects/cloud-app1/zones/us-west1-a/instances/web-0
  zone = us-west1-a
google_compute_instance.web.1:
  id = web-1
  cpu_platform = Intel Broadwell
  machine_type = f1-micro
  name = web-1
  network_interface.# = 1
  network_interface.0.access_config.# = 1
  network_interface.0.access_config.0.assigned_nat_ip = 
  network_interface.0.access_config.0.nat_ip = 34.83.16.240
  network_interface.0.address = 
  network_interface.0.name = nic0
  network_interface.0.network = https://www.googleapis.com/compute/v1/projects/cloud-app1/global/networks/default
  network_interface.0.network_ip = 10.138.0.21
  project = cloud-app1
  self_link = https://www.googleapis.com/compute/v1/projects/cloud-app1/zones/us-west1-a/instances/web-1
  zone = us-west1-a
google_compute_instance.app.1:
  id = app-1
  cpu_platform = Intel Broadwell
  machine_type = f1-micro
  name = app-1
  network_interface.# = 1
  network_interface.0.access_config.# = 1
  network_interface.0.access_config.0.assigned_nat_ip = 
  network_interface.0.access_config.0.nat_ip = 35.197.93.137
  network_interface.0.address = 
  network_interface.0.name = nic0
  network_interface.0.network = https://www.googleapis.com/compute/v1/projects/cloud-app1/global/networks/default
  network_interface.0.network_ip = 10.138.0.23
  project = cloud-app1
  self_link = https://www.googleapis.com/compute/v1/projects/cloud-app1/zones/us-west1-a/instances/app-1
  zone = us-west1-a</codeblock><p><b>Prompt plugin</b></p><p>The <codeph>prompt</codeph> plugin can be used to allow users to interactively enter sensitive configuration information on the CLI instead of storing that data in the inventory file. Data is looked up only when the value is needed for the target and once the value has been stored, it is re-used for the rest of the run. The <codeph>prompt</codeph> plugin can be used only when nested under <codeph>config</codeph>. The prompt plugin can be used by replacing the config value with a hash that has the following keys:</p><ul><li><p><codeph>_plugin</codeph>: The value of <codeph>_plugin</codeph> must be <codeph>prompt</codeph></p></li><li><p><codeph>message</codeph>: The value of <codeph>message</codeph> must be the text to show when prompting the user on the CLI</p></li></ul><codeblock xml:space="preserve">version: 2
targets:
  - uri: 192.168.100.179
    config:
      transport: ssh
      ssh:
        user: root
        password: 
          _plugin: prompt
          message: please enter your ssh password</codeblock><p><b>pkcs7 plugin</b></p><p>This plugin allows config values to be stored in encrypted in the inventory file and decrypted only as needed.</p><p><codeph>_plugin</codeph>: The value of <codeph>_plugin</codeph> must be <codeph>pkcs7</codeph></p><p><codeph>encrypted-value</codeph>: The encrypted value. Generate encrypted values with <codeph>bolt secret encrypt &lt;plaintext&gt;</codeph></p><codeblock xml:space="preserve">version: 2
targets:
  - uri: 192.168.100.179
    config:
      transport: ssh
      ssh:
        user: root
        password:
          _plugin: pkcs7
          encrypted-value: |
                ENC[PKCS7,MIIBeQYJKoZIhvcNAQcDoIIBajCCAWYCAQAxggEhMIIBHQIBADAFMAACAQEw
                DQYJKoZIhvcNAQEBBQAEggEAdCVkiddtK8jHz4g1y1pkB27VHCZx7dVzEiyT
                33BgFv9atk8Ns/WE1tveFvyuEaDpk9y/FKisuh8DsTnR2mfGvHtX+BQdNqV6
                L8/nIdwoEqYFd5sKFJnOlpdm7BMX4QDoCfGb+b2UB8A/7eJJ5AcgBVtrJLLE
                VvqSCtqME12ltifdMivMP1hnVJOAhIpib8CwOIIP+Dtv7P7cPaHGTdQpR6Dp
                jbe+AUDM6kcKGADLOYriPQ1UV6zDz5aeUbrwbr4FicHL/sQBPDcWIJR2elwY
                bh8hCDe/IIWE7TOiauXOPyMPKohz622KNoJDJbmv5MhBwNFHSjgKAlOAxL3i
                DK7XXzA8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBCvjDMKTjsHloKP04WO
                Dq0ogBAUjTZMjbKjkndMSqPC5mGC]</codeblock><p>Before using the pkcs7 plugin you need to create encryption keys. You can create these keys automatically with <codeph>bolt secret createkeys</codeph> or reuse existing hiera-eyaml pkcs7 keys with <codeph>bolt secret</codeph>. You can then encrypt values with the <codeph>bolt secret encrypt &lt;plaintext&gt;</codeph> command and copy the result into your inventory file. If you need to inspect an encrypted value from the inventory, you can decrypt it with <codeph>bolt secret decrypt &lt;encrypted-value&gt;</codeph>.</p><p><b>Configuration</b></p><p>By default, keys are stores in the <codeph>keys</codeph> directory of the Bolt project repo. If you're sharing your project directory, you can move the private key outside the project directory by configuring the key location in bolt.yaml</p><codeblock xml:space="preserve">---
plugins:
  pkcs7:
    private-key: ~/bolt_private_key.pem</codeblock><ul><li><p><codeph>keysize</codeph>: They size of the key to generate with <codeph>bolt secret createkeys</codeph>. The default is <codeph>2048</codeph></p></li><li><p><codeph>private-key</codeph>: The path to the private key file. The default is <codeph>keys/private_key.pkcs7.pem</codeph></p></li><li><p><codeph>public-key</codeph>: The path to the public key file. The default is <codeph>keys/public_key.pkcs7.pem</codeph></p></li></ul></body></topic></topic></topic><topic id="inventory-config"><title>Inventory config</title><body><p>You can set transport configuration only in the inventory file. This means using a top level <codeph>transport</codeph> value to assign a transport to the target and all values in the section named for the transport. You can set config on targets or groups in the inventory file. Bolt performs a depth first search of targets, followed by a search of groups, and uses the first value it finds. Nested hashes are merged.</p><p>This inventory file example defines two top-level groups: <codeph>ssh_targets</codeph> and <codeph>win_targets</codeph>. The <codeph>ssh_targets</codeph> group contains two other groups: <codeph>webservers</codeph> and <codeph>memcached</codeph>. Five targets are configured to use ssh transport and four other nodes to use WinRM transport.</p><codeblock xml:space="preserve">groups:
  - name: ssh_targets
    groups:
      - name: webservers
        targets:
          - name: my_node1
            uri: 192.168.100.179
          - 192.168.100.180
          - 192.168.100.181
      - name: memcached
        targets:
          - 192.168.101.50
          - 192.168.101.60
        config:
          ssh:
            user: root
    config:
      transport: ssh
      ssh:
        user: centos
        private-key: ~/.ssh/id_rsa
        host-key-check: false
  - name: win_targets
    groups:
      - name: domaincontrollers
        targets:
          - 192.168.110.10
          - 192.168.110.20
      - name: testservers
        targets:
          - 172.16.219.20
          - 172.16.219.30
        config:
          winrm:
            user: vagrant
            password: vagrant
            ssl: false
    config:
      transport: winrm
      winrm:
        user: DOMAIN\opsaccount
        password: S3cretP@ssword
        ssl: true
</codeblock></body><topic id="override-a-user-for-a-specific-target"><title>Override a user for a specific target</title><body><codeblock xml:space="preserve">targets:
  - uri: linux1.example.com
    config:
      ssh:
        user: me</codeblock></body></topic><topic id="provide-an-alias-to-a-target"><title>Provide an alias to a target</title><body><p>The inventory can be used to create aliases to refer to a target. Aliases can be useful to refer to nodes with long or complicated names, like db.uswest.acme.example.com, or for targets that include protocol or port for uniqueness, such as 127.0.0.1:2222 and 127.0.0.1:2223. Aliases can also be useful when generating targets in a dynamic environment to give generated targets stable names to refer to.</p><p>An alias can be a single name or list of names. Each alias must match the regex <codeph>/[a-zA-Z]\w+/</codeph>. When using Bolt, you may refer to a target by its alias anywhere the target name would be applicable, such as the <codeph>--targets</codeph> command line argument or a <codeph>TargetSpec</codeph>.</p><codeblock xml:space="preserve">targets:
  - uri: linux1.example.com
    alias: linux1
    config:
      ssh:
        port: 2222</codeblock><p>Aliases must be unique across the entire inventory. You can use the same alias multiple places, but they must all refer to the same target. Alias names must not match any group or target names used in the inventory.</p><p>A list of targets may refer to a target by its alias, for example:</p><codeblock xml:space="preserve">targets:
  - uri: 192.168.110.10
    alias: linux1
groups:
  - name: group1
    targets:
      - linux1</codeblock></body></topic></topic><topic id="inventory-facts-vars-and-features"><title>Inventory facts, vars, and features</title><body><p>In addition to config values you can store information relating to <codeph>facts</codeph>, <codeph>vars</codeph> and <codeph>features</codeph> for targets in the inventory. <codeph>facts</codeph> represent observed information about the target including what can be collected by Facter. <codeph>vars</codeph> contain arbitrary data that may be passed to run\\\_\\\* functions or used for logic in plans. <codeph>features</codeph> represent capabilities of the target that can be used to select a specific task implementation.</p><codeblock xml:space="preserve">groups:
  - uri: centos_targets
    targets:
      - foo.example.com
      - bar.example.com
      - baz.example.com
    facts:
      operatingsystem: CentOS
  - name: production_targets
    vars:
      environment: production
    features: ['puppet-agent']
</codeblock></body></topic><topic id="objects"><title>Objects</title><body><p>The inventory file uses the following objects.</p><ul><li><p><b>Config</b></p><p>A config is a map that contains transport specific configuration options.</p></li><li><p><b>Group</b></p><p>A group is a map that requires a <codeph>name</codeph> and can contain any of the following:</p><ul><li><p><codeph>targets</codeph> : <codeph>Array[Node]</codeph></p></li><li><p><codeph>groups</codeph> : Groups object</p></li><li><p><codeph>config</codeph> : Config object</p></li><li><p><codeph>facts</codeph> : Facts object</p></li><li><p><codeph>vars</codeph> : Vars object</p></li><li><p><codeph>features</codeph> : <codeph>Array[Feature]</codeph></p></li></ul><p>A group name must match the regular expression values <codeph>/[a-zA-Z]\w+/</codeph>. These are the same values used for environments.</p><p>A group may contain other groups. Any nodes in the nested groups will also be in the parent group. The configuration of nested groups will override the parent group.</p></li><li><p><b>Groups</b></p><p>An array of group objects.</p></li><li><p><b>Facts</b></p><p>A map of fact names and values. Values may include arrays or nested maps.</p></li><li><p><b>Feature</b></p><p>A string describing a feature of the target.</p></li><li><p><b>Target</b></p><p>A target can be just the string of its target URI or a map that requires a name key and can contain a config. For example, a target block can contain any of the following:</p><codeblock xml:space="preserve">"host1.example.com"</codeblock><codeblock xml:space="preserve">uri: "host1.example.com"</codeblock><codeblock xml:space="preserve">uri: "host1.example.com"
config:
					   transport: "ssh"</codeblock><p>If the target entry is a map, it may contain any of the following:</p><ul><li><p><codeph>alias</codeph> : <codeph>String</codeph> or <codeph>Array[String]</codeph></p></li><li><p><codeph>config</codeph> : Config object</p></li><li><p><codeph>facts</codeph> : Facts object</p></li><li><p><codeph>vars</codeph> : Vars object</p></li><li><p><codeph>features</codeph> : <codeph>Array[Feature]</codeph></p></li></ul></li><li><p><b>Target name</b></p><p>The name used to refer to a target.</p></li><li><p><b>Targets</b></p><p>An array of target objects.</p></li><li><p><b>Vars</b></p><p>A map of value names and values. Values may include arrays or nested maps.</p></li></ul></body></topic><topic id="file-format"><title>File format</title><body><p>The inventory file is a yaml file that contains a single group. This group can be referred to as "all". In addition to the normal group fields, the top level has an inventory file version key that defaults to 1.</p></body></topic><topic id="precedence"><title>Precedence</title><body><p>If a target specifies a <codeph>uri</codeph> or is created from a URI string any URI-based configuration information like host, transport or port will override config values even those defined in the same block. For config values, the first value found for a target is used. Node values take precedence over group values and are searched first. Values are searched for in a depth first order. The first item in an array is searched first.</p><p>Configure transport for targets.</p><codeblock xml:space="preserve">groups:
  - name: linux
    targets:
      - linux1.example.com
      - linux2.example.com
      - linux3.example.com
  - name: win
    targets:
      - win1.example.com
      - win2.example.com
      - win3.example.com
    config:
      transport: winrm</codeblock><p>Configure login and escalation for a specific target.</p><codeblock xml:space="preserve">targets:
  - uri: host1.example.com
    config:
      ssh:
          user: me
          run-as: root</codeblock></body></topic><topic id="remote-targets"><title>Remote targets</title><body><p>Configure a remote target. When using the remote transport, the protocol of the target name does not have to map to the transport if you set the transport config option. This is useful if the target is an http API, for example:</p><codeblock xml:space="preserve">targets:
  - host1.example.com
  - uri: https://user1:secret@remote.example.com
    config:
      transport: remote
      remote:
        # The remote transport will use the host1.example.com target from
        # inventory to proxy tasks execution on.
        run-on: host1.example.com
  # This will execute on localhost.
  - remote://my_aws_account</codeblock><p><b>Related information</b></p><p><xref href="writing_tasks.md#" format="dita" type="topic">Naming tasks</xref></p></body></topic></topic>
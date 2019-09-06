<?xml version="1.0" encoding="UTF-8"?><?path2rootmap-uri ./?>
<!DOCTYPE topic
  PUBLIC "-//OASIS//DTD DITA Topic//EN" "topic.dtd">
<topic id="inventory-file"><title>Inventory file</title><prolog><author>Kate Lopresti &lt;kate.lopresti@puppet.com\&gt;</author></prolog><body><p>In Bolt, you can use an inventory file to store information about your nodes. For example, you can organize your nodes into groups or set up connection information for nodes or node groups.</p><p>The inventory file is a yaml file stored by default at<codeph>inventory.yaml</codeph> inside the <xref href="bolt_project_directories.md#" format="dita" type="topic">Bolt project directory</xref>. At the top level it contains an array of nodes and groups. Each node can have a config, facts, vars, and features specific to that node. Each group can have an array of nodes, an array of child groups, and can set default config, vars, and features for the entire group.</p><p><b>Note:</b> Configuration values set at the top level of inventory will only apply to targets included in that inventory file. Set values for unknown targets in the Bolt configuration file.</p></body><topic id="inventory-config"><title>Inventory config</title><body><p>You can set transport configuration only in the inventory file. This means using a top level <codeph>transport</codeph> value to assign a transport to the target and all values in the section named for the transport.</p><p>You can set config on nodes or groups in the inventory file. Bolt performs a depth first search of nodes, followed by a search of groups, and uses the first value it finds. Nested hashes are merged.</p><p>This inventory file example defines two top-level groups: <codeph>ssh_nodes</codeph> and <codeph>win_nodes</codeph>. The <codeph>ssh_nodes</codeph> group contains two other groups: <codeph>webservers</codeph> and <codeph>memcached</codeph>. Five nodes are configured to use ssh transport and four other nodes to use WinRM transport.</p><codeblock xml:space="preserve">groups:
  - name: ssh_nodes
    groups:
      - name: webservers
        nodes:
          - 192.168.100.179
          - 192.168.100.180
          - 192.168.100.181
      - name: memcached
        nodes:
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
  - name: win_nodes
    groups:
      - name: domaincontrollers
        nodes:
          - 192.168.110.10
          - 192.168.110.20
      - name: testservers
        nodes:
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
</codeblock></body></topic><topic id="override-a-user-for-a-specific-node"><title>Override a user for a specific node</title><body><codeblock xml:space="preserve">nodes: 
  - name: linux1.example.com
    config: 
      ssh:
        user: me</codeblock></body></topic><topic id="provide-an-alias-to-a-node"><title>Provide an alias to a node</title><body><p>The inventory can be used to create aliases to refer to a target. Aliases can be useful to refer to nodes with long or complicated names, like db.uswest.acme.example.com, or for targets that include protocol or port for uniqueness, such as 127.0.0.1:2222 and 127.0.0.1:2223. Aliases can also be useful when generating nodes in a dynamic environment to give generated targets stable names to refer to.</p><p>An alias can be a single name or list of names. Each alias must match the regex<codeph>/[a-zA-Z]\w+/</codeph>. When usingBolt, you may refer to a node by its alias anywhere the node name would be applicable, such as the<codeph>--nodes</codeph> command line argument or a<codeph>TargetSpec</codeph>.</p><codeblock xml:space="preserve">nodes:
  - name: linux1.example.com
    alias: linux1
    config:
      ssh:
        port: 2222</codeblock><p>Aliases must be unique across the entire inventory. You can use the same alias multiple places, but they must all refer to the same target. Alias names must not match any group or target names used in the inventory.</p><p>A list of nodes may refer to a node by its alias, for example:</p><codeblock xml:space="preserve">nodes:
  - linux1</codeblock></body></topic><topic id="inventory-facts-vars-and-features"><title>Inventory facts, vars, and features</title><body><p>In addition to config values you can store information relating to <codeph>facts</codeph>, <codeph>vars</codeph> and <codeph>features</codeph> for nodes in the inventory. <codeph>facts</codeph> represent observed information about the node including what can be collected by Facter. <codeph>vars</codeph> contain arbitrary data that may be passed to run\_\* functions or used for logic in plans. <codeph>features</codeph> represent capabilities of the target that can be used to select a specific task implementation.</p><codeblock xml:space="preserve">groups:
  - name: centos_nodes
    nodes:
      - foo.example.com
      - bar.example.com
      - baz.example.com
    facts:
      operatingsystem: CentOS
  - name: production_nodes
    vars:
      environment: production
    features: ['puppet-agent']
</codeblock></body></topic><topic id="objects"><title>Objects</title><body><p>The inventory file uses the following objects.</p><ul><li><p><b>Config</b></p><p>A config is a map that contains transport specific configuration options.</p></li><li><p><b>Group</b></p><p>A group is a map that requires a <codeph>name</codeph> and can contain any of the following:</p><ul><li><p><codeph>nodes</codeph> : <codeph>Array[Node]</codeph></p></li><li><p><codeph>groups</codeph> : Groups object</p></li><li><p><codeph>config</codeph> : Config object</p></li><li><p><codeph>facts</codeph> : Facts object</p></li><li><p><codeph>vars</codeph> : Vars object</p></li><li><p><codeph>features</codeph> : <codeph>Array[Feature]</codeph></p></li></ul><p>A group name must match the regular expression values <codeph>/[a-zA-Z]\w+/</codeph>. These are the same values used for environments.</p><p>A group may contain other groups. Any nodes in the nested groups will also be in the parent group. The configuration of nested groups will override the parent group.</p></li><li><p><b>Groups</b></p><p>An array of group objects.</p></li><li><p><b>Facts</b></p><p>A map of fact names and values. Values may include arrays or nested maps.</p></li><li><p><b>Feature</b></p><p>A string describing a feature of the target.</p></li><li><p><b>Node</b></p><p>A node can be just the string of its node name or a map that requires a name key and can contain a config. For example, a node block can contain any of the following:</p><codeblock xml:space="preserve">"host1.example.com"</codeblock><codeblock xml:space="preserve">name: "host1.example.com"</codeblock><codeblock xml:space="preserve">name: "host1.example.com"
config:
					   transport: "ssh"</codeblock><p>If the node entry is a map, it may contain any of the following:</p><ul><li><p><codeph>alias</codeph> : <codeph>String</codeph> or <codeph>Array[String]</codeph></p></li><li><p><codeph>config</codeph> : Config object</p></li><li><p><codeph>facts</codeph> : Facts object</p></li><li><p><codeph>vars</codeph> : Vars object</p></li><li><p><codeph>features</codeph> : <codeph>Array[Feature]</codeph></p></li></ul></li><li><p><b>Node name</b></p><p>The URI used to create the node.</p></li><li><p><b>Nodes</b></p><p>An array of node objects.</p></li><li><p><b>Vars</b></p><p>A map of value names and values. Values may include arrays or nested maps.</p></li></ul></body></topic><topic id="file-format"><title>File format</title><body><p>The inventory file is a yaml file that contains a single group. This group can be referred to as "all". In addition to the normal group fields, the top level has an inventory file version key that defaults to 1.</p></body></topic><topic id="precedence"><title>Precedence</title><body><p>When searching for node config, the URI used to create the target is matched to the node-name. Any URI-based configuration information like host, transport or port will override config values even those defined in the same block. For config values, the first value found for a node is used. Node values take precedence over group values and are searched first. Values are searched for in a depth first order. The first item in an array is searched first.</p><p>Configure transport for nodes.</p><codeblock xml:space="preserve">groups:
  - name: linux
    nodes:
      - linux1.example.com
      - linux2.example.com
      - linux3.example.com
  - name: win
    nodes:
      - win1.example.com
      - win2.example.com
      - win3.example.com
    config:
      transport: winrm</codeblock><p>Configure login and escalation for a specific node.</p><codeblock xml:space="preserve">nodes:
  - name: host1.example.com
    config:
      ssh:
          user: me
          run-as: root</codeblock></body></topic><topic id="remote-targets"><title>Remote targets</title><body><p>Configure a remote target. When using the remote transport, the protocol of the node name does not have to map to the transport if you set the transport config option. This is useful if the target is an http API, for example:</p><codeblock xml:space="preserve">nodes:
  - host1.example.com
  - name: https://user1:secret@remote.example.com
    config:
      transport: remote
      remote:
        # The remote transport will use the host1.example.com target from
        # inventory to proxy tasks execution on.
        run-on: host1.example.com  
  # This will execute on localhost.
  - remote://my_aws_account</codeblock><ul><li><p><b><xref href="inventory_file_v2.md#" format="dita" type="topic">Inventory file version 2</xref></b><?linebreak?>Version 2 of the inventory file is experimental and might experience breaking changes in future releases.</p></li><li><p><b><xref href="inventory_file_generating.md" format="dita" type="topic">Generating inventory files</xref></b><?linebreak?>Use the <codeph>bolt-inventory-pdb</codeph> script to generate inventory files based on PuppetDB queries.</p></li></ul><p><b>Related information</b></p><p><xref href="writing_tasks.md#" format="dita" type="topic">Naming tasks</xref></p></body></topic></topic>
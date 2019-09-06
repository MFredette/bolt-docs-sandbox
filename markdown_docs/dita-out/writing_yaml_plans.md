<?xml version="1.0" encoding="UTF-8"?><?path2rootmap-uri ./?>
<!DOCTYPE topic
  PUBLIC "-//OASIS//DTD DITA Topic//EN" "topic.dtd">
<topic id="writing-plans-in-yaml"><title>Writing plans in YAML</title><prolog><author>Melissa Amos &lt;melissa.amos@puppet.com\&gt;</author></prolog><body><p>YAML plans run a list of steps in order, which allows you to define simple workflows. Steps can contain embedded Puppet code expressions to add logic where necessary.</p><p><b>Note:</b> YAML plans are an experimental feature and might experience breaking changes in future minor \(y\) releases.</p><p><b>Parent topic:</b><xref href="writing_tasks_and_plans.md" format="dita" type="topic">Tasks and plans</xref></p></body><topic id="naming-plans"><title>Naming plans</title><body><p>Plan names are named based on the filename of the plan, the name of the module containing the plan, and the path to the plan within the module.</p><p>Place plan files in your module's <codeph>./plans</codeph> directory, using these file extensions:</p><ul><li><p>Puppet plans — <codeph>.pp</codeph></p></li><li><p>YAML plans — <codeph>.yaml</codeph>, not <codeph>.yml</codeph></p></li></ul><p>Plan names are composed of two or more name segments, indicating:</p><ul><li><p>The name of the module the plan is located in.</p></li><li><p>The name of the plan file, without the extension.</p></li><li><p>The path within the module, if the plan is in a subdirectory of <codeph>./plans</codeph>.</p></li></ul><p>For example, given a module called <codeph>mymodule</codeph> with a plan defined in <codeph>./mymodule/plans/myplan.pp</codeph>, the plan name is <codeph>mymodule::myplan</codeph>. A plan defined in <codeph>./mymodule/plans/service/myplan.pp</codeph>would be <codeph>mymodule::service::myplan</codeph>. This name is how you refer to the plan when you run commands.</p><p>The plan filename <codeph>init</codeph> is special: the plan it defines is referenced using the module name only. For example, in a module called <codeph>mymodule</codeph>, the plan defined in <codeph>init.pp</codeph> is the <codeph>mymodule</codeph> plan.</p><p>Avoid giving plans the same names as constructs in the Puppet language. Although plans do not share their namespace with other language constructs, giving plans these names makes your code difficult to read.</p><p>Each plan name segment must begin with a lowercase letter and:</p><ul><li><p>May include lowercase letters.</p></li><li><p>May include digits.</p></li><li><p>May include underscores.</p></li><li><p>Must not be a <xref href="https://docs.puppet.com/puppet/5.3/lang_reserved.html" format="html" scope="external">reserved word</xref>.</p></li><li><p>Must not have the same name as any Puppet data types.</p></li><li><p>Namespace segments must match the following regular expression <codeph>\A[a-z][a-z0-9_]*\Z</codeph></p></li></ul></body></topic><topic id="plan-structure"><title>Plan structure</title><body><p>YAML plans contain a list of steps with optional parameters and results.</p><p>YAML maps accept these keys:</p><ul><li><p><codeph>steps</codeph>: The list of steps to perform</p></li><li><p><codeph>parameters</codeph>: \(Optional\) The parameters accepted by the plan</p></li><li><p><codeph>return</codeph>: \(Optional\) The value to return from the plan</p></li></ul></body><topic id="steps-key"><title>Steps key</title><body><p>The <codeph>steps</codeph> key is an array of step objects, each of which corresponds to a specific action to take.</p><p>When the plan runs, each step is executed in order. If a step fails, the plan halts execution and raises an error containing the result of the step that failed.</p><p>Steps use these fields:</p><ul><li><p><codeph>name</codeph>: A unique name that can be used to refer to the result of the step later</p></li><li><p><codeph>description</codeph>: \(Optional\) An explanation of what the step is doing.</p></li></ul><p>Other available keys depend on the type of step.</p></body><topic id="command-step"><title>Command step</title><body><p>Use a <codeph>command</codeph> step to run a single command on a list of targets and save the results, containing stdout, stderr, and exit code.</p><p>The step fails if the exit code of any command is non-zero.</p><p>Command steps use these fields:</p><ul><li><p><codeph>command</codeph>: The command to run</p></li><li><p><codeph>target</codeph>: A target or list of targets to run the command on</p></li></ul><p>For example:</p><codeblock xml:space="preserve">steps:
  - command: hostname -f
    target:
      - web1.example.com
      - web2.example.com
      - web3.example.com
    description: "Get the webserver hostnames"</codeblock></body></topic><topic id="task-step"><title>Task step</title><body><p>Use a <codeph>task</codeph> step to run a Bolt task on a list of targets and save the results.</p><p>Task steps use these fields:</p><ul><li><p><codeph>task</codeph>: The task to run</p></li><li><p><codeph>target</codeph>: A target or list of targets to run the task on</p></li><li><p><codeph>parameters</codeph>: \(Optional\) A map of parameter values to pass to the task</p></li></ul><p>For example:</p><codeblock xml:space="preserve">steps:
  - task: package
    target:
      - web1.example.com
      - web2.example.com
      - web3.example.com
    description: "Check the version of the openssl package on the webservers"
    parameters:
      action: status
      name: openssl</codeblock></body></topic><topic id="script-step"><title>Script step</title><body><p>Use a <codeph>script</codeph> step to run a script on a list of targets and save the results.</p><p>The script must be in the <codeph>files/</codeph> directory of a module. The name of the script must be specified as <codeph>&lt;modulename&gt;/path/to/script</codeph>, omitting the <codeph>files</codeph> directory from the path.</p><p>Script steps use these fields:</p><ul><li><p><codeph>script</codeph>: The script to run</p></li><li><p><codeph>target</codeph>: A target or list of targets to run the script on</p></li><li><p><codeph>arguments</codeph>: \(Optional\) An array of command-line arguments to pass to the script</p></li></ul><p>For example:</p><codeblock xml:space="preserve">steps:
  - script: mymodule/check_server.sh
    target:
      - web1.example.com
      - web2.example.com
      - web3.example.com
    description: "Run mymodule/files/check_server.sh on the webservers"
    arguments:
      - "/index.html"
      - 60</codeblock></body></topic><topic id="file-upload-step"><title>File upload step</title><body><p>Use a file upload step to upload a file to a specific location on a list of targets.</p><p>The file to upload must be in the <codeph>files/</codeph> directory of a Puppet module. The source for the file must be specified as <codeph>&lt;modulename&gt;/path/to/file</codeph>, omitting the <codeph>files</codeph> directory from the path.</p><p>File upload steps use these fields:</p><ul><li><p><codeph>source</codeph>: The location of the file to be uploaded</p></li><li><p><codeph>destination</codeph>: The location where the file should be uploaded to</p></li></ul><p>For example:</p><codeblock xml:space="preserve">steps:
  - source: mymodule/motd.txt
    destination: /etc/motd
    target:
      - web1.example.com
      - web2.example.com
      - web3.example.com
    description: "Upload motd to the webservers"</codeblock></body></topic><topic id="plan-step"><title>Plan step</title><body><p>Use a <codeph>plan</codeph> step to run another plan and save its result.</p><p>Plan steps use these fields:</p><ul><li><p><codeph>plan</codeph>: The name of the plan to run</p></li><li><p><codeph>parameters</codeph>: \(Optional\) A map of parameter values to pass to the plan</p></li></ul><p>For example:</p><codeblock xml:space="preserve">steps:
  - plan: facts
    description: "Gather facts for the webservers using the built-in facts plan"
    parameters:
      nodes:
        - web1.example.com
        - web2.example.com
        - web3.example.com</codeblock></body></topic><topic id="resources-step"><title>Resources step</title><body><p>Use a <codeph>resources</codeph> step to apply a list of Puppet resources. A resource defines the desired state for part of a target. Bolt ensures each resource is in its desired state. Like the steps in a <codeph>plan</codeph>, if any resource in the list fails, the rest are skipped.</p><p>Resources steps use these fields:</p><ul><li><p><codeph>resouces</codeph>: An array of resources to apply</p></li><li><p><codeph>target</codeph>: A target or list of targets to apply the resources on</p></li></ul><p>Each resource is a YAML map with a type and title, and optionally a <codeph>parameters</codeph> key. The resource type and title can either be specified separately with the <codeph>type</codeph> and <codeph>title</codeph> keys, or can be specified in a single line by using the type name as a key with the title as its value.</p><p>For example:</p><codeblock xml:space="preserve">steps:
  - resources:
    # This resource is type 'package' and title 'nginx'
    - package: nginx
      parameters:
        ensure: latest
    # This resource is type 'service' and title 'nginx'
    - type: service
      title: nginx
      parameters:
        ensure: running
    target:
      - web1.example.com
      - web2.example.com
      - web3.example.com
    description: "Set up nginx on the webservers"</codeblock></body></topic></topic><topic id="parameters-key"><title>Parameters key</title><body><p>Plans accept parameters with the <codeph>parameters</codeph> key. The value of <codeph>parameters</codeph> is a map, where each key is the name of a parameter and the value is a map describing the parameter.</p><p>Parameter values can be referenced from steps as variables.</p><p>Parameters use these fields:</p><ul><li><p><codeph>type</codeph>: \(Optional\) A valid <xref href="https://puppet.com/docs/puppet/latest/lang_data.html#puppet-data-types" format="html" scope="external">Puppet data type</xref>. The value supplied must match the type or the plan fails.</p></li><li><p><codeph>default</codeph>: \(Optional\) Used if no value is given for the parameter</p></li><li><p><codeph>description</codeph>: \(Optional\)</p></li></ul><p>For example, this plan accepts a <codeph>load_balancer</codeph> name as a string, two sets of nodes called <codeph>frontends</codeph> and <codeph>backends</codeph>, and a <codeph>version</codeph> string:</p><codeblock xml:space="preserve">parameters:
  # A simple parameter definition doesn't need a type or description
  load_balancer:
  frontends:
    type: TargetSpec
    description: "The frontend web servers"
backends:
    type: TargetSpec
    description: "The backend application servers"
  version:
    type: String
              description: "The new application version to deploy"</codeblock></body></topic><topic id="how-strings-are-evaluated"><title>How strings are evaluated</title><body><p>The behavior of strings is defined by how theyre written in the plan.</p><p><codeph>'single-quoted strings'</codeph> are treated as string literals without any interpolation.</p><p><codeph>"double-quoted strings"</codeph> are treated as Puppet language double-quoted strings with variable interpolation.</p><p><codeph>| block-style strings</codeph> are treated as expressions of arbitrary Puppet code. Note the string itself must be on a new line after the <codeph>|</codeph> character.</p><p><codeph>bare strings</codeph> are treated dynamically based on their content. If they begin with a <codeph>$</codeph>, they're treated as Puppet code expressions. Otherwise, they're treated as YAML literals.</p><p>Here's an example of different kinds of strings in use:</p><codeblock xml:space="preserve">parameters:
  message:
    type: String
    default: "hello"

steps:
  - eval: hello
    description: 'This will evaluate to: hello'
  - eval: $message
    description: 'This will evaluate to: hello'
  - eval: '$message'
    description: 'This will evaluate to: $message'
  - eval: "${message} world"
    description: 'This will evaluate to: hello world'
  - eval: |
      [$message, $message, $message].join(" ")
    description: 'This will evaluate to: hello hello hello'</codeblock></body></topic></topic><topic id="using-variables-and-simple-expressions"><title>Using variables and simple expressions</title><body><p>Parameters and step results are available as variables during plan execution, and they can be used to compute the value for each field of a step.</p><p>The simplest way to use a variable is to reference it directly by name. For example, this plan takes a parameter called <codeph>nodes</codeph> and passes it as the target list to a step:</p><codeblock xml:space="preserve">parameters:
  nodes:
    type: TargetSpec

steps:
  - command: hostname -f
    target: $nodes</codeblock><p>Variables can also be interpolated into string values. The string must be double-quoted to allow interpolation. For example:</p><codeblock xml:space="preserve">parameters:
  username:
    type: String

steps:
  - task: echo
    message: "hello ${username}"
           target: $nodes</codeblock><p>Many operations can be performed on variables to compute new values for step parameters or other fields.</p></body><topic id="indexing-arrays-or-hashes"><title>Indexing arrays or hashes</title><body><p>You can retrieve a value from an Array or a Hash using the <codeph>[]</codeph> operator. This operator can also be used when interpolating a value inside a string.</p><codeblock xml:space="preserve">parameters:
  users:
    # Array[String] is a Puppet data type representing an array of strings
    type: Array[String]

steps:
  - task: user::add
    target: 'host.example.com'
    parameters:
      name: $users[0]
  - task: echo
    target: 'host.example.com'
    parameters:
      message: "hello ${users[0]}"</codeblock></body></topic><topic id="calling-functions"><title>Calling functions</title><body><p>You can call a built-in <xref href="plan_functions.md#" format="dita" type="topic">Bolt function</xref> or <xref href="https://puppet.com/docs/puppet/latest/function.html" format="html" scope="external">Puppet function</xref> to compute a value.</p><codeblock xml:space="preserve">parameters:
  users:
    type: Array[String]

steps:
  - task: user::add
    parameters:
      name: $users.first
  - task: echo
    message: "hello ${users.join(',')}"</codeblock></body></topic><topic id="using-code-blocks"><title>Using code blocks</title><body><p>Some Puppet functions take a block of code as an argument. For instance, you can filter an array of items based on the result of a block of code.</p><p>The result of the <codeph>filter</codeph> function is an array here, not a string, because the expression isn't inside quotes</p><codeblock xml:space="preserve">parameters:
  numbers:
    type: Array[Integer]

steps:
  - task: sum
    description: "add up the numbers &gt; 5"
    parameters:
      indexes: $numbers.filter |$num| { $num &gt; 5 }</codeblock></body></topic></topic><topic id="connecting-steps"><title>Connecting steps</title><body><p>You can connect multiple steps by using the result of one step to compute the parameters for another step.</p></body><topic id="name-key"><title><codeph>name</codeph> key</title><body><p>The <codeph>name</codeph> key makes its results available to later steps in a variable with that name.</p><p>This example uses the <codeph>map</codeph> function to get the value of <codeph>stdout</codeph> from each command result and then joins them into a single string separated by commas.</p><codeblock xml:space="preserve">parameters:
  nodes:
    type: TargetSpec

steps:
  - name: hostnames
    command: hostname -f
    target: $nodes
  - task: echo
    parameters:
      message: $hostnames.map |$hostname_result| { $hostname_result['stdout'] }.join(',')</codeblock></body></topic><topic id="eval-step"><title><codeph>eval</codeph> step</title><body><p>The <codeph>eval</codeph> step evaluates an expression and saves the result in a variable. This is useful to compute a variable to use multiple times later.</p><codeblock xml:space="preserve">parameters:
  count:
    type: Integer

steps:
  - name: double_count
    eval: $count * 2
  - task: echo
    target: web1.example.com
    parameters:
      message: "The count is ${count}, and twice the count is ${double_count}"</codeblock></body></topic></topic><topic id="returning-results"><title>Returning results</title><body><p>You can return a result from a plan by setting the <codeph>return</codeph> key at the top level of the plan. When the plan finishes, the <codeph>return</codeph> key is evaluated and returned as the result of the plan. If no <codeph>return</codeph> key is set, the plan returns <codeph>undef</codeph>.</p><codeblock xml:space="preserve">steps:
  - name: hostnames
    command: hostname -f
    target: $nodes

return: $hostnames.map |$hostname_result| { $hostname_result['stdout'] }</codeblock></body></topic><topic id="computing-complex-values"><title>Computing complex values</title><body><p>To compute complex values, you can use a Puppet code expression as the value of any field of a step except the <codeph>name</codeph>.</p><p>Bolt loads the plan as a YAML data structure. As it executes each step, it evaluates any expressions embedded in the step. Each plan parameter and the values of every previous named step are available in scope.</p><p>This lets you take advantage of the power of Puppet language in the places it's necessary, while keeping the rest of your plan simple.</p><p>When your plans need more sophisticated control flow or error handling beyond running a list of steps in order, it's time to convert them to <xref href="writing_plans.md#" format="dita" type="topic">Puppet language plans</xref>.</p></body></topic><topic id="converting-yaml-plans-to-puppet-plans"><title>Converting YAML plans to Puppet plans</title><body><p>You can convert a YAML plan to a Puppet plan with the <codeph>bolt plan convert</codeph> command.</p><codeblock xml:space="preserve">bolt plan convert path/to/my/plan.yaml</codeblock><p>This command takes the relative or absolute path to the YAML plan to be converted and prints the converted Puppet plan to stdout.</p><p><b>Note:</b> Converting a YAML plan might result in a Puppet plan which is syntactically correct, but behaviorally different from its YAML plan. Always manually verify a converted Puppet plan's functionality. If you convert a YAML plan to Puppet and it doesn't have the same behavior as the YAML plan, you can <xref href="https://github.com/puppetlabs/bolt/issues" format="html" scope="external">file an issue in our</xref>Git repo.</p><p>For example, with this YAML plan:</p><codeblock xml:space="preserve"># site-modules/mymodule/plans/yamlplan.yaml
parameters:
  nodes:
    type: TargetSpec
steps:
  - name: run_task
    task: sample
    target: $nodes
    parameters:
      message: "hello world"
return: $run_task</codeblock><p>Run the following conversion:</p><codeblock xml:space="preserve">$ bolt plan convert site-modules/mymodule/plans/yamlplan.yaml
# WARNING: This is an autogenerated plan. It may not behave as expected.
plan mymodule::yamlplan(
  TargetSpec $nodes
) {
  $run_task = run_task('sample', $nodes, {'message' =&gt; "hello world"})
  return $run_task
}</codeblock></body></topic></topic>
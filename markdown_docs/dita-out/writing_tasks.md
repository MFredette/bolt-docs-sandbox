<?xml version="1.0" encoding="UTF-8"?><?path2rootmap-uri ./?>
<!DOCTYPE topic
  PUBLIC "-//OASIS//DTD DITA Topic//EN" "topic.dtd">
<topic id="writing-tasks"><title>Writing tasks</title><prolog><author>Jean Bond &lt;jean@puppet.com\&gt;</author></prolog><body><p>Bolt tasks are similar to scripts, but they are kept in modules and can have metadata. This allows you to reuse and share them.</p><p>You can write tasks in any programming language the target nodes run, such as Bash, PowerShell, or Python. A task can even be a compiled binary that runs on the target. Place your task in the <codeph>./tasks</codeph> directory of a module and add a metadata file to describe parameters and configure task behavior.</p><p>For a task to run on remote \*nix systems, it must include a shebang \(<codeph>#!</codeph>\) line at the top of the file to specify the interpreter.</p><p>For example, the Puppet <codeph>mysql::sql</codeph> task is written in Ruby and provides the path to the Ruby interpreter. This example also accepts several parameters as JSON on <codeph>stdin</codeph> and returns an error.</p><codeblock xml:space="preserve">#!/opt/puppetlabs/puppet/bin/ruby
require 'json'
require 'open3'
require 'puppet'

def get(sql, database, user, password)
  cmd = ['mysql', '-e', "#{sql} "]
  cmd &lt;&lt; "--database=#{database}" unless database.nil?
  cmd &lt;&lt; "--user=#{user}" unless user.nil?
  cmd &lt;&lt; "--password=#{password}" unless password.nil?
  stdout, stderr, status = Open3.capture3(*cmd) # rubocop:disable Lint/UselessAssignment
  raise Puppet::Error, _("stderr: ' %{stderr}') % { stderr: stderr }") if status != 0
  { status: stdout.strip }
end

params = JSON.parse(STDIN.read)
database = params['database']
user = params['user']
password = params['password']
sql = params['sql']

begin
  result = get(sql, database, user, password)
  puts result.to_json
  exit 0
rescue Puppet::Error =&gt; e
  puts({ status: 'failure', error: e.message }.to_json)
  exit 1
end</codeblock><p><b>Parent topic:</b><xref href="writing_tasks_and_plans.md" format="dita" type="topic">Tasks and plans</xref></p></body><topic id="secure-coding-practices-for-tasks"><title>Secure coding practices for tasks</title><body><p>Use secure coding practices when you write tasks and help protect your system.</p><p><b>Note:</b> The information in this topic covers basic coding practices for writing secure tasks. It is not an exhaustive list.</p><p>One of the methods attackers use to gain access to your systems is remote code execution, where by running an allowed script they gain access to other parts of the system and can make arbitrary changes. Because Bolt executes scripts across your infrastructure, it is important to be aware of certain vulnerabilities, and to code tasks in a way that guards against remote code execution.</p><p>Adding task metadata that validates input is one way to reduce vulnerability. When you require an enumerated \(<codeph>enum</codeph>\) or other non-string types, you prevent improper data from being entered. An arbitrary string parameter does not have this assurance.</p><p>For example, if your task has a parameter that selects from several operational modes that are passed to a shell command, instead of</p><codeblock xml:space="preserve">String $mode = 'file'</codeblock><p>use</p><codeblock xml:space="preserve">Enum[file,directory,link,socket] $mode = file</codeblock><p>If your task has a parameter that identifies a file on disk, ensure that a user can't specify a relative path that takes them into areas where they shouldn't be. Reject file names that have slashes.</p><p>Instead of</p><codeblock xml:space="preserve">String $path</codeblock><p>use</p><codeblock xml:space="preserve">Pattern[/\A[^\/\\]*\z/] $path</codeblock><p>In addition to these task restrictions, different scripting languages each have their own ways to validate user input.</p></body><topic id="powershell"><title>PowerShell</title><body><p>In PowerShell, code injection exploits calls that specifically evaluate code. Do not call <codeph>Invoke-Expression</codeph> or <codeph>Add-Type</codeph> with user input. These commands evaluate strings as C\# code.</p><p>Reading sensitive files or overwriting critical files can be less obvious. If you plan to allow users to specify a file name or path, use <codeph>Resolve-Path</codeph> to verify that the path doesn't go outside the locations you expect the task to access. Use <codeph>Split-Path -Parent $path</codeph> to check that the resolved path has the desired path as a parent.</p><p>For more information, see <xref href="https://docs.microsoft.com/en-us/powershell/scripting/PowerShell-Scripting?view=powershell-6" format="html" scope="external">PowerShell Scripting</xref> and <xref href="https://blogs.msdn.microsoft.com/powershell/2008/09/30/powershells-security-guiding-principles/" format="html" scope="external">Powershell's Security Guiding Principles</xref>.</p></body></topic><topic id="bash"><title>Bash</title><body><p>In Bash and other command shells, shell command injection takes advantage of poor shell implementations. Put quotations marks around arguments to prevent the vulnerable shells from evaluating them.</p><p>Because the <codeph>eval</codeph> command evaluates all arguments with string substitution, avoid using it with user input; however you can use <codeph>eval</codeph> with sufficient quoting to prevent substituted variables from being executed.</p><p>Instead of</p><codeblock xml:space="preserve">eval "echo $input"</codeblock><p>use</p><codeblock xml:space="preserve">eval "echo '$input'"</codeblock><p>These are operating system-specific tools to validate file paths: <codeph>realpath</codeph> or <codeph>readlink -f</codeph>.</p></body></topic><topic id="python"><title>Python</title><body><p>In Python malicious code can be introduced through commands like <codeph>eval</codeph>, <codeph>exec</codeph>, <codeph>os.system</codeph>, <codeph>os.popen</codeph>, and <codeph>subprocess.call</codeph> with <codeph>shell=True</codeph>. Use <codeph>subprocess.call</codeph> with <codeph>shell=False</codeph> when you include user input in a command or escape variables.</p><p>Instead of</p><codeblock xml:space="preserve">os.system('echo '+input)
</codeblock><p>use</p><codeblock xml:space="preserve">subprocess.check_output(['echo', input])</codeblock><p>Resolve file paths with <codeph>os.realpath</codeph> and confirm them to be within another path by looping over <codeph>os.path.dirname</codeph> and comparing to the desired path.</p><p>For more information on the vulnerabilities of Python or how to escape variables, see Kevin London's blog post on <xref href="https://www.kevinlondon.com/2015/07/26/dangerous-python-functions.html" format="html" scope="external">Dangerous Python Functions</xref>.</p></body></topic><topic id="ruby"><title>Ruby</title><body><p>In Ruby, command injection is introduced through commands like <codeph>eval</codeph>, <codeph>exec</codeph>, <codeph>system</codeph>, backtick \(\`\`\) or <codeph>%x()</codeph> execution, or the Open3 module. You can safely call these functions with user input by passing the input as additional arguments instead of a single string.</p><p>Instead of</p><codeblock xml:space="preserve">system("echo #{flag1} #{flag2}")</codeblock><p>use</p><codeblock xml:space="preserve">system('echo', flag1, flag2)</codeblock><p>Resolve file paths with <codeph>Pathname#realpath</codeph>, and confirm them to be within another path by looping over <codeph>Pathname#parent</codeph> and comparing to the desired path.</p><p>For more information on securely passing user input, see the blog post <xref href="https://www.hilman.io/blog/2016/01/stop-using-backtick-to-run-shell-command-in-ruby/" format="html" scope="external">Stop using backtick to run shell command in Ruby</xref>.</p></body></topic></topic><topic id="naming-tasks"><title>Naming tasks</title><body><p>Task names are named based on the filename of the task, the name of the module, and the path to the task within the module.</p><p>You can write tasks in any language that runs on the target nodes. Give task files the extension for the language they are written in \(such as <codeph>.rb</codeph> for Ruby\), and place them in the top level of your module's <codeph>./tasks</codeph> directory.</p><p>Task names are composed of one or two name segments, indicating:</p><ul><li><p>The name of the module where the task is located.</p></li><li><p>The name of the task file, without the extension.</p></li></ul><p>For example, the <codeph>puppetlabs-mysql</codeph> module has the <codeph>sql</codeph> task in <codeph>./mysql/tasks/sql.rb</codeph>, so the task name is <codeph>mysql::sql</codeph>. This name is how you refer to the task when you run tasks.</p><p>The task filename <codeph>init</codeph> is special: the task it defines is referenced using the module name only. For example, in the <codeph>puppetlabs-service</codeph> module, the task defined in <codeph>init.rb</codeph> is the <codeph>service</codeph> task.</p><p>Each task or plan name segment must begin with a lowercase letter and:</p><ul><li><p>Must start with a lowercase letter.</p></li><li><p>May include digits.</p></li><li><p>May include underscores.</p></li><li><p>Namespace segments must match the following regular expression <codeph>\A[a-z][a-z0-9_]*\Z</codeph></p></li><li><p>The file extension must not use the reserved extensions .md or .json.</p></li></ul></body><topic id="single-platform-tasks"><title>Single-platform tasks</title><body><p>A task can consist of a single executable with or without a corresponding metadata file. For instance, <codeph>./mysql/tasks/sql.rb</codeph> and <codeph>./mysql/tasks/sql.json</codeph>. In this case, no other <codeph>./mysql/tasks/sql.*</codeph> files can exist.</p></body></topic><topic id="cross-platform-tasks"><title>Cross-platform tasks</title><body><p>A task can have multiple implementations, with metadata that explains when to use each one. A primary use case for this is to support different implementations for different target platforms, referred to as cross-platform tasks.</p><p>A task can also have multiple implementations, with metadata that explains when to use each one. A primary use case for this is to support different implementations for different target platforms, referred to as <codeph>cross-platform tasks</codeph>. For instance, consider a module with the following files:</p><codeblock xml:space="preserve">- tasks
  - sql_linux.sh
  - sql_linux.json
  - sql_windows.ps1
  - sql_windows.json
  - sql.json</codeblock><p>This task has two executables \(<codeph>sql_linux.sh</codeph> and <codeph>sql_windows.ps1</codeph>\), each with an implementation metadata file and a task metadata file. The executables have distinct names and are compatible with older task runners such as Puppet Enterprise 2018.1 and earlier. Each implementation has it's own metadata which documents how to use the implementation directly or marks it as private to hide it from UI lists.</p><p>An implementation metadata example:</p><codeblock xml:space="preserve">{
  "name": "SQL Linux",
  "description": "A task to perform sql operations on linux targets",
  "private": true
}</codeblock><p>The task metadata file contains an implementations section:</p><codeblock xml:space="preserve">{
  "implementations": [
    {"name": "sql_linux.sh", "requirements": ["shell"]},
    {"name": "sql_windows.ps1", "requirements": ["powershell"]}
  ]
}</codeblock><p>Each implementations has a <codeph>name</codeph> and a list of <codeph>requirements</codeph>. The requirements are the set of <i>features</i> which must be available on the target in order for that implementation to be used. In this case, the <codeph>sql_linux.sh</codeph> implementation requires the <codeph>shell</codeph> feature, and the <codeph>sql_windows.ps1</codeph> implementations requires the PowerShell feature.</p><p>The set of features available on the target is determined by the task runner. You can specify additional features for a target via <codeph>set_feature</codeph> or by adding <codeph>features</codeph> in the inventory. The task runner chooses the <i>first</i>implementation whose requirements are satisfied.</p><p>The following features are defined by default:</p><ul><li><p><codeph>puppet-agent</codeph>: Present if the target has the Puppet agent package installed. This feature is automatically added to hosts with the name <codeph>localhost</codeph>.</p></li><li><p><codeph>shell</codeph>: Present if the target has a posix shell.</p></li><li><p><codeph>powershell</codeph>: Present if the target has PowerShell.</p></li></ul></body></topic></topic><topic id="sharing-executables"><title>Sharing executables</title><body><p>Multiple task implementations can refer to the same executable file.</p><p>Executables can access the <codeph>_task</codeph> metaparameter, which contains the task name. For example, the following creates the tasks <codeph>service::stop</codeph> and <codeph>service::start</codeph>, which live in the executable but appear as two separate tasks.</p><codeblock xml:space="preserve">myservice/tasks/init.rb</codeblock><codeblock xml:space="preserve">#!/usr/bin/env ruby
require 'json'

params = JSON.parse(STDIN.read)
action = params['action'] || params['_task']
if ['start',  'stop'].include?(action)
  `systemctl #{params['_task']} #{params['service']}`
end
</codeblock><codeblock xml:space="preserve">myservice/tasks/start.json</codeblock><codeblock xml:space="preserve">{
  "description": "Start a service",
  "parameters": {
    "service": {
      "type": "String",
      "description": "The service to start"
    }
  },
  "implementations": [
    {"name": "init.rb"}
  ]
}</codeblock><codeblock xml:space="preserve">myservice/tasks/stop.json</codeblock><codeblock xml:space="preserve">{
  "description": "Stop a service",
  "parameters": {
    "service": {
      "type": "String",
      "description": "The service to stop"
    }
  },
  "implementations": [
    {"name": "init.rb"}
  ]
}</codeblock></body></topic><topic id="sharing-task-code"><title>Sharing task code</title><body><p>Multiple tasks can share common files between them. Tasks can additionally pull library code from other modules.</p><p>To create a task that includes additional files pulled from modules, include the files property in your metadata as an array of paths. A path consists of:</p><ul><li><p>the module name</p></li><li><p>one of the following directories within the module:</p><ul><li><p><codeph>files</codeph> — Most helper files. This prevents the file from being treated as a task or added to the Puppet Ruby loadpath.</p></li><li><p><codeph>tasks</codeph> — Helper files that can be called as tasks on their own.</p></li><li><p><codeph>lib</codeph> — Ruby code that might be reused by types, providers, or Puppet functions.</p></li></ul></li><li><p>the remaining path to a file or directory; directories must include a trailing slash <codeph>/</codeph></p></li></ul><p>All path separators must be forward slashes. An example would be <codeph>stdlib/lib/puppet/</codeph>.</p><p>The <codeph>files</codeph> property can be included both as a top-level metadata property, and as a property of an implementation, for example:</p><codeblock xml:space="preserve">{
  "implementations": [
    {"name": "sql_linux.sh", "requirements": ["shell"], "files": ["mymodule/files/lib.sh"]},
    {"name": "sql_windows.ps1", "requirements": ["powershell"], "files": ["mymodule/files/lib.ps1"]}
  ],
  "files": ["emoji/files/emojis/"]
}</codeblock><p>When a task includes the <codeph>files</codeph> property, all files listed in the top-level property and in the specific implementation chosen for a target are copied to a temporary directory on that target. The directory structure of the specified files is preserved such that paths specified with the <codeph>files</codeph> metadata option are available to tasks prefixed with <codeph>_installdir</codeph>. The task executable itself is located in its module location under the <codeph>_installdir</codeph> as well, so other files can be found at <codeph>../../mymodule/files/</codeph>relative to the task executable's location.</p><p>For example, you can create a task and metadata in a module at <codeph>~/.puppetlabs/bolt/site-modules/mymodule/tasks/task.{json,rb}</codeph>.</p><p><b>Metadata</b></p><codeblock xml:space="preserve">{
  "files": ["multi_task/files/rb_helper.rb"]
}</codeblock><p><b>File resource</b></p><p><codeph>multi_task/files/rb_helper.rb</codeph></p><codeblock xml:space="preserve">def useful_ruby
  { helper: "ruby" }
end</codeblock><p><b>Task</b></p><codeblock xml:space="preserve">#!/usr/bin/env ruby
require 'json'

params = JSON.parse(STDIN.read)
require_relative File.join(params['_installdir'], 'multi_task', 'files', 'rb_helper.rb')
# Alternatively use relative path
# require_relative File.join(__dir__, '..', '..', 'multi_task', 'files', 'rb_helper.rb')
 puts useful_ruby.to_json</codeblock><p><b>Output</b></p><codeblock xml:space="preserve">Started on localhost...
Finished on localhost:
  {
    "helper": "ruby"
  }
Successful on 1 node: localhost
Ran on 1 node in 0.12 seconds</codeblock></body><topic id="task-helpers"><title>Task helpers</title><body><p>To help with writing tasks, Bolt includes <xref href="https://github.com/puppetlabs/puppetlabs-python_task_helper" format="html" scope="external">python\_task\_helper</xref> and <xref href="https://github.com/puppetlabs/puppetlabs-ruby_task_helper" format="html" scope="external">ruby\_task\_helper</xref>. It also makes a useful demonstration of including code from another module.</p></body></topic><topic id="python-example"><title>Python example</title><body><p>Create task and metadata in a module at <codeph>~/.puppetlabs/bolt/site-modules/mymodule/tasks/task.{json,py}</codeph>.</p><p><b>Metadata</b></p><codeblock xml:space="preserve">{
  "files": ["python_task_helper/files/task_helper.py"],
  "input_method": "stdin"
}</codeblock><p><b>Task</b></p><codeblock xml:space="preserve">#!/usr/bin/env python
import os, sys
sys.path.append(os.path.join(os.path.dirname(__file__), '..', '..', 'python_task_helper', 'files'))
from task_helper import TaskHelper

class MyTask(TaskHelper):
  def task(self, args):
    return {'greeting': 'Hi, my name is '+args['name']}

if __name__ == '__main__':
    MyTask().run()</codeblock><p><b>Output</b></p><codeblock xml:space="preserve">$ bolt task run mymodule::task -n localhost name='Julia'
Started on localhost...
Finished on localhost:
  {
    "greeting": "Hi, my name is Julia"
  }
Successful on 1 node: localhost
Ran on 1 node in 0.12 seconds</codeblock></body></topic><topic id="ruby-example"><title>Ruby example</title><body><p>Create task and metadata in a new module at <codeph>~/.puppetlabs/bolt/site-modules/mymodule/tasks/mytask.{json,rb}</codeph>.</p><p><b>Metadata</b></p><codeblock xml:space="preserve">{
  "files": ["ruby_task_helper/files/task_helper.rb"],
  "input_method": "stdin"
}</codeblock><p><b>Task</b></p><codeblock xml:space="preserve">#!/usr/bin/env ruby
require_relative '../../ruby_task_helper/files/task_helper.rb'

class MyTask &lt; TaskHelper 
  def task(name: nil, **kwargs)
    { greeting: "Hi, my name is #{name}" }
  end
end


MyTask.run if __FILE__ == $0</codeblock><p><b>Output</b></p><codeblock xml:space="preserve">$ bolt task run mymodule::mytask -n localhost name="Robert'); DROP TABLE Students;--"
Started on localhost...
Finished on localhost:
  {
    "greeting": "Hi, my name is Robert'); DROP TABLE Students;--"
  }
Successful on 1 node: localhost
Ran on 1 node in 0.12 seconds</codeblock></body></topic></topic><topic id="writing-remote-tasks"><title>Writing remote tasks</title><body><p>Some targets are hard or impossible to execute tasks on directly. In these cases, you can write a task that runs on a proxy target and remotely interacts with the real target.</p><p>For example, a network device might have a limited shell environment or a cloud service might be driven only by HTTP APIs. By writing a remote task, Bolt allows you to specify connection information for remote targets in their inventory file and injects them into the <codeph>_target</codeph> metaparam.</p><p>This example shows how to write a task that posts messages to Slack and reads connection information from <codeph>inventory.yaml</codeph>:</p><codeblock xml:space="preserve">#!/usr/bin/env ruby
# modules/slack/tasks/message.rb

require 'json'
require 'net/http'

params = JSON.parse(STDIN.read)
# the slack API token is passed in from inventory
token = params['_target']['token']
		
uri = URI('https://slack.com/api/chat.postMessage')
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

req = Net::HTTP::Post.new(uri, 'Content-type' =&gt; 'application/json')
req['Authorization'] = "Bearer #{params['_target']['token']}"
req.body = { channel: params['channel'], text: params['message'] }.to_json

resp = http.request(req)

puts resp.body</codeblock><p>To prevent accidentally running a normal task on a remote target and breaking its configuration, Bolt won't run a task on a remote target unless its metadata defines it as remote:</p><codeblock xml:space="preserve">{
  "remote": true
}</codeblock><p>Add Slack as a remote target in your inventory file:</p><codeblock xml:space="preserve">---
nodes:
  - name: my_slack
    config:
      transport: remote
      remote:
	token: &lt;SLACK_API_TOKEN&gt;</codeblock><p>Finally, make <codeph>my_slack</codeph> a target that can run the <codeph>slack::message</codeph>:</p><codeblock xml:space="preserve">bolt task run slack::message --nodes my_slack message="hello" channel=&lt;slack channel id&gt;</codeblock></body></topic><topic id="defining-parameters-in-tasks"><title>Defining parameters in tasks</title><body><p>Allow your task to accept parameters as either environment variables or as a JSON hash on standard input.</p><p>Tasks can receive input as either environment variables, a JSON hash on standard input, or as PowerShell arguments. By default, the task runner submits parameters as both environment variables and as JSON on <codeph>stdin</codeph>.</p><p>If your task should receive parameters only in a certain way, such as <codeph>stdin</codeph> only, you can set the input method in your task metadata. For Windows tasks, it's usually better to use tasks written in PowerShell. See the related topic about task metadata for information about setting the input method.</p><p>Environment variables are the easiest way to implement parameters, and they work well for simple JSON types such as strings and numbers. For arrays and hashes, use structured input instead because parameters with undefined values \(<codeph>nil</codeph>, <codeph>undef</codeph>\) passed as environment variables have the <codeph>String</codeph> value <codeph>null</codeph>. For more information, see <xref href="writing_tasks.md#" format="dita" type="topic">Structured input and output</xref>.</p><p>To add parameters to your task as environment variables, pass the argument prefixed with the Puppet task prefix <codeph>PT_</codeph> .</p><p>For example, to add a <codeph>message</codeph> parameter to your task, read it from the environment in task code as <codeph>PT_message</codeph>. When the user runs the task, they can specify the value for the parameter on the command line as <codeph>message=hello</codeph>, and the task runner submits the value <codeph>hello</codeph> to the <codeph>PT_message</codeph> variable.</p><codeblock xml:space="preserve">#!/usr/bin/env bash
echo your message is $PT_message</codeblock></body><topic id="defining-parameters-in-windows"><title>Defining parameters in Windows</title><body><p>For Windows tasks, you can pass parameters as environment variables, but it's easier to write your task in PowerShell and use named arguments. By default tasks with a <codeph>.ps1</codeph> extension use PowerShell standard argument handling.</p><p>For example, this PowerShell task takes a process name as an argument and returns information about the process. If no parameter is passed by the user, the task returns all of the processes.</p><codeblock xml:space="preserve">[CmdletBinding()]
Param(
  [Parameter(Mandatory = $False)]
 [String]
  $Name
  )

if ($Name -eq $null -or $Name -eq "") {
  Get-Process
} else {
  $processes = Get-Process -Name $Name
  $result = @()
  foreach ($process in $processes) {
    $result += @{"Name" = $process.ProcessName;
                 "CPU" = $process.CPU;
                 "Memory" = $process.WorkingSet;
                 "Path" = $process.Path;
                 "Id" = $process.Id}
  }
  if ($result.Count -eq 1) {
    ConvertTo-Json -InputObject $result[0] -Compress
  } elseif ($result.Count -gt 1) {
    ConvertTo-Json -InputObject @{"_items" = $result} -Compress
  }
}</codeblock><p>To pass parameters in your task as environment variables \(<codeph>PT_parameter</codeph>\), you must set <codeph>input_method</codeph> in your task metadata to <codeph>environment</codeph>. To run Ruby tasks on Windows, the Puppet agent must be installed on the target nodes.</p></body></topic></topic><topic id="returning-errors-in-tasks"><title>Returning errors in tasks</title><body><p>To return a detailed error message if your task fails, include an <codeph>Error</codeph> object in the task's result.</p><p>When a task exits non-zero, the task runner checks for an error key \(\`\_error\`\). If one is not present, the task runner generates a generic error and adds it to the result. If there is no text on <codeph>stdout</codeph> but text is present on <codeph>stderr</codeph>, the <codeph>stderr</codeph> text is included in the message.</p><codeblock xml:space="preserve">{ "_error": {
    "msg": "Task exited 1:\nSomething on stderr",
    "kind": "puppetlabs.tasks/task-error",
    "details": { "exitcode": 1 }
}</codeblock><p>An error object includes the following keys:</p><ul><li><p><b>msg</b></p><p>A human readable string that appears in the UI.</p></li><li><p><b>kind</b></p><p>A standard string for machines to handle. You may share kinds between your modules or namespace kinds per module.</p></li><li><p><b>details</b></p><p>An object of structured data about the tasks.</p></li></ul><p>Tasks can provide more details about the failure by including their own error object in the result at <codeph>_error</codeph>.</p><codeblock xml:space="preserve">#!/opt/puppetlabs/puppet/bin/ruby

require 'json'

begin
  params = JSON.parse(STDIN.read)
  result = {}
  result['result'] = params['dividend'] / params['divisor']

rescue ZeroDivisionError
  result[:_error] = { msg: "Cannot divide by zero",
                      # namespace the error to this module
                      kind: "puppetlabs-example_modules/dividebyzero",
                      details: { divisor: divisor },
                    }
rescue Exception =&gt; e
  result[:_error] = { msg: e.message,
                     kind: "puppetlabs-example_modules/unknown",
                     details: { class: e.class.to_s },
                   }
end

puts result.to_json</codeblock></body></topic><topic id="structured-input-and-output"><title>Structured input and output</title><body><p>If you have a task that has many options, returns a lot of information, or is part of a task plan, consider using structured input and output with your task.</p><p>The task API is based on JSON. Task parameters are encoded in JSON, and the task runner attempts to parse the output of the tasks as a JSON object.</p><p>The task runner can inject keys into that object, prefixed with <codeph>_</codeph>. If the task does not return a JSON object, the task runner creates one and places the output in an <codeph>_output</codeph> key.</p></body><topic id="structured-input"><title>Structured input</title><body><p>For complex input, such as hashes and arrays, you can accept structured JSON in your task.</p><p>By default, the task runner passes task parameters as both environment variables and as a single JSON object on stdin. The JSON input allows the task to accept complex data structures.</p><p>To accept parameters as JSON on stdin, set the <codeph>params</codeph> key to accept JSON on <codeph>stdin</codeph>.</p><codeblock xml:space="preserve">#!/opt/puppetlabs/puppet/bin/ruby
require 'json'

params = JSON.parse(STDIN.read)

exitcode = 0
params['files'].each do |filename|
  begin
    FileUtils.touch(filename)
    puts "updated file #{filename}"
  rescue
    exitcode = 1
    puts "couldn't update file #{filename}"
  end
end
exit exitcode</codeblock><p>If your task accepts input on <codeph>stdin</codeph> it should specify <codeph>"input_method": "stdin"</codeph> in its <codeph>metadata.json</codeph> file, or it may not work with sudo for some users.</p></body></topic><topic id="returning-structured-output"><title>Returning structured output</title><body><p>To return structured data from your task, print only a single JSON object to <codeph>stdout</codeph> in your task.</p><p>Structured output is useful if you want to use the output in another program, or if you want to use the result of the task in a Puppet task plan.</p><codeblock xml:space="preserve">#!/usr/bin/env python
import json
import sys
minor = sys.version_info
result = { "major": sys.version_info.major, "minor": sys.version_info.minor }
json.dump(result, sys.stdout)</codeblock></body></topic></topic><topic id="converting-scripts-to-tasks"><title>Converting scripts to tasks</title><body><p>To convert an existing script to a task, you can either write a task that wraps the script or you can add logic in your script to check for parameters in environment variables.</p><p>If the script is already installed on the target nodes, you can write a task that wraps the script. In the task, read the script arguments as task parameters and call the script, passing the parameters as the arguments.</p><p>If the script isn't installed or you want to make it into a cohesive task so that you can manage its version with code management tools, add code to your script to check for the environment variables, prefixed with <codeph>PT_</codeph>, and read them instead of arguments.</p><p>CAUTION:</p><p>For any tasks that you intend to use with PE and assign RBAC permissions, makesure the script safely handles parameters or validate them to prevent shell injection vulnerabilities.</p><p>Given a script that accepts positional arguments on the command line:</p><codeblock xml:space="preserve">version=$1
[ -z "$version" ] &amp;&amp; echo "Must specify a version to deploy &amp;&amp; exit 1

if [ -z "$2" ]; then
  filename=$2
else
  filename=~/myfile
fi</codeblock><p>To convert the script into a task, replace this logic with task variables:</p><codeblock xml:space="preserve">version=$PT_version #no need to validate if we use metadata
if [ -z "$PT_filename" ]; then
  filename=$PT_filename
else
  filename=~/myfile
fi</codeblock></body><topic id="wrapping-an-existing-script"><title>Wrapping an existing script</title><body><p>If a script is not already installed on targets and you don't want to edit it, for example if it's a script someone else maintains, you can wrap the script in a small task without modifying it.</p><p>CAUTION:</p><p>For any tasks that you intend to use with PE and assign RBAC permissions, makesure the script safely handles parameters or validate them to prevent shell injection vulnerabilities.</p><p>Given a script, <codeph>myscript.sh</codeph>, that accepts 2 positional args, <codeph>filename</codeph> and <codeph>version</codeph>:</p><ol><li><p>Copy the script to the module's <codeph>files/</codeph> directory.</p></li><li><p>Create a metadata file for the task that includes the parameters and file dependency.</p></li></ol><codeblock xml:space="preserve">{
     "input_method": "environment",
     "parameters": {
       "filename": { "type": "String[1]" },
       "version": { "type": "String[1]" }
     },
     "files": [ "script_example/files/myscript.sh" ]
   }</codeblock><ol><li><p>Create a small wrapper task that reads environment variables and calls the task.</p><codeblock xml:space="preserve">#!/usr/bin/env bash
set -e

script_file="$PT__installdir/script_example/files/myscript.sh"
# If this task is going to be run from windows nodes the wrapper must make sure it's exectutable
chmod +x $script_file
commandline=("$script_file" "$PT_filename" "$PT_version")
# If the stderr output of the script is important redirect it to stdout.
"${commandline[@]}" 2&gt;&amp;1</codeblock></li></ol></body></topic></topic><topic id="supporting-no-op-in-tasks"><title>Supporting no-op in tasks</title><body><p>Tasks support no-operation functionality, also known as no-op mode. This function shows what changes the task would make, without actually making those changes.</p><p>No-op support allows a user to pass the <codeph>--noop</codeph> flag with a command to test whether the task will succeed on all targets before making changes.</p><p>To support no-op, your task must include code that looks for the <codeph>_noop</codeph> metaparameter. No-op is supported only in Puppet Enterprise.</p><p>If the user passes the <codeph>--noop</codeph> flag with their command, this parameter is set to <codeph>true</codeph>, and your task must not make changes. You must also set <codeph>supports_noop</codeph> to <codeph>true</codeph> in your task metadata or the task runner will refuse to run the task in noop mode.</p></body><topic id="no-op-metadata-example"><title>No-op metadata example</title><body><codeblock xml:space="preserve">{
  "description": "Write content to a file.",
  "supports_noop": true,
  "parameters": {
    "filename": {
      "description": "the file to write to",
      "type": "String[1]"
    },
    "content": {
      "description": "The content to write",
      "type": "String"
    }
  }
}</codeblock></body></topic><topic id="no-op-task-example"><title>No-op task example</title><body><codeblock xml:space="preserve">#!/usr/bin/env python
import json
import os
import sys

params = json.load(sys.stdin)
filename = params['filename']
content = params['content']
noop = params.get('_noop', False)

exitcode = 0

def make_error(msg):
  error = {
      "_error": {
          "kind": "file_error",
          "msg": msg,
          "details": {},
      }
  }
  return error

try:
  if noop:
    path = os.path.abspath(os.path.join(filename, os.pardir))
    file_exists = os.access(filename, os.F_OK)
    file_writable = os.access(filename, os.W_OK)
    path_writable = os.access(path, os.W_OK)

    if path_writable == False:
      exitcode = 1
      result = make_error("Path %s is not writable" % path)
    elif file_exists == True and file_writable == False:
      exitcode = 1
      result = make_error("File %s is not writable" % filename)
    else:
      result = { "success": True , '_noop': True }
  else:
    with open(filename, 'w') as fh:
      fh.write(content)
      result = { "success": True }
except Exception as e:
  exitcode = 1
  result = make_error("Could not open file %s: %s" % (filename, str(e)))
print(json.dumps(result))
exit(exitcode)</codeblock></body></topic></topic><topic id="task-metadata"><title>Task metadata</title><body><p>Task metadata files describe task parameters, validate input, and control how the task runner executes the task.</p><p>Your task must have metadata to be published and shared on the Forge. Specify task metadata in a JSON file with the naming convention <codeph>&lt;TASKNAME&gt;.json</codeph> . Place this file in the module's <codeph>./tasks</codeph> folder along with your task file.</p><p>For example, the module <codeph>puppetlabs-mysql</codeph> includes the <codeph>mysql::sql</codeph> task with the metadata file, <codeph>sql.json</codeph>.</p><codeblock xml:space="preserve">{
  "description": "Allows you to execute arbitrary SQL",
  "input_method": "stdin",
  "parameters": {
    "database": {
      "description": "Database to connect to",
      "type": "Optional[String[1]]"
    },
    "user": {
      "description": "The user",
      "type": "Optional[String[1]]"
    },
    "password": {
      "description": "The password",
      "type": "Optional[String[1]]",
      "sensitive": true
    },
     "sql": {
      "description": "The SQL you want to execute",
      "type": "String[1]"
    }
  }
}</codeblock></body><topic id="adding-parameters-to-metadata"><title>Adding parameters to metadata</title><body><p>To document and validate task parameters, add the parameters to the task metadata as JSON object, <codeph>parameters</codeph>.</p><p>If a task includes <codeph>parameters</codeph> in its metadata, the task runner rejects any parameters input to the task that aren't defined in the metadata.</p><p>In the <codeph>parameter</codeph> object, give each parameter a description and specify its Puppet type. For a complete list of types, see the <xref href="https://docs.puppet.com/puppet/latest/lang_data_type.html" format="html" scope="external">types documentation</xref>.</p><p>For example, the following code in a metadata file describes a <codeph>provider</codeph> parameter:</p><codeblock xml:space="preserve">"provider": {
  "description": "The provider to use to manage or inspect the service, defaults to the system service manager",
  "type": "Optional[String[1]]"
 }</codeblock></body><topic id="define-sensitive-parameters"><title>Define sensitive parameters</title><body><p>You can define task parameters as sensitive, for example, passwords and API keys. These values are masked when they appear in logs and API responses. When you want to view these values, set the log file to <codeph>level: debug</codeph>.</p><p>To define a parameter as sensitive within the JSON metadata, add the <codeph>"sensitive": true</codeph> property.</p><codeblock xml:space="preserve">{
  "description": "This task has a sensitive property denoted by its metadata",
  "input_method": "stdin",
  "parameters": {
    "user": {
      "description": "The user",
      "type": "String[1]"
    },
    "password": {
      "description": "The password",
      "type": "String[1]",
      "sensitive": true
    }
  }
}</codeblock></body></topic></topic><topic id="task-metadata-reference"><title>Task metadata reference</title><body><p>The following table shows task metadata keys, values, and default values.</p></body><topic id="task-metadata-1"><title><b>Task metadata</b></title><body><table><tgroup cols="4"><colspec colname="col1" colnum="1"/><colspec colname="col2" colnum="2"/><colspec colname="col3" colnum="3"/><colspec colname="col4" colnum="4"/><thead><row><entry colname="col1">Metadata key</entry><entry colname="col2">Description</entry><entry colname="col3">Value</entry><entry colname="col4">Default</entry></row></thead><tbody><row><entry colname="col1">"description"</entry><entry colname="col2">A description of what the task does.</entry><entry colname="col3">String</entry><entry colname="col4">None</entry></row><row><entry colname="col1">"input\_method"</entry><entry colname="col2">What input method the task runner should use to pass parameters to the task.</entry><entry colname="col3">-   <codeph>environment</codeph></entry><entry colname="col4"/></row></tbody></tgroup></table><ul><li><p><codeph>stdin</codeph></p></li><li><p><codeph>powershell</codeph></p></li></ul><p>|Both <codeph>environment</codeph> and <codeph>stdin</codeph> unless <codeph>.ps1</codeph> tasks, in which case <codeph>powershell</codeph></p><p>|
|"parameters"|The parameters or input the task accepts listed with a puppet type string and optional description. See <xref href="writing_tasks.md#" format="dita" type="topic">adding parameters to metadata</xref> for usage information.|Array of objects describing each parameter|None|
|"puppet\_task\_version"|The version of the spec used.|Integer|1 \(This is the only valid value.\)|
|"supports\_noop"|Whether the task supports no-op mode. Required for the task to accept the <codeph>--noop</codeph> option on the command line.|Boolean|False|
|"implementations"|A list of task implementations and the requirements used to select one to run. See <xref href="writing_tasks.md#" format="dita" type="topic">Cross-platform tasks</xref> for usage information.|Array of Objects describing each implementation|None|
|"files"|A list of files to be provided when running the task, addressed by module. See <xref href="writing_tasks.md#" format="dita" type="topic">Sharing task code</xref> for usage information.|Array of Strings|None|
|"private"|Do not display task by default when listing for UI.|Boolean|False|</p></body></topic></topic><topic id="task-metadata-types"><title>Task metadata types</title><body><p>Task metadata can accept most Puppet data types.</p></body><topic id="common-task-data-types"><title>Common task data types</title><body><p><b>Restriction:</b></p><p>Some types supported by Puppet can not be represented as JSON, such as <codeph>Hash[Integer, String]</codeph>, <codeph>Object</codeph>, or <codeph>Resource</codeph>. These should not be used in tasks, because they can never be matched.</p><table><tgroup cols="2"><colspec colname="col1" colnum="1"/><colspec colname="col2" colnum="2"/><thead><row><entry colname="col1">Type</entry><entry colname="col2">Description</entry></row></thead><tbody><row><entry colname="col1"><codeph>String</codeph></entry><entry colname="col2">Accepts any string.</entry></row><row><entry colname="col1"><codeph>String[1]</codeph></entry><entry colname="col2">Accepts any non-empty string \(a String of at least length 1\).</entry></row><row><entry colname="col1"><codeph>Enum[choice1, choice2]</codeph></entry><entry colname="col2">Accepts one of the listed choices.</entry></row><row><entry colname="col1"><codeph>Pattern[/\A\w+\Z/]</codeph></entry><entry colname="col2">Accepts Strings matching the regex <codeph>/\w+/</codeph> or non-empty strings of word characters.</entry></row><row><entry colname="col1"><codeph>Integer</codeph></entry><entry colname="col2">Accepts integer values. JSON has no Integer type so this can vary depending on input.</entry></row><row><entry colname="col1"><codeph>Optional[String[1]]</codeph></entry><entry colname="col2">Optional makes the parameter optional and permits null values. Tasks have no required nullable values.</entry></row><row><entry colname="col1"><codeph>Array[String]</codeph></entry><entry colname="col2">Matches an array of strings.</entry></row><row><entry colname="col1"><codeph>Hash</codeph></entry><entry colname="col2">Matches a JSON object.</entry></row><row><entry colname="col1"><codeph>Variant[Integer, Pattern[/\A\d+\Z/]]</codeph></entry><entry colname="col2">Matches an integer or a String of an integer</entry></row><row><entry colname="col1"><codeph>Boolean</codeph></entry><entry colname="col2">Accepts Boolean values.</entry></row></tbody></tgroup></table><p><b>Related information</b></p><p><xref href="https://puppet.com/docs/puppet/latest/lang_data_type.html" format="html" scope="external">Data type syntax</xref></p></body></topic></topic></topic><topic id="specifying-parameters"><title>Specifying parameters</title><body><p>Parameters for tasks can be passed to the <codeph>bolt</codeph> command as CLI arguments or as a JSON hash.</p><p>To pass parameters individually to your task or plan, specify the parameter value on the command line in the format <codeph>parameter=value</codeph>. Pass multiple parameters as a space-separated list. Bolt attempts to parse each parameter value as JSON and compares that to the parameter type specified by the task or plan. If the parsed value matches the type, it is used; otherwise, the original string is used.</p><p>For example, to run the <codeph>mysql::sql</codeph>task to show tables from a database called <codeph>mydatabase</codeph>:</p><codeblock xml:space="preserve">bolt task run mysql::sql database=mydatabase sql="SHOW TABLES" --nodes neptune --modules ~/modules
</codeblock><p>To pass a string value that is valid JSON to a parameter that would accept both quote the string. For example to pass the string <codeph>true</codeph> to a parameter of type <codeph>Variant[String, Boolean]</codeph> use <codeph>'foo="true"'</codeph>. To pass a String value wrapped in <codeph>"</codeph> quote and escape it <codeph>'string="\"val\"'</codeph>. Alternatively, you can specify parameters as a single JSON object with the <codeph>--params</codeph> flag, passing either a JSON object or a path to a parameter file.</p><p>To specify parameters as JSON, use the parameters flag followed by the JSON: <codeph>--params '{"name": "openssl"}'</codeph></p><p>To set parameters in a file, specify parameters in JSON format in a file, such as <codeph>params.json</codeph>. For example, create a <codeph>params.json</codeph> file that contains the following JSON:</p><codeblock xml:space="preserve">{
  "name":"openssl"
}</codeblock><p>Then specify the path to that file \(starting with an at symbol, <codeph>@</codeph>\) on the command line with the parameters flag: <codeph>--params @params.json</codeph></p></body></topic></topic>
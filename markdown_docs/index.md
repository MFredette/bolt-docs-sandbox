# Bolt

-   [Welcome to Bolt](bolt.md)
-   [Bolt release notes](bolt_release_notes.md)
    -   [Bolt versioning](bolt_versioning.md)
    -   [New features](bolt_new_features.md)
    -   [Resolved issues](bolt_resolved_issues.md)
    -   [Known issues](bolt_known_issues.md)
    -   [Deprecations and removals](bolt_deprecations_and_removals.md)
-   [Installing Bolt](bolt_installing.md#concept-8499)
    -   [Install Bolt on Windows](bolt_installing.md#task-8890)
        -   [Install Bolt with MSI ](bolt_installing.md#task-5218)
        -   [Install Bolt with Chocolatey](bolt_installing.md#task-9426)
    -   [Install Bolt on macOS](bolt_installing.md#task-3621)
        -   [Install Bolt with macOS installer ](bolt_installing.md#task-1894)
        -   [Install Bolt with Homebrew](bolt_installing.md#task-5322)
    -   [Install Bolt on \*nix](bolt_installing.md#task-4877)
        -   [Install Bolt on Debian or Ubuntu](bolt_installing.md#task-7569)
        -   [Install Bolt on RHEL, SLES, or Fedora](bolt_installing.md#task-5651)
    -   [Install gems with Bolt packages](bolt_installing.md#task-2023)
    -   [Install Bolt as a gem](bolt_installing.md#task-4481)
    -   [Analytics data collection](bolt_installing.md#concept-8242)
-   [Using Bolt commands](running_bolt.md)
    -   [Running basic Bolt commands](running_bolt_commands.md#concept-3531)
        -   [Run a command on remote nodes](running_bolt_commands.md#concept-4161)
        -   [Running commands with redirection or pipes](running_bolt_commands.md#concept-325)
        -   [Run a script on remote nodes](running_bolt_commands.md#concept-6503)
        -   [Upload files or directories to remote nodes](running_bolt_commands.md#concept-6839)
    -   [Adding options to Bolt commands](bolt_options.md#concept-7943)
        -   [Specify target nodes](bolt_options.md#concept-743)
        -   [Set a default transport](bolt_options.md#concept-8521)
        -   [Specify connection credentials](bolt_options.md#concept-8428)
        -   [Rerunning commands based on the last result](bolt_options.md#rerunning-based-on-result)
    -   [Bolt command reference](bolt_command_reference.md#reference-9397)
-   [Self-paced training](getting_started_with_bolt.md)
-   [Tasks and plans](writing_tasks_and_plans.md)
    -   [Inspecting tasks and plans](inspecting_tasks_and_plans.md)
    -   [Running tasks](bolt_running_tasks.md#concept-207)
        -   [Passing structured data](bolt_running_tasks.md#passing-structured-data)
        -   [Specifying the module path](bolt_running_tasks.md#concept-9325)
    -   [Running plans](bolt_running_plans.md#concept-8843)
        -   [Passing structured data](bolt_running_plans.md#passing-structured-data)
        -   [Specifying the module path](bolt_running_plans.md#concept-9325)
    -   [Installing modules](bolt_installing_modules.md#installing-modules)
        -   [Packaged modules](bolt_installing_modules.md#packaged-modules)
        -   [Configure Bolt to download and install modules](bolt_installing_modules.md#install-modules)
    -   [Directory structures for tasks and plans](directory_structure.md#concept-4733)
        -   [Directory structure of a module](directory_structure.md#directory-structure-module)
        -   [Modules for projects](directory_structure.md#concept-6910)
        -   [Standalone modules](directory_structure.md#concept-9775)
    -   [Writing tasks](writing_tasks.md#concept-8341)
        -   [Secure coding practices for tasks](writing_tasks.md#concept-8077)
        -   [Naming tasks](writing_tasks.md#concept-676)
            -   [Single-platform tasks](writing_tasks.md#single-platform-tasks)
            -   [Cross-platform tasks](writing_tasks.md#cross-platform-tasks)
        -   [Sharing executables](writing_tasks.md#sharing-executables)
        -   [Sharing task code](writing_tasks.md#concept-2127)
        -   [Writing remote tasks](writing_tasks.md#writing-remote-tasks)
        -   [Defining parameters in tasks](writing_tasks.md#concept-5468)
        -   [Returning errors in tasks](writing_tasks.md#concept-2414)
        -   [Structured input and output](writing_tasks.md#concept-6846)
            -   [Structured input](writing_tasks.md#concept-1477)
            -   [Returning structured output](writing_tasks.md#concept-87)
        -   [Converting scripts to tasks](writing_tasks.md#concept-4523)
            -   [Wrapping an existing script](writing_tasks.md#wrapping-existing-script)
        -   [Supporting no-op in tasks](writing_tasks.md#concept-8306)
        -   [Task metadata](writing_tasks.md#concept-677)
            -   [Adding parameters to metadata](writing_tasks.md#concept-1466)
            -   [Task metadata reference](writing_tasks.md#reference-9297)
            -   [Task metadata types](writing_tasks.md#reference-3806)
        -   [Specifying parameters](writing_tasks.md#concept-21)
    -   [Writing plans in Puppet language](writing_plans.md#concept-7890)
        -   [Naming plans](writing_plans.md#concept-4485)
        -   [Defining plan parameters](writing_plans.md#concept-2302)
        -   [Returning results from plans](writing_plans.md#concept-6234)
        -   [Returning errors in plans](writing_plans.md#concept-3968)
        -   [Success and failure in plans](writing_plans.md#concept-7361)
        -   [Puppet and Ruby functions in plans](writing_plans.md#concept-1196)
        -   [Handling plan function results](writing_plans.md#concept-2722)
        -   [Passing sensitive data to tasks](writing_plans.md#concept-2235)
        -   [Target objects](writing_plans.md#concept-5598)
        -   [Plan logging](writing_plans.md#concept-1852)
        -   [Example plans](writing_plans.md#example-plans)
        -   [Plan execution functions and standard libraries](plan_functions.md#concept-6192)
            -   [Bolt functions](plan_functions.md#bolt_functions)
        -   [Applying manifest blocks](applying_manifest_blocks.md#concept-4926)
            -   [How manifest blocks are applied](applying_manifest_blocks.md#concept-3038)
            -   [Using Hiera data in a manifest block](applying_manifest_blocks.md#concept-4446)
            -   [Available plan functions](applying_manifest_blocks.md#concept-9990)
            -   [Manifest block limitations](applying_manifest_blocks.md#concept-3628)
            -   [Create a sample manifest for nginx on Linux](applying_manifest_blocks.md#task-6111)
            -   [Create a sample manifest for IIS on Windows](applying_manifest_blocks.md#task-6559)
            -   [Using Puppet device modules from an apply statement](applying_manifest_blocks.md#puppet_device_modules_apply)
    -   [Writing plans in YAML](writing_yaml_plans.md#writing-yaml-plans)
        -   [Naming plans](writing_yaml_plans.md#concept-4485)
        -   [Plan structure](writing_yaml_plans.md#yaml-plan-structure)
            -   [Steps key](writing_yaml_plans.md#steps-key)
            -   [Parameters key](writing_yaml_plans.md#parameters_key)
            -   [How strings are evaluated](writing_yaml_plans.md#string-evaluation)
        -   [Using variables and simple expressions](writing_yaml_plans.md#using-variables-and-simple-expressions)
        -   [Connecting steps](writing_yaml_plans.md#connecting-steps)
        -   [Returning results](writing_yaml_plans.md#returning-results)
        -   [Computing complex values](writing_yaml_plans.md#computing-complex-values)
        -   [Converting YAML plans to Puppet plans](writing_yaml_plans.md#converting-yaml-plans-to-puppet-plans)
-   [Configuring Bolt](configuring_bolt.md)
    -   [Project directories](bolt_project_directories.md#project-directories)
        -   [Types of project directories](bolt_project_directories.md#project-directory-types)
        -   [How the project directory is chosen](bolt_project_directories.md#project-directory-selection)
        -   [Project directory structure](bolt_project_directories.md#project-directory-structure)
    -   [Bolt configuration options](bolt_configuration_options.md)
    -   [Using Bolt with Puppet Enterprise](bolt_configure_orchestrator.md)
    -   [Connecting Bolt to PuppetDB](bolt_connect_puppetdb.md)
-   [Inventory file](inventory_file.md)
    -   [Inventory file version 2](inventory_file_v2.md#inventory-file-v2)
        -   [Migrating to version 2](inventory_file_v2.md#migrating-to-v2)
        -   [Version 2 features](inventory_file_v2.md#features_v2)
            -   [Creating a node with a human readable name and IP address](inventory_file_v2.md#human-readable-name)
            -   [Plugins and dynamic inventory](inventory_file_v2.md#plugins-and-dynamic-inventory)
        -   [Inventory config](inventory_file_v2.md#inventory-config-v2)
        -   [Inventory facts, vars, and features](inventory_file_v2.md#inventory-facts-vars-features-v2)
        -   [Objects](inventory_file_v2.md#objects-v2)
        -   [File format](inventory_file_v2.md#file-format-v2)
        -   [Precedence](inventory_file_v2.md#precedence-v2)
        -   [Remote targets](inventory_file_v2.md#remote-targets-v2)
    -   [Generating inventory files](inventory_file_generating.md)
-   [Bolt examples](bolt_examples.md#using-bolt-to-do-stuff)
    -   [Run a PowerShell script that restarts a service](bolt_examples.md#powershell_script_that_restarts_a_service)

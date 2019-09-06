<?xml version="1.0" encoding="UTF-8"?><?path2rootmap-uri ./?>
<!DOCTYPE topic
  PUBLIC "-//OASIS//DTD DITA Topic//EN" "topic.dtd">
<topic id="project-directories"><title>Project directories</title><prolog><author>Melissa Amos &lt;melissa.amos@puppet.com\&gt;</author></prolog><body><p>Bolt runs in the context of a project directory or a <codeph>Boltdir</codeph>. This directory contains all of the configuration, code, and data loaded by Bolt.</p><p>The project directory structure makes it easy to share Bolt code by committing the project directory to Git. You can then check different repositories of Bolt code into different directories in order to manage various applications.</p><p><b>Parent topic:</b><xref href="configuring_bolt.md" format="dita" type="topic">Configuring Bolt</xref></p></body><topic id="types-of-project-directories"><title>Types of project directories</title><body><p>There are three types of project directories that you can use depending on how you're using Bolt.</p></body><topic id="local-project-directory"><title>Local project directory</title><body><p>Bolt treats a directory containing a <codeph>bolt.yaml</codeph> file as a project directory. Use this type of directory to track and share management code in a dedicated repository.</p><p><b>Tip:</b> You can use an existing control repo as a Bolt project directory by adding a <codeph>bolt.yaml</codeph> file to it and configuring the <codeph>modulepath</codeph> to match the <codeph>modulepath</codeph> in <codeph>environment.conf</codeph>.</p><p>A project directory of this type has a structure like:</p><codeblock xml:space="preserve">project/
├── Puppetfile
├── bolt.yaml
├── data
│   └── common.yaml
├── inventory.yaml
└── site-modules
    └── project
        ├── manifests
        │   └── my_class.pp
        ├── plans
        │   ├── deploy.pp
        │   └── diagnose.pp
        └── tasks
            ├── init.json
            └── init.py</codeblock></body></topic><topic id="embedded-project-directory"><title>Embedded project directory</title><body><p>Bolt treats a directory containing a subdirectory called <codeph>Boltdir</codeph> as a project directory. Use this type of directory to embed Bolt management code into another repo.</p><p>For example, you can store management code in the same repo as the application it manages without cluttering up the top level with multiple files. This structure allows you to run Bolt from anywhere in the application's directory structure.</p><p>A project with an embedded project directory has a structure like:</p><codeblock xml:space="preserve">project/
├── Boltdir
│   ├── Puppetfile
│   ├── bolt.yaml
│   ├── data
│   │   └── common.yaml
│   ├── inventory.yaml
│   └── site-modules
│       └── project
│           ├── manifests
│           │   └── my_class.pp
│           ├── plans
│           │   ├── deploy.pp
│           │   └── diagnose.pp
│           └── tasks
│               ├── init.json
│               └── init.py
├── src #non Bolt source code for the project
└── tests #non Bolt tests for the project</codeblock><p><b>Note:</b> If a directory contains both <codeph>Boltdir</codeph> and <codeph>bolt.yaml</codeph>, the <codeph>Boltdir</codeph> directory is used as the project directory rather then the parent.</p></body></topic><topic id="user-project-directory"><title>User project directory</title><body><p>If Bolt can't find a project directory based on <codeph>Boltdir</codeph> or <codeph>bolt.yaml</codeph>, it uses <codeph>~/.puppetlabs/bolt</codeph> as the project directory. Use this type of directory if you have a single set of Bolt code and data that you use across all projects.</p></body></topic></topic><topic id="how-the-project-directory-is-chosen"><title>How the project directory is chosen</title><body><p>Bolt uses these methods, in order, to choose a project directory.</p><ol><li><p>Manually specified</p><p>You can specify on the command line what directory Bolt should use with <codeph>--boltdir &lt;DIRECTORY_PATH&gt;</codeph>. There is not an equivalent configuration setting because the project directory must be known in order to load configuration.</p></li><li><p>Parent directory</p><p>Bolt traverses parents of the current directory until it finds a directory containing a <codeph>Boltdir</codeph> or <codeph>bolt.yaml</codeph>, or it reaches the root of the file system.</p></li><li><p>User project directory</p><p>If no directory is specified manually or found in a parent directory, the user project directory is used.</p></li></ol></body></topic><topic id="project-directory-structure"><title>Project directory structure</title><body><p>The default paths for all Bolt configuration, code, and data are relative to the modulepath.</p><table><tgroup cols="2"><colspec colname="col1" colnum="1"/><colspec colname="col2" colnum="2"/><thead><row><entry colname="col1">Directory</entry><entry colname="col2">Description</entry></row></thead><tbody><row><entry colname="col1"><codeph>[bolt.yaml](bolt_configuration_options.md)</codeph></entry><entry colname="col2">Contains configuration options for Bolt.</entry></row><row><entry colname="col1"><codeph>hiera.yaml</codeph></entry><entry colname="col2">Contains the Hiera config to use for node-specific data when using <codeph>apply</codeph>.</entry></row><row><entry colname="col1"><codeph>[inventory.yaml](inventory_file.md)</codeph></entry><entry colname="col2">Contains a list of known targets and target specific data.</entry></row><row><entry colname="col1"><codeph>[Puppetfile](bolt_installing_modules.md#)</codeph></entry><entry colname="col2">Specifies which modules should be installed for the project.</entry></row><row><entry colname="col1"><codeph>[modules/](bolt_installing_modules.md#)</codeph></entry><entry colname="col2">The directory where modules from the <codeph>Puppetfile</codeph> are installed. In most cases, these modules should not be edited locally.</entry></row><row><entry colname="col1"><codeph>[site-modules](writing_tasks_and_plans.md)</codeph></entry><entry colname="col2">Local modules that are edited and versioned with the project directory.</entry></row><row><entry colname="col1"><codeph>data/</codeph></entry><entry colname="col2">The standard path to store static Hiera data files.</entry></row></tbody></tgroup></table></body></topic></topic>
---
tags:
  - centos
---
Puppet Server v4 tested on CentOS 7. 
#### Overview

**Architecture**

- **Agent-Based**: Puppet requires an agent installed on each managed node. The Puppet Master server sends configuration data to these agents, which apply the configurations on the nodes.
- **Pull-Based**: Puppet operates in a pull-based model, where agents on nodes periodically check in with the Puppet Master to pull configurations.

**Language and Syntax**

- **Domain-Specific Language (DSL)**: Puppet uses its own DSL, which is Ruby-based. This makes Puppet powerful but can be more challenging to learn and read for those unfamiliar with Ruby or programming.
- **Declarative**: Puppet is primarily declarative, meaning you define the end state you want, and Puppet determines how to get there.

**Idempotency and State Management**

* Puppet is inherently idempotent and **stateful**. Since Puppet agents check in periodically  with the Puppet Master, they can detect configuration drift automatically and enforce the desired state continuously.

**Use Cases**

- Often favored in **large, complex, or highly regulated environments** where continuous configuration management and strict control are priorities.
- Well-suited for compliance-heavy environments (like finance and healthcare) that require strict configuration state enforcement.

**Extensibility and Ecosystem**

* `Puppet Forge` provides a comprehensive set of pre-built modules.
 * `PuppetDB` for storing node information and **Bolt** for orchestration, adding extensibility for larger infrastructures.
* `Facter`: A tool for discovering system facts on nodes. 
* `Hiera`: A key-value lookup database for declaring Puppet configuration data
* `MCollective`: An orchestration tool for managing Puppet nodes

Puppet relies on TLS to authenticate connections between master and nodes

#### The Puppet Hierarchy

Environment -> Module -> Class -> Resource Type -> Attribute

##### Environment

- **Definition**: Environments in Puppet allow you to manage multiple, isolated sets of configurations. This is useful for organizing configurations by stages, such as `production`, `development`, and `testing`.
- **Location**: Typically located under `/etc/puppetlabs/code/environments/`.

##### Modules

- **Definition**: A module is a self-contained package of code and data, primarily used to manage specific applications or functions (e.g., `apache`, `ssh`, `nginx`). Modules are stored within an environment and contain manifests, templates, files, and sometimes plugins.
- **Location**: Typically under `environments/production/modules/` or other environment-specific directories.

##### Class

- **Definition**: Classes are the main configuration units within a module. A class groups related configuration resources (e.g., `package`, `file`, `service` resources) and parameters. Each class should represent a logical grouping of configurations for a particular function, like `apache::install` or `apache::config`.
- **Location**: Defined within the `manifests` directory of a module, often in `init.pp` for the main class or in separate files if the module has multiple classes.

##### Resource Type

- **Definition**: A resource type is the smallest unit of configuration in Puppet. Resources define specific states for components of a system, like a package, service, or file. There are core resource types (e.g., `file`, `service`, `package`) and defined types, which are custom resource types built from other resources.
- **Components**: Resources have attributes (like `ensure`, `path`, `content`, etc.) that specify the desired state of the system component.

#### Puppet Master Server

Repo:
[YUM Puppet](https://yum.puppet.com/puppet/el/9/x86_64/index.html)

Package:
`puppetserver`

Port:
`8140/tcp`

Configuration files
- `/etc/puppetlabs/puppetserver/conf.d/puppetserver.conf`: Principal configuration file. Paths, TLS
- `/etc/puppetlabs/puppet/puppet.conf`: Main `conf` file. Environment is set here

The environment directory structure

```
/etc/puppetlabs/code/environments/production/
├── environment.conf
├── hieradata
├── manifests
│   └── site.pp
├── modules
└── other_modules
```

`/etc/puppetlabs/code/environments/production`: The root environment directory. 
`/etc/puppetlabs/code/environments/production/manifests` directory is used for environment-wide configuration, typically containing a main manifest file that orchestrates the configuration of nodes within that environment - the site manifest (`site.pp`)
`environment.conf`: Module root directories are defined here
#### Site Manifest (`site.pp`):

- **Purpose**: The `site.pp` file is the primary entry point for the Puppet environment. This is where you define the global configuration for nodes in the environment and can include node-specific declarations or global classes.
- **Contents**: It often includes:
    - **Node Definitions**: Specific configurations for particular nodes or groups of nodes.
    - **Global Declarations**: Classes and resources you want applied to all nodes in the environment. (node default)
    - **Role/Profiles**: Definitions to include role and profile classes that structure the overall environment.

##### **Node Definitions**:

- Here you can define specific node blocks in `site.pp` or a separate files.
- Node definitions allow you to apply configurations directly to individual nodes based on hostname, IP address, or regex matching.

`/etc/puppetlabs/code/environments/production/manifests/site.pp`
```
node default {
	include profile::base
}

node 'web1.ohio.cc' {
	include profile::web_server
}

node /^web\d+\.ohio.cc$/ {
    include profile::web_server
}
```

Additional organizational manifest files can be included in the directory for better organization, which then can be referenced in `site.pp`

`site.pp`
```
import 'nodes.pp'
import 'roles.pp'
```

Start the Puppet Master Server

``` bash
/opt/puppetlabs/bin/puppet resource service puppetserver ensure=running enable=true
```

#### Connecting Puppet Clients (nodes)

Repo
`puppet repos...`

Packages
`puppet-agent`

Connect to the Puppet server

```bash
/opt/puppetlabs/bin/puppet agent --server puppet.ohio.cc --test --waitforcert 15
```

```
Info: Creating a new SSL key for web1.ohio.cc
Info: Caching certificate for ca
Info: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for web1.ohio.cc
Info: Certificate Request fingerprint (SHA256): F9:F8:26:E2:48:4E:6A:20:81:42:32:
Notice: Did not receive certificate
```

- `--test`: Runs the Puppet client in the foreground and prevents it from running as a daemon. 
- `--noop`: Runs the Puppet client non-destructively

The client created a private key and a certificate signing request, and waits for the puppet server to sign it, trying every 15 seconds.

On the server, to list Puppet client certificate requests waiting to be signed

```
/opt/puppetlabs/bin/puppet cert list
```

```
"web1.ohio.cc" (SHA256) F9:F8:26:E2:48:4E:6A:20:81:42:32:A1:29:FF:6F:3C:FE:15:83:
```

Sign the request

```
/opt/puppetlabs/bin/puppet cert sign web1.ohio.cc
```

```
Signing Certificate Request for:
  "web1.ohio.cc" (SHA256) F9:F8:26:E2:48:4E:6A:20:81
  Notice: Signed certificate request for web1.ohio.cc
```

If cert signing has been successful, the client should receive

```
Info: Caching certificate for web1.ohio.cc
Info: Caching certificate_revocation_list for ca
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Error: Facter: error while resolving custom facts i
Error: Could not retrieve catalog from remote server: Error 500 on SERVER: Ser
Warning: Not using cache on failed catalog
Error: Could not retrieve catalog; skipping run
```

#### Modules

A **Puppet module** is a reusable bundle of code, files, and resources that defines a specific set of configurations or tasks in a structured way. Modules are the building blocks for organizing and sharing Puppet code, allowing you to manage resources (like users, packages, services, or configurations) across multiple nodes consistently. They provide a standardized way to organize related manifests, templates, files, and custom functions for easy distribution and reuse

Module paths must be included in the `environment.conf` file.

`environment.conf`
```
modulepath = ./<module_name>:./modules:$basemodulepath
```

##### Module Directory Structure Overview

```
apache/     # Module
├── manifests/
│   ├── init.pp        # Main class that sets up Apache
│   └── config.pp      # Subclass for managing Apache configuration files
├── files/
│   └── httpd.conf     # Example static configuration file
├── templates/
│   └── vhost.conf.epp # Template for creating virtual host files
├── metadata.json      # Module metadata
```

A typical Puppet module has a well-defined directory structure:
* `apache`: Name of the module
- **`manifests/`**: Contains the core configuration logic, typically with an `init.pp` file, which defines the `apache` main class that represent the `apache` module.
- **`files/`**: Stores static files that can be copied to nodes (e.g., configuration files).
- **`templates/`**: Contains templates, usually written in Embedded Puppet (EPP) or ERB, that generate dynamic content.
- **`metadata.json`**: Provides metadata about the module, like version, author, and dependencies, which is useful for the Puppet Forge and version management.

##### Module Namespace

**Module Name as Namespace Prefix**: When you create a module, the module name becomes a prefix for any classes, defines, functions, or other resources within that module. For example, if you have a module named `security`, then classes inside this module belong to the `security` namespace and are referenced as `security::class_name`

```
modulepath/
└── security
    └── manifests
        └── firewall.pp
        └── ppolicy.pp
```

This ensures that Puppet knows exactly where to find the class and avoids ambiguity if another module has a similarly named class.

``` puppet
class security::firewall {
 ...
}
```

``` puppet
include security::firewall
include security::ppolicy
```

Classes that are not part of a module belong to the **global** or **root** namespace. They are usually used for quick testing and are stored in the `manifests/` directory in the root environment directory.
##### The `init.pp` 

The `init.pp` is where the main class of the module is defined - the `security` class, which represents the module itself and serves as an entry point for the module, allowing additional class files (like `firewall.pp` or `ppolicy.pp`) to extend functionality rather than serve as the primary setup. Puppet interprets `security` as `security::init` if the `init.pp` is present.

`security/manifests/init.pp`
``` puppet
class security {
	include security::firewall
	include security::ppolicy
}
```

Now the main module class can be included like

``` puppet
include security
```

##### Module Administration

`puppet help module`

``` bash
puppet module list --tree
```

```
/etc/puppetlabs/code/environments/production/sites
├─┬ puppetlabs-apache (v1.11.0)
│ ├── puppetlabs-stdlib (v4.25.0)
│ └── puppetlabs-concat (v2.1.0)
├── role (???)
└── profile (???)
/etc/puppetlabs/code/environments/production/modules
└── sudo (???)
```

##### Third-Party Modules

[Puppet Modules](https://forge.puppet.com/modules/puppetlabs/concat/dependencies) Check this site for modules, versions and dependencies

Search for puppet modules

``` bash
/opt/puppetlabs/bin/puppet module search apache
```

Install a puppet module

```
/opt/puppetlabs/bin/puppet module install puppetlabs-apache [--version VERSION]
```

#### Interpolation in Strings

`"This is a {variable}"`
`f"This is a {variable}"`

**Formatting**: It’s best to wrap variable names directly in the string without spaces to prevent unexpected syntax issues. Also, escape any special characters like `$name`

``` puppet
notify { "Kostana we, ${variable}?": }
```

Single quotes (`''`): Everything inside single quotes is taken literally. Used for strings

```
notify { 'Using single quotes, $variable will not be interpolated': }
```

Double quotes (`""`): Use double quotes when you want to include variables or special characters (like `\n` for a newline) within a string

```
notify { "Using double quotes, $variable will be interpolated": }
```

Nested  quotes must be escaped (`notify { 'Hello, \'Debian\'!' }`)

```
notify { 'Here\'s an example with single-quote escaping': }
notify { "This is \"double-quote\" escaping within a double-quoted string": }
```

#### Puppet Classes

In Puppet, a **class** is a block of code that organizes configuration into reusable, structured, and modular units. Classes typically define how specific resources (like packages, files, or services) should be managed and configure nodes with predictable and reusable settings.

Define a standalone class (not belonging to a module)

`/etc/puppetlabs/code/environments/production/manifests/example_class.pp`
``` puppet
class class_name {
	# Resource declarations here {
	}
}
```

Define a class belonging to a module

`/etc/puppetlabs/code/environments/production/sites/profile/manifests/web_server.pp`
``` puppet
class module_name::class_name {
    # Additional configurations
    }
}
```

##### Declaring Classes

In Puppet, "declaring a class" means creating an instance of that class

``` puppet
include apache
```

`include` simply declares the class and applies it. If the class is already declared elsewhere, `include` will not apply it again; it ensures that the class is included only once.

Declare the `apache` class within the `web1.ohio.cc` node, meaning all resources in the `apache` class will be applied to that node.

`site.pp`
``` puppet
node 'web1.ohio.cc' {
	include apache
}
```

Classes can also be declared within other classes. This can be done either with `include` or with a parameterized declaration.

``` puppet
class profile::web_server {
    include apache
}
```
#### Resource Types

[Core Puppet Resource Types](https://www.puppet.com/docs/puppet/6/cheatsheet_core_types#cheatsheet_core_types)

Resource types are declared

``` puppet
res_type { 'TITLE':
	attribute => 'VALUE',
}
```
##### Services

``` puppet
service { 'my_ssh_service':
	name => 'sshd'
	ensure => 'running'
}
```

`my_ssh_service`: The title serves as an unique identifier. 
`name`: The actual name of the service

When the `name` attribute is omitted, the title value must match the actual name of the service that is to be managed, or else Puppet will return an error if it can't find the service.

``` puppet
service { 'sshd':
  ensure => 'running',
  enable => true,
}
```

##### Resource Path Referencing 

Inside the module, all paths are relative to the module's base path because the `modulepath` is already defined in the environment.

```
modules/ssh/
├── files
│   └── sshd_config_web1.conf
├── manifests
│   ├── init.pp
└── templates
    └── sshd_config.epp
```

For example, referencing a template within `ssh` module would simply be:

``` puppet
content => epp('ssh/sshd_config.epp', ...)
```

Same is with files inside the `files` directory. 

``` puppet
file {'/etc/sudoers':
    source => [ "puppet:///modules/ssh/sshd_config_web1.conf", ]
```

#### Apply Configurations to Nodes

Pull the configuration from the master

``` bash
/opt/puppetlabs/bin/puppet agent --server puppet.ohio.cc --test
```

```
Info: Caching catalog for web1.ohio.cc
Info: Applying configuration version '1730147337'
Notice: profile::web_server - loaded
Notice: /Stage[main]/Profile::Web_server/Notify[profile::web_server - loaded]
```

This confirms that the `profile::web_server` class has been loaded and applied to the node. Using `notify` this way is straightforward for debugging or tracking which classes are applied.

#### Roles and Profiles 

- **Profiles (Generic)**: Profiles are intended to be reusable, modular components that focus on configuring a specific aspect of a system. For example, `profile::web_server` might define settings common to any web server, whether it’s Apache or Nginx, without concern for the exact context in which it’s used. The idea is to make profiles versatile enough to be included in multiple roles without needing modification.
- **Roles (Specific)**: Roles are purpose-driven and map closely to the actual roles that nodes will fulfill in your environment. For example, `role::apache_web` is configured to set up a node specifically as an Apache-based web server. It would include `profile::web_server` to cover general web server requirements, but it could also include other profiles (like logging or monitoring) tailored to the needs of an Apache server.

**Profiles Module** (`profile`):

 **Profiles can be thought of as the “how”**—how to configure a web server, a database, a firewall, etc., in a way that is reusable and adaptable.
- Profiles are often stored in `modules/profile/manifests/`.

`../modules/profile/manifests/web_server.pp`
``` puppet
class profile::web_server {
    notify { "profile::web_server - loaded":
    }
}
```

**Roles Module** (`role`):

- Combines profiles to represent the overall function of a node.
- Roles are often stored in `modules/role/manifests/`.

`modules/role/manifests/apache_web.pp`
``` puppet
class role::apache_web {
  include profile::web_server
  include profile::monitoring
  notify { "role::apache_web - loaded":
    }
}
```

**Assigning Roles to Nodes:

`site.pp`
``` puppet
node 'web1.example.com' {
  include role::apache_web
}
```

So the result of this configuration when called from a node should be

```
Notice: profile::web_server - loaded
Notice: /Stage[main]/Profile::Web_server/Notify[profile::web_server - loaded]/message: defined 'message' as 'profile::web_server - loaded'
Notice: role::apache_web - loaded
Notice: /Stage[main]/Role::Apache_web/Notify[role::apache_web - loaded]/message: defined 'message' as 'role::apache_web - loaded'
```

#### Class Parameters

Puppet classes can take parameters, allowing flexibility in configuration. Parameters allow you to reuse a class with different values.

``` puppet
class my_class (
  String $param1,
  Integer $param2 = 8080,
  Boolean $param3 = true,
) {
  # Class code goes here
}
```

``` puppet
class apache ($port_number) {
  notify { "Apache is set to use port: $port_number": }
}
```

To instantiate (declare) the class and pass the arguments

``` puppet
class { 'apache':
	port_number => 8080,
    }
}
```

##### Organizing Classes: Role and Profile Pattern

Puppet environments often use **roles** and **profiles** to separate concerns:

- **Profile classes** contain reusable configurations and logical groups of resources.
- **Role classes** define the specific role of a node, often by including one or more profile classes.

``` puppet
# Profile for a web server setup
class profile::web_server {
	include apache
	include php
}
```

``` puppet
# Role for a basic web server
class role::web_server {
	include profile::web_server
	# Additional conf
}
```

#### Facter and Variables

The command `facter -p` is used to display facts about the system on which Puppet Facter is running. Facter is a tool that collects and reports various details about the operating system, hardware, network interfaces, and other system attributes. This allows for configure multiple host configurations

```bash
/opt/puppetlabs/bin/facter -p
```

```
networking => {
  dhcp => "192.168.137.3",
  domain => "ohio.cc",
  fqdn => "ldap.ohio.cc",
  hostname => "ldap",
  ...
  }
```

In Puppet, facts gathered by `Facter` can be used as variables within your manifests. Facts can be accessed in Puppet manifests using the `$facts` variable, like `$facts['key']...`
Therefore, `$facts['networking']['domain]` will return the value 'ohio.cc'.

Trusted Facts: Cannot be self-reported by the node. Example the hostname retrieved from a TLS certifacate

Using the `file` resource type to distribute files to clients, based on hostname and os family. Files are placed in `./files` directory of the module.

`/etc/puppetlabs/code/environments/production/modules/sudo/manifests/init.pp`
```
class sudo {
        notify {
                "sudo - loaded mada fakaars":
        }
        package { sudo:
                ensure => 'present',
        }
        file {'/etc/sudoers':
            source => [
				"puppet:///modules/sudo/sudo_${hostname}",
                "puppet:///modules/sudo/sudo_${os['family']}",
                'puppet:///modules/sudo/sudo_default',
            ],
        }
}
```

Even though the URL is written as if it points directly to `modules/sudo`, Puppet actually interprets it as pointing to `modules/sudo/files/`. Here’s the breakdown:

**`puppet:///modules/sudo/sudo_${os['family']}`** points to **`modules/sudo/files/sudo_${os['family']}`** on the Puppet server.

Directory tree

```
../modules/
└── sudo
    ├── files
    │   ├── sudo_Debian
    │   ├── sudo_default
    │   ├── sudo_RedHat
    │   ├── sudo_web1.ohio.cc
    │   └── sudo_web2.ohio.cc
    └── manifests
        └── init.pp
```

#### Deploy Apache Server

`/etc/puppetlabs/code/environments/production/modules/role/manifests/apache_web.pp`
``` puppet
class role::apache_web (
	String $vhost_name,
	String $vhost_doc_root,
	Integer $vhost_port,
) {
    # Ensure the Apache module is included (provided by puppetlabs-apache)
	include apache

	# Define the Apache virtual host with specified parameters
	apache::vhost { $vhost_name:
    port    => $vhost_port,
    docroot => $vhost_doc_root,
  }
}
```

`class role::apache_web ( params )` : Defines a class with parameters
The `include` statement loads the `apache` class to ensure that the Apache web server package and service are installed and configured.
`apache::vhost` is a defined resource type, provided by the `puppetlabs-apache` module.
`$vhost_name` is used as the title of this specific `apache::vhost` resource.
`port` is an attribute of the `apache::vhost` resource.

Apply the configuration to a node by passing the class arguments directly

``` puppet
node 'kavarna.ohio.cc' {
    class { 'role::apache_web':
        vhost_port => 80,
        vhost_name => 'www.example.cc',
        vhost_doc_root => '/var/www/html/ebredebre' ,
    }
}
```

Pull the configuration from the puppet master on the node

``` bash
/opt/puppetlabs/bin/puppet agent --server puppet.ohio.cc --test
```

```
Info: /Stage[main]/Apache/File[/etc/httpd/conf/httpd.conf]:
```

The client node now has an installed and running Apache server with a configured virtual host

```
# ************************************
# Vhost template in module puppetlabs-apache
# Managed by Puppet
# ************************************

<VirtualHost *:80>
  ServerName www.example.com

  ## Vhost docroot
  DocumentRoot "/var/www/html"

  ## Directories, there should at least be a declaration for /var/www/html

  <Directory "/var/www/html">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Require all granted
  </Directory>

  ## Logging
  ErrorLog "/var/log/httpd/www.example.com_error.log"
  ServerSignature Off
  CustomLog "/var/log/httpd/www.example.com_access.log" combined
</VirtualHost>
```

#### Conditional Statements

##### Cases

``` puppet
class host {
    case $facts['os']['family'] {
	    'RedHat': {
	        notify { "Ako si 'RedHat' si pederasz": }
        }
        'Debian': {
	        notify { "Ako si 'Debian' si debil": }
        }
        'default': {
	        notify { "Nqma takova neshto, brad!": }
        }
    }
}
```

``` puppet
$names = $facts['os']['name']
	case $names {
	    'CentOS', 'RedHat': { include centos }
        /^(Ubuntu|Debian)$/: { include debian }
		default: { notify { "Ne staa": } }
```

`'CentOS', 'RedHat'`: is a **literal list** of strings. Puppet checks for an exact match to each element in the list individually.
##### Ternary Operator (Selector)

``` bash
class host {
    $web_server = $facts['os']['family'] ? {
        'RedHat' => 'httpd',
        'Debian' => 'apache2',
        default => 'httpd',
    }
    service { $web_server
	    ensure => 'running'
    }
}
```

The title attribute `$web_server` uses its value to determine which service to apply the action to.
##### IF-ELSE

``` puppet
if Integer($facts['os']['release']['major']) < 9 {
    notify { "Ti si po-malak ot 9": }
    } else {
        notify { "Brawo we! Po-golqm ili raeven!": }
}
```

#### Resource Dependencies Metaparameters

[Metaparameters](https://www.puppet.com/docs/puppet/8/metaparameter.html)

##### `require`

``` puppet
service { 'seseha': 
	name => 'sshd',
	ensure => 'running',
	require => File['/etc/ssh/sshd_config'],
}
```

`require`: Meta parameter. `File['/etc/ssh/sshd_config']` will be configured before the `Service['seseha']` resource.

``` puppet
class httpd {
	package { 'httpd': 
		ensure => 'present',
	}
	service { 'httpd': 
		ensure => 'running', 
		enabled => true,
		require => Package['httpd'], 
	}
}
```

##### `notify`

`notify`: Sends a refresh to another resource.

`notify` is used to establish a relationship between two resources, where any changes to the resource `File['/etc/ssh/sshd_config']` will cause the notified resource (`Service['seseha']`) to refresh so that the configuration changes take effect.

```puppet
class ssh::sesega {
        service { 'sshd':
                ensure => 'running',
        }
        file { '/etc/ssh/sshd_config.d/port.conf':
                ensure => 'file',
                notify => Service['sshd'],
                content => epp('ssh/sshd_config.epp', {
                        'ssh_port' => 22,
                }),
        }
}
```

```
Notice: /Stage[main]/Ssh::Sesega/File[...]/content: content changed 
Info: /Stage[main]/Ssh/File[/etc/...conf]: Scheduling refresh of Service[sshd]
Notice: /Stage[main]/Ssh::Sesega/Service[sshd]: Triggered 'refresh' from 1 events
```
##### `subscribe`

`subscribe`: Listens for changes in another resource.

When a resource is configured with `subscribe`, it will:

- Check for any changes in the subscribed-to resource (such as a configuration file).
- Trigger a refresh (like restarting a service) if the subscribed-to resource changes.

``` puppet
file { '/etc/ssh/sshd_config':
  ensure => file,
  source => 'puppet:///modules/ssh/sshd_config',
}

service { 'sshdaemon':
  name    => 'sshd',
  ensure  => 'running',
  enable  => true,
  subscribe => File['/etc/ssh/sshd_config'],
}
```

The `Service['seseha']` subscribes to `File['/etc/ssh/sshd_config']`

In Puppet, `subscribe` listens for changes specifically on resources (like files) that Puppet itself manages—meaning resources defined and enforced through the Puppet server’s catalog. When a resource, such as a file, is changed by Puppet during an agent run, `subscribe` will detect that managed change and then trigger an action, like restarting a service.

If a file is modified outside Puppet's management (e.g., edited manually on the client), `subscribe` won’t detect this because it only listens for changes Puppet applies to the resource, not external or unmanaged modifications. For changes made outside of Puppet's control, those alterations would remain undetected until the next Puppet agent run re-applies the configuration, restoring it to the desired state.

##### `before`

``` puppet
file { '/etc/ssh/sshd_config':
  ensure => file,
  source => 'puppet:///modules/ssh/sshd_config',
  before => Service['sshdaemon'],  # Service runs only after the file is created
}

service { 'sshdaemon':
  name   => 'sshd',
  ensure => 'running',
}
```

`before` ensures that `Service['sshdaemon']` will start only **after** the `File['/etc/ssh/sshd_config']` resource has been applied.

#### Templates

[Templates using EPP](https://www.puppet.com/docs/puppet/8/lang_template_epp.html)

Template files are placed in the `module_name/templates/` directory and are referenced as `epp('module_name/template_name.epp')`

Templates allow for the dynamic creation of configuration files or content by embedding variables, conditional logic, and even loops within text files. Templates are especially useful when configurations vary between nodes or environments, but share a similar structure.

Template formats in Puppet:
* Embedded Puppet (EPP):
	**Syntax:** `<%= expression %>` for embedding variables, `<%- conditional statements -%>` for logic. `.epp` extension.
* Embedded Ruby (ERB)
	* Syntax: `<%= expression %>` and `<% control logic %>`. `.erb`

``` puppet
file { '/etc/ssh/sshd_config':
	path => '/etc/ssh/sshd_config',
	content => epp('ssh/sshd_config.epp', { 
		'root_login' => 'no',
		'port'       =>  22,
		'banner'     =>  '/etc/motd'
	}),
	notify => Service['sshdaemon'],
	}
}
```

`ssh/sshd_config.epp`
```
Port <%= $port %>

Protocol 2

ListenAddress <%= $ipaddress %>

SyslogFacility AUTHPRIV 
PermitRootLogin <%= $root_login %> 
PasswordAuthentication no 
ChallengeResponseAuthentication no 
GSSAPIAuthentication yes GSSAPICleanupCredentials yes UsePAM yes

X11Forwarding yes 
Banner <%= $banner %>
```

`<%= $value %>`: Variable syntax in templates
`$ipaddress`: node fact or class parameter

#### Functions

[Built-in Functions](https://www.puppet.com/docs/puppet/8/function.html)

`include`
`template`

#### Reports

[Reporting](https://docs.puppet. com/puppet/latest/reporting_about.html)

#### Executing Commands on Nodes

